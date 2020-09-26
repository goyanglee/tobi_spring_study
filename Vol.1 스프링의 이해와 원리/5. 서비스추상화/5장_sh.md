# 5장 서비스 추상화

성격이 비슷한 많은 기술들을 스프링에서 어떻게 추상화해서 일관되게 사용할 수 있도록 제공해주는 지 이해한다. 

<br/>

## 5.1 사용자 레벨 관리 기능 추가

사용자 관리 기능에 "정해진 조건에 따라 사용자의 레벨을 주기적으로 변경"해주는 비즈니스 로직을 추가한다. 



### [필드 추가]

#### 레벨 ("BASIC", "SILVER", "GOLD") 필드를 추가하기

- 데이터베이스에 문자열보단 숫자타입으로 필드를 생성하는 것이 좋다. 범위가 작은 숫자로 관리하는 것이 DB 용량 확보에 좋다. (BASIC = 1 / SILVER = 2 / GOLD = 3)
- 반면에 자바 엔티티의 프로퍼티 타입에 숫자 타입을 직접 사용하는 것은 좋지 않다. 범위에서 벗어나는 숫자를 프로퍼티에 넣게되어도 컴파일러가 체크해주지 못해서 안전하지 않다. 

<br/>

### "범위가 정해진 프로퍼티를 선언할 경우에는 숫자 타입을 직접 사용하는 것보다 이늄(Enum) 타입을 사용해서 선언하는 것이 안전하고 편리하다. "

```
public enum Level {
	BASIC(1), SILVER(2), GOLD(3); //enum object's
	
	private final int value; //내부에는 DB에 저장할 int 타입의 레벨 값 존재
	Level(int value) {
		this.value = value; 
	}	
	public int intValue() {
		return value;
	}
	
	public static Level valueOf(int value) {
		switch(value) {
			case 1: return BASIC;
			case 2: return SILVER;
			case 3: return GOLD;
			default: throw new AssertionError("Unknown value: " + value);
		}
	}
}
```

```
public class User {
	...
	Level level;
	int login; //로그인횟수 - 레벨업에 사용되는 필드1
	int recommend; //추천수 - 레벨업에 사용되는 필드2
	
	public Level getLevel() {
		return level;
	}
	public void setLevel(Level level) {
		this.level = level;
	}
	...
}
```

- DB로 데이터를 넣으려면 **``` user.getLevel().intValue()```**
  - Level 이늄은 오브젝트이므로 DB에 저장될 수 있는 SQL 타입이아니다. 따라서 DB 저장 가능한 정수형 값으로 변환해줘야 한다. 
- DB에서 데이터를 불러오려면 **``` user.setLevel(Level.valueOf(rs.getInt("level")))```**
  - 조회하면 ResultSet에서는 DB 타입인 int로 레벨 정보를 가져온다. 이 값을 setLevel()로 전달하면 타입 불일치 에러가 발생한다. 따라서 Level의 스태틱 메소드인 valueOf()를 사용해서 int 타입의 값을 Level 타입의 이늄 오브젝트로 만들어서 setLevel() 메소드에 넣어줘야 한다. 

<br/>



### [사용자 수정 기능 추가]

#### 수정할 정보가 담긴 유저 오브젝트를 전달하면 id를 참고해서 사용자를 찾아 필드 정보를 변경해주는 메소드 만들기

```
public class UserDaoJdbc implements UserDao {
	...
	public void update (User user) {
		this.jdbcTemplate.update(
			"update users set name = ?, password = ?, level = ?, login = ?, recomend = ? where id = ?", user.getName(), user.getPassword(), user.getLevel().intValue(), user.getLogin(), user.getRecommend(), user.getId());
		)
	}
	...
}
```

<br/>

유저정보 업데이트 테스트를 아래코드처럼 하는 경우를 생각해보자. 

```
...
@Test
public void update() {
	dao.deleteAll();
	dao.add(user1); //초기 user1 추가
	
	//user1 업데이트 내용 셋
	user1.setName("켱");
	user1.setPassword("1123");
	user1.setLevel(Level.GOLD);
	user1.setLogin(1000);
	user1.setRecommend(999);
	dao.update(user1);
	
	//DB에서 조회한 정보가 업데이트한 정보와 일치하는 지 확인
	User user1update = dao.get(user1.getId());
	checkSameUser(user1, user1update);
}
...
private void checkSameUser(User user1, User user2) {
	asssertThat(user1.getId(), is(user2.getId()));
	asssertThat(user1.getName(), is(user2.getName()));
	asssertThat(user1.getPassword(), is(user2.getPassword()));
	asssertThat(user1.getLevel(), is(user2.getLevel()));
	asssertThat(user1.getLogin(), is(user2.getLogin()));
	asssertThat(user1.getRecommend(), is(user2.getRecommend))); 
}
```

