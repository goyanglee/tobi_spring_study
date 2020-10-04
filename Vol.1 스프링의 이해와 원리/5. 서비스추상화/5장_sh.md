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



#### STEP1. 기본 로직 흐름을 만들고 구체적인 기능은 메소드화 한다.

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



#### STEP2. 레벨에 대한 작업은 Level 클래스에서 하도록 만든다.

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
<br/>


#### STEP3. 유저의 레벨이 변경되는 부분을 User에서 하도록 만든다. User의 내부 정보가 바뀌는 것은 UserService 보다는 User가 스스로 다루는 게 적절하다.

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


<br/>

## 5.2 트랜잭션 서비스 추상화

지금까지 작성한 코드는 트랜잭션 처리가 안되어 있다. 

<br/>

모든 사용자에 대해 레벨 업그레이드 작업을 진행하다가 중간에 예외가 발생해서 작업이 중단되어도, 그 전까지 변경된 사용자 정보는 그대로 유지된다. 트랜잭션 문제이다. 모든 사용자의 레벨을 업그레이드하는 작업인 ``` upgradeLevels()``` 메소드가 하나의 트랜잭션 안에서 동작하지 않았기 때문이다. 

<br/>

> ### 트랜잭션이란
>
> 더 이상 나눌 수 없는 단위 작업을 의미한다. 작업을 쪼개서 작은 단위로 만들 수 없다는 것은 트랜잭션의 핵심 속성인 원자성을 의미한다. 

<br/>

## 트랜잭션 경계설정

하나의 SQL 명령을 처리하는 경우는 DB에서 트랜잭션을 보장해주지만, 여러 개의 SQL이 사용되는 작업을 하나의 트랜잭션으로 처리해야만 하는 경우가 있다. 여러 개의 작업이 하나의 트랜잭션이 되려면 커밋과 롤백 개념이 필요하다. 

- 트랜잭션 롤백 : 모든 작업을 무효화한다. (N번 째 SQL이 성공적으로 DB에서 수행되기 전에 문제가 발생한 경우에는, 이전에 처리했던 SQL 작업도 전부 취소시켜야 한다)
- 트랜잭션 커밋 : 모든 작업을 확정한다. (모든 SQL 작업이 성공적으로 마무리됐다고 DB에 알려서 작업을 확정시켜야 한다)

<br/>

> ### 트랜잭션의 경계란
>
> 애플리케이션 내에서 트랜잭션이 시작되고 끝나는 위치이다. 복잡한 로직흐름에서 정확하게 트랜잭션 경계를 설정하는 일은 매우 중요한 작업이다. 

<br/>



### JDBC 트랜잭션의 트랜잭션 경계설정

- JDBC 트랜잭션은 하나의 Connection을 가져와 사용하다가 닫는 사이에서 일어난다. 
- 트랜잭션의 시작과 종료는 Connection 오브젝트를 통해 발생한다.
  - 트랜잭션의 시작: 자동커밋 옵션을 false로 설정해주는 시점.  ```setAutoCommit(false)``` 
  - 트랜잭션의 종료: 커밋 또는 롤백이 호출되는 시점. ```commit()``` ```rollback()``` 
- 위처럼 트랜잭션의 시작을 선언하고 종료하는 작업을 **트랜잭션의 경계설정**이라고 한다. 
- 하나의 DB 커넥션 안에서 만들어지는 트랜잭션을 **로컬 트랜잭션**이라고 한다. 

<br/>

#### 문제점

JdbcTemplate의 메소드를 사용하는 UserDao는 각 메소드마다 하나의 독립적인 트랜잭션으로 실행된다. 즉, UserDao는 JdbcTemplate를 통해 매번 새로운 DB 커넥션과 트랜잭션을 만들어서 사용한다. UserService의 전체적인 비즈니스 로직 상의 트랜잭션 처리는 불가능한 상태

<br/>

### 비즈니스 로직 내의 트랜잭션 경계설정

위 문제점을 해결하려면 즉 어떤 일련의 작업이 하나의 트랜잭션으로 묶이려면 그 작업이 진행되는 동안 DB 커넥션도 하나만 사용되어야 한다. 결국 트랜잭션의 경계설정 작업을 UserService 쪽으로 가져와야 한다. 

트랜잭션 경계를 ```upgradeLeveles()``` 메소드 안에 두려면 DB 커넥션도 이 메소드에서 만들고 종료시켜야 한다. UserDao의 ```update()``` 메소드가 반드시 ```upgradeLevels()``` 메소드에서 만든 커넥션을 사용해야만 같은 트랜잭션 안에서 동작할 수 있다. 

