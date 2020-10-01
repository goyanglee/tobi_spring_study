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

![그림5-1](https://github.com/goyanglee/tobi_spring_study/blob/master/Vol.1%20%EC%8A%A4%ED%94%84%EB%A7%81%EC%9D%98%20%EC%9D%B4%ED%95%B4%EC%99%80%20%EC%9B%90%EB%A6%AC/5.%20%EC%84%9C%EB%B9%84%EC%8A%A4%EC%B6%94%EC%83%81%ED%99%94/5%EC%9E%A5_sh/%E1%84%80%E1%85%B3%E1%84%85%E1%85%B5%E1%86%B75-1.jpeg)

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

<br/>
<br/>




### 코드개선

#### 체크 포인트

- 코드에 중복된 부분은 없는가? 
- 코드가 무엇을 하는 지 이해하기 불편하지 않은가?
- 코드가 자신이 있어야 할 자리에 있는가?
- 앞으로 변경이 일어난다면 어떤 것이 있을 수 있고, 그 변화에 쉽게 대응할 수 있게 작성되어 있는가?

<br/>



#### 예제 : UserService의 upgradeLevels 메소드 로직 리펙토링

```
public void upgradeLevels() {
	List<User> users = userDao.getAll(); 
	
	for(User user : users) {
		Boolean changed = null; 
		
		if (user.getLevel() == Level.BASIC && user.getLogin() >= 50) { //BASIC 레벨 업
			user.setLevel(Level.SILVER);
			changed = true; 
		}
		else if (user.getLevel() == Level.SILVER && user.getRecommend() >= 30) { //SILVER 레벨 업
			user.setLevel(Level.GOLD);
			changed = true; 
		}
		else if (user.getLevel() == Level.GOLD) { //GOLD 레벨 업
			changed = false; 
		}
		else {
			changed = false; 
		}
		
		if (changed) { //레벨변경 true이면 DB update
			userDao.update(user);
		}
	}
}
```



**개선해야 할 점**

- 성격이 다른 로직들이 한 데 함께 존재한다. (레벨의 변화 단계와 업그레이드 조건, 조건이 충족됐을 때 해야 할 작업, 조건 충족 여부를 확인하는 플래그 처리) 
- if 조건 블록이 레벨 개수만큼 반복된다. 새로운 레벨이 추가되면 Level 이늄도 수정해야 하고 upgradeLevels()에 레벨 업그레이드 로직을 담은 코드에 if 조건식과 블록이 추가되어야 한다.
- 현제 레벨과 업그레이드 조건을 동시에 비교하는 부분 즉 성격이 다른 부분이 한 곳에서 처리된다. 

<br/>



**Step1. 기본 로직 흐름을 만들고 구체적인 기능은 메소드화 한다. **

```
public void upgradeLevels() {
	List<User> users = userDao.getAll();
	
	for(User user : users) {
		if (canUpgradeLevel(user)) {
			upgradeLevel(user);
		}
	}
}
```

```
private boolean canUpgradeLevel(User user) {
	Level currentLevel = user.getLevel();
	
	swtich(currentLevel) {
		case BASIC: return (user.getLogin() >= 50);
		case SILVER: return (user.getRecommend() >= 30);
		case GOLD: return false; 
		default: throw new IllegalArgumentException("Unknown Level: " + currentLevel);
	}
}
```

```
private void upgradeLevel(User user) {
	if (user.getLevel() == Level.BASIC) user.setLevel(Level.SILVER);
	else if (user.getLevel() == Level.SILVER) user.setLevel(Level.GOLD);
	userDao.update(user);
}
```

- 새로운 레벨과 그에 대한 로직이 추가될 수 있도록 현재 로직에서 다룰 수 없는 레벨이 주어지면 예외를 발생시킨다. 
- **upgradeLevel() 코드에서 개선 할 점**
  - 다음단계가 무엇인 지 체크하는 부분과 사용자 오브젝트의 레벨 필드를 변경해주는 로직이 함께 존재한다. 
  - 예외 상황에 대한 처리가 없다. (다음 단계가 없는 GOLD 레벨 사용자를 업그레이드하려고 이 로직을 호출하게 되면 아무 처리도 안하고 dao의 업데이트 메소드만 실행됨)
  - 새로운 레벨이 추가되면 조건문이 계속 추가된다. 

<br/>



**Step2. 레벨에 대한 작업은 Level 클래스에서 하도록 만든다. **

```
public enum Level {
	GOLD(3, null), SILVER(2, GOLD), BASIC(1, SILVER); //
	
	private final int value; 
	private final Level next; //
	
	Level(int value, Level next) {
		this.value = value; 
		this.next = next; //
	}
	
	public int intValue() {
		return value; 
	}
	
	public Lvel nextLevel() { //
		return this.next; 
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



**Step3. 유저의 레벨이 변경되는 부분을 User에서 하도록 만든다. User의 내부 정보가 바뀌는 것은 UserService 보다는 User가 스스로 다루는 게 적절하다.  **

> User는 사용자 정보를 담고 있는 단순 자바빈이지만 엄연히 자바 오브젝트이고 내부 정보를 다루는 기능이 있을 수 있다. UserService가 일일이 레벨 업그레이드 시에 User의 어떤 필드를 수정한다는 로직을 갖고 있는 것보다는, User에게 레벨 업그레이드를 해야하니 정보를 변경하라고 요청하는 편이 낫다. 

<br/>

```
public void upgradeLevel() {
	Level nextLevel = this.level.nextLevel();
	if (nextLevel == null) {
		throw new IllegalStateException(this.level + "은 업그레이드가 불가능합니다");
	}
	else {
		this.level = nextLevel; 
	}
}
```

- 더이상 업그레이드가 불가능한 경우에 대한 에러 처리를 해준다. 
  - canUpgradeLevel() 메소드에서 업그레이드 가능 여부를 미리 판단해주기는 하지만
  - User 오브젝트를 UserService에서만 사용하는 것이 아니기 때문에 스스로 예외 상황에 대한 검증 기능을 갖고 있는 편이 안전하다. 

<br/>

```
private void upgradeLevel(User user) {
	user.upgradeLevel(); //
	userDao.update(user);
}
```

<br/>



#### 정리

각 오브젝트와 메소드가 각각 자기 몫의 책임을 맡아 일을 하는 구조로 변경 - UserService/User/Level이 내부 정보를 다루는 자신의 책임에 충실한 기능을 갖고 있으면서 필요가 생기면 이런 작업을 수행해달라고 서로 요청하는 구조로 변경

- [x] 성격이 다른 로직들이 한 데 함께 존재한다.
- [x] if 조건 블록이 레벨 개수만큼 반복된다. 
- [x] 현제 레벨과 업그레이드 조건을 동시에 비교하는 부분 즉 성격이 다른 부분이 한 곳에서 처리된다. 


   