위처럼 업데이트 테스트를 하게되면 수정할 로우(유저)의 내용이 변경된 것만 확인 가능하고, 수정하지 않아야 할 로우(유저)의 내용이 그대로 남아 있는지는 확인해주지 못한다. 이렇게 되면 문제가 되는 부분이 UPDATE는 WHERE 값이 없어도 아무 경고 없이 정상적으로 동작하는 것처럼 보인다는 것이다. 즉 WHERE 값을 누락했을 때 대비가 불가능하다. 이것을 해결하려면

1. JdbcTemplate의 update()가 돌려주는 리턴 값을 확인하거나, 

   리턴 타입을 int로 변경하고 반환되는 값이 1인 지 확인하는 코드를 추가한다. 영향받은 로우 개수가 1개 이상이면 문제

2. 테스트를 보강해서 원하는 사용자 외의 정보는 변경되지 않았음을 직접 확인해야 한다. (사용자 두명등록해서 하나만 수정한 뒤에 둘 정보를 모두 확인한다)

   ```
   ...
   @Test
   public void update() {
   	dao.deleteAll();
   	dao.add(user1);
   	dao.add(user2); //수정하지 않을 사용자 추가
   	
   	user1.setName("켱");
   	user1.setPassword("1123");
   	user1.setLevel(Level.GOLD);
   	user1.setLogin(1000);
   	user1.setRecommend(999);
   	dao.update(user1);
   	
   	User user1update = dao.get(user1.getId());
   	checkSameUser(user1, user1update);
   	//수정안한 user2 정보가 일치하는 지 확인
   	User user2same = dao.get(user2.getId());
   	checkSameUser(user2, user2same); 
   }
   ...
   ```

<br/>



### [UserService.upgradeLevels()]

#### 전체 사용자 목록을 불러와서 레벨을 업그레이드해주는 기능을 하는 비즈니스 로직을 추가하기

> ! 사용자 관리로직은 서비스 레벨의 클래스에서 코딩한다. UserDaoJdbc 같은 DAO는 데이터를 어떻게 가져오고 조작할 지를 다루는 곳이므로 비즈니스 로직을 두는 곳이 아니다. 

<br/>

그림 5-1

<br/>



사용자 레벨 업그레이드 기능을 구현한 코드

```
public class UserService {
	UserDao userDao; 
	public void setUserDao(UserDao userDao) {
		this.userDao = userDao; 
	}

	public void upgradeLevels() {
    -- 코딩생략 -- 
    //전체 유저목록을 가져와서
    //조건에 부합하는 유저를 찾아 레벨up해주는 로직
  }
}
```

코드를 작성한 후에는, 모든 케이스를 테스트해줘야 한다. (BASIC에서 업O / BASIC에서 업X / SILVER에서 업O / SILVER에서 업 X / GOLD에서 업 X)

<br/>



### [UserService.add()]

#### 최초 가입자 레벨 다루기 - 기본적으로 BASIC

<I>최초 가입자는 기본적으로 BASIC 레벨이어야 한다. 이 부분을 어디에 구현하는 것이 좋을까?</I>

1. UserDaoJdbc? **적합X**

2. User 클래스에서 level 필드를 Level.BASIC으로 초기화? **보통**

   처음 가입할 때를 제외하면 무의미한 정보인데 이 로직만을 위해 클래스에서 직접 초기화하는 것은 별로 좋지 않다. 

3. 비즈니스 로직을 담고 있는 UserService? **적합O**

   UserDao의 add()가 사용자 정보를 담은 User를 받아서 DB에 넣어주는 데에만 집중한다면, UserService에도 마찬가지로 사용자가 등록될 때 처리해야하는 비즈니스 로직에 집중할 add()를 만든다. 

   ```
   public class UserService {
   	...
   	public void add(User user) {
   		if (user.getLevel() == null) user.setLevel(Level.BASIC);
   		userDao.add(user);
   	}
   }
   ```

   