(그림 5-40)

<br/>

#### 문제점

- DB 커넥션을 비롯한 리소스의 깔끔한 처리를 가능하게 했던 JdbcTemplate을 활용할 수 없다.
- DAO의 메소드와 비즈니스 로직을 담고 있는 UserService의 메소드에 Connection 파라미터가 추가되어야 한다. 
- Connection 파라미터가 UserDao 인터페이스 메소드에 추가되면 UserDao는 더 이상 데이터 액세스 기술에 독립적일 수 없다. 
- DAO 메소드에 Connection 파라미터를 받게 하면 테스트 코드에도 영향을 미친다. 테스트 코드에서 직접 Connection 오브젝트를 일일이 만들어서 DAO 메소드를 호출하도록 변경해야 한다. 

<br/>



### 트랜잭션 동기화

> ### 트랜잭션 동기화란
>
> UserService에서 트랜잭션을 시작하기 위해 만든 Connection 오브젝트를 특별한 저장소에 보관해두고, 이후에 호출되는 DAO의 메소드에서는 저장된 Connection을 가져다가 사용하게 하는 것이다. 트랜잭션 동기화 저장소는 작업 스레드마다 독립적으로 Connection 오브젝트를 저장하고 관리하므로 멀티스레드 환경에서도 안전하다. 

<br/>

#### 스프링에서는 TransactionSynchronizationManager라는 트랜잭션 동기화 관리 클래스를 제공한다. 

- ```initSynchronization()``` : 트랜잭션 동기화 작업 초기화
- ```unbindResource()``` ```clearSynchronization()``` : 트랜잭션 동기화 작업 종료 및 정리



```
private DataSource dataSource; 
public void setDataSource(DataSource dataSource) {
	this.dataSource = dataSource;
}

public void upgradeLevels() throws Exception {
	TransactionSynhronizationManager.initSynchronization(); 
	
	Connection c = DataSourceUtils.getConnection(dataSource);
	c.setAutoCommit(false); 
	
	try {
		//유저목록 전체 불러와서 레벨 업그레이드 수행하는 로직(코드 생략)
		c.commit(); 
	} catch (Exception e) {
		c.rollback();
		throw e; 
	} finally {
		DataSourceUtils.releaseConnection(c, dataSource);
		TransactionSynchronizationManager.unbindResource(this.dataSource);
		TransactionSynchronizationManager.clearSynchronization();
	}
}
```

```
<bean id="userService" class="com.sh.service.UserService">
	<property name="userDao" ref="userDao" />
	<proeprty name="dataSource" ref="dataSource" />
</bean>
```

**[참고]** 

```DataSourceUtils.getConnection(dataSource)``` : 스프링 유틸리티 메소드를 사용해서 Connection 오브젝트를 생성해주고 트랜잭션 동기화에 사용하도록 저장소에 바인딩해준다. 

```DataSourceUtils.releaseConnection(c, dataSource)``` : 스프링 유틸리티 메소드를 사용해서 DB 커넥션을 안전하게 닫는다. 

<br/>

> ### (!) JdbcTemplate
>
> JdbcTemplate은 ```update()``` ```query()``` 같은 JDBC 작업의 템플릿 메소드를 호출하면 직접 Connection을 생성하고 종료하는 일을 모두 담당한다. 
>
> #### 기능 정리
>
> - JDBC 코드의 try/catch/finally 작업 흐름 지원
> - SQLException의 예외 변환
> - **트랜잭션 처리** 
>   - 트랜잭션 동기화 저장소에 미리 생성되어 등록된 커넥션이 없으면 직접 커넥션을 만들고 트랜잭션을 시작해준다. 
>   - 이미 트랜잭션이 시작되어 있다면 동기화 저장소에 있는 커넥션을 가져와서 사용한다.

<br/>



### 트랜잭션 서비스 추상화

비즈니스 요구사항에 따라 각각 별도의 트랜잭션 관리 코드를 사용하는 경우가 존재한다. 예를 들어 로컬 트랜잭션으로 커버가 가능한 요구사항도 있지만, 만약 디비를 여러 개 사용해야하는 상황이라면 글로벌 트랜잭션을 고려해야 한다. 이 글로벌 트랜잭션을 필요로 하는 곳을 위해서는 JTA를 사용한 트랜잭션 관리 코드를 적용해야만 한다. 즉 서비스 로직이 바뀌지 않더라도 기술 환경에 따라 코드가 변경되어야 하는 문제가 생긴다. 

> #### 글로벌 트랜잭션이란
>
> 별도의 트랜잭션 관리자를 통해 트랜잭션을 관리하는 방식이다. 글로벌 트랜잭션을 적용하면 트랜잭션 매니저를 통해 여러 개의 DB가 참여하는 작업을 하나의 트랜잭션으로 만들 수 있다. 또 JMS와 같은 트랜잭션 기능을 지원하는 서비스도 트랜잭션에 참여시킬 수 있다. 
>
> - 자바에서는 글로벌 트랜잭션을 지원하는 트랜잭션 매니저를 지원하기 위한 API로 **JTA (Java Transaction API)** 를 제공한다. 

<br/>



#### UserService 코드가 특정 트랜잭션 방식에 의존하지 않으려면 "추상화"를 고려해야 한다. 스프링은 트랜잭션 기술의 공통점을 담은 트랜잭션 추상화 기술을 제공한다. (일관된 방식으로 트랜잭션을 제어하는 트랜잭션 경계설정 작업 가능)

<br/>

(그림 5-6)

<br/>

> #### PlatformTransactionManager
>
> 스프링이 제공하는 트랜잭션 경계설정을 위한 추상 인터페이스

<br/>

```
public void upgradeLevels() {
	PlatformTransactionManager transactionManager = 
										new DataSourceTransactionManager(dataSource); 
	
	TransactionStatus status = 
										transactionManager.getTransaction(new DefaultTransactionDefinition());
										
	try {
		//트랜잭션 안에서 진행되는 작업(코드 생략) 
		transactionManager.commit(status);
	} catch (RuntimException e) {
		transactionManager.rollback(status);
		throw e; 
	}
											
}
```

- JDBC 로컬 트랜잭션이 필요하면 PlatformTransactionManager를 구현한 DataSourceTransactionManager를 사용한다. 글로벌 트랜잭션으로 변경하려면 JTATransactionManager를 사용하면 된다. 
- ```getTransaction()``` : 트랜잭션을 가져오는 요청. 필요에 따라 트랜잭션 매니저가 디비 커넥션을 가져오는 작업도 같이 수행해준다. 
- PlatformTransactionManager로 시작한 트랜잭션은 동기화 저장소에 저장된다. 
- PlatformTransactionManager를 구현한 DataSourceTransactionManager 오브젝트는 JdbcTemplate에서 사용될 수 있는 방식으로 트랜잭션을 관리해준다. 따라서 PlatformTransactionManager를 통해 시작한 트랜잭션은 UserDao의 JdbcTemplate 안에서 사용된다. 
- 트랜잭션 작업을 모두 수행한 후에는 트랜잭션을 만들 때 돌려받은 TransactionStatus 오브젝트를 파라미터로 해서 ```commit()``` ```rollback()``` 메소드를 수행한다. 

<br/>



#### 트랜잭션 기술 설정의 분리가 필요하다. 사용할 트랜잭션 매니저 구현 클래스는 컨테이너를 통해 외부에서 제공받게 하는 스프링 DI 방식으로 선언해야 한다. 

> **[참고]** 스프링이 제공하는 모든 PlatformTransactionManager 구현 클래스는 싱글톤으로 사용이 가능하므로 스프링의 싱글톤 빈으로 등록해도 괜찮다. 

<br/>

```
public class UserService {
	...
	private PlatoformTransactionManager transactionManager; 
	public void setTransactionManager(PlatformTransactionManager transactionManager) {
		this.transactionManager = transactionManager; 
	}
	
	public void upgradeLevels() {
		...
	}
	...
}
```

```
<bean id="userService" class="com.sh.service.UserService">
	<property name="userDao" ref="userDao" />
	<property name="transactionManager" ref="transactionManager" />
</bean>

<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
	<property name="dataSource" ref="dataSource" />
</bean>
```

<br/>

> #### 단일 책임 원칙 (Single Responseibility Principle)
>
> 하나의 모듈은 한 가지 책임을 가져야 한다. 어떤 변경이 필요할 때 수정 대상이 명확해지는 장점이 있다. 

<br/>

### 메일 서비스 추상화

메일 전송 로직 구현 시에도 추상화가 필요하다. 

단지 테스트일 뿐인데 실제 메일 서버로 메일을 전송하는 것은 서버에 부담이 갈 수 있다. 테스트 중에는 JavaMail 대신 테스트에서 사용할, JavaMail과 같은 인터페이스를 갖는 오브젝트를 만들어서 사용하는 것이 좋다. 

스프링에서는 JavaMail에 대한 추상화 기능을 제공한다. 

```
package org.springframework.mail;
...
public interface MailSender {
	void send(SimpleMailMessage simpleMessage) throws MailException;
	void send(SimpleMailMessage[] simpleMessages) throws MailException;
}
```

   
