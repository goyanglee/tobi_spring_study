# 6장 AOP

<br/>

*- 선언적 트랜잭션 기능에 AOP 기술 적용*

*- 스프링이 AOP를 도입한 이유*

<br/>

## 6.1 트랜잭션 코드의 분리

#### 5장 트랜잭션이 적용된 ```upgradeLevels()``` 코드의 문제점 

- 비즈니스 로직코드와 트랜잭션 경계설정 코드가 구분되어 함께 존재함
- 이 두 가지 타입의 코드 간에 주고받는 정보가 없음

<br/>

#### 문제해결

- 비즈니스 로직과 트랜잭션 경계설정 로직을 분리한다. (메소드 분리)

- 더 나아가서 둘 간의 주고받는 정보가 없으므로 클래스 외부로 아예 분리한다. (클래스 분리)

  > #### 트랜잭션 경계설정을 위한 클래스 도입
  >
  > ![그림6-3](https://github.com/goyanglee/tobi_spring_study/blob/master/Vol.1%20%EC%8A%A4%ED%94%84%EB%A7%81%EC%9D%98%20%EC%9D%B4%ED%95%B4%EC%99%80%20%EC%9B%90%EB%A6%AC/6.%20AOP/6%EC%9E%A5_sh/%E1%84%80%E1%85%B3%E1%84%85%E1%85%B5%E1%86%B76-3.jpeg)

<br/>

#### 그림 6-3 구조 만들기

1. **인터페이스 도입**

   ```
   public interface UserService {
   	void add(User user);
   	void upgradeLevels();
   }
   ```

   ```
   public class UserServiceImpl implements UserService {
   	UserDao userDao;
   	
   	public void upgradeLevels() {
   		List<User> users = userDao.getAll();
   		for (User user : users) {
   			if (canUpgradeLEvel(user)) {
   				upgradeLevel(user);
   			}
   		}
   	}
   	...
   }
   ```

   <br/>

2. **트랜잭션 로직을 담은 클래스 분리**

   ```
   public class UserServiceTx implements UserService {
   	UserService userService; 
   	PlatformTransactionManager transactionManager; 
   	
   	public void setTransactionManager(PlatformTransactioManager transactionManager) {
   		this.transactionManager = transactionManager; 
   	}
   	
   	public setUserService(UserService userService) {
   		this.userService = userService; 
   	}
   	
   	public void add(User user) {
   		this.userService.add(user);
   	}
   	
   	public void upgradeLevels() {
   		TransactionStatus status = this.transactionManager.getTransaction(new DefaultTransactionDefinition());
   		
   		try {
   		 	userService.upgradeLevels();
   		 	this.transactionManager.commit(status);
   		} catch (RuntimeException e) {
   		 	this.transactionManager.rollback(status);
   		 	throw e; 
   		}
   	}
   }
   ```

   -사용자관리 비즈니스 로직은 포함하지 않고 다른 ```UserService``` 구현 오브젝트에 기능을 위임한다. 

   -트랜잭션 경계설정이라는 부가적인 작업 로직만 추가되어 있다. 

   <br/>

3. **트랜잭션 적용을 위한 DI 설정**

   ```
   <bean id="userService" class="com.sh.service.UserServiceTx">
   	<property name="transactionManager" ref="transactionManager" />
   	<property name="userService" ref="userServiceImpl" />
   </bean>
   
   <bean id="userServiceImpl" class="com.sh.service.UserServiceImpl">
   	<property name="userDao" ref="userDao" />
   </bean>
   ```

<br/>

#### 정리

![그림6-4](https://github.com/goyanglee/tobi_spring_study/blob/master/Vol.1%20%EC%8A%A4%ED%94%84%EB%A7%81%EC%9D%98%20%EC%9D%B4%ED%95%B4%EC%99%80%20%EC%9B%90%EB%A6%AC/6.%20AOP/6%EC%9E%A5_sh/%E1%84%80%E1%85%B3%E1%84%85%E1%85%B5%E1%86%B76-4.jpeg)

- 클라이언트가 ```UserService``` 인터페이스를 통해 사용자 관리 로직을 이용하려 할 때
- 먼저 트랜잭션을 담당하는 오브젝트가 사용돼서 트랜잭션에 관련된 작업을 진행해주고
- 실제 사용자 관리 로직을 담은 오브젝트가 이후에 호출돼서 비즈니스 로직에 관련된 작업을 수행한다. 

<br/>

#### 코드 분리의 장점

- 비즈니스 로직을 포함하는 ```UserServiceImpl``` 코드를 작성할 때 트랜잭션과 같은 기술적인 내용을 고려안해도 된다. 
- 비즈니스 로직에 대한 테스트를 쉽게 작성할 수 있다. (6.2) 

<br/>

<br/>

## 6.2 고립된 단위 테스트

> 테스트는 작은 단위로 하는 것이 좋다. 하지만 테스트 대상이 다른 오브젝트와 환경에 의존하고 있따면 작은 단위의 테스트가 주는 장점을 얻기 힘들다. 

<br/>

- 테스트의 대상이 환경이나 외부 서버, 다른 클래스의 코드에 종속되고 영향을 받지 않도록 고립시킬 필요가 있다. 
- 그러기 위해서 **테스트 대역**을 사용하면 된다. 
- 고립된 테스트를 하면 테스트가 다른 의존 대상에 영향을 받을 경우를 대비해 복잡하게 준비할 필요가 없을 뿐만아니라 테스트 수행 성능도 크게 향상된다. 

<br/>

### `upgradeLevels()` 에서 ```UserDao```의 ```update()``` 메소드를 호출하는 부분에 대해 검증이 필요하다. 즉, ```update()``` 메소드에 대해서 목 오브젝트로서 동작하는 ```UserDao``` 타입의 테스트 대역이 필요하다. 

<br/>

#### 목 오브젝트 만들기

```
static class MockUserDao implements UserDao {
	private List<User> users; 
	private List<User> updated = new ArrayList();
	
	private MockUserDao(List<User>) users) {
		this.users = users;
	}
	
	public List<User> getUPdated() {
		return this.updated; 
	}
	
	public List<User> getAll() {
		return this.users; 
	}
	
	public void update(User user) {
		updated.add(user);
	}
	
	public void add(User user) { throw new UnsupportedOperationException(); }
	public void deleteAll() { throw new UnsupportedOperationException(); }
	public User get(String id) { throw new UnsupportedOperationException(); }
	public int getCount() { throw new UnsupportedOperationException(); }
}
```

- 두 개의 유저 타입 리스트 정의
  - ```users``` : 생성자를 통해 전달받은 사용자 목록을 저장해뒀다가 ```getAll()``` 메소드가 호출되면 DB에서 가져온 것처럼 돌려주는 용도로 사용된다. 미리 준비된 테스트용 리스트를 메모리에 갖고 있다가 돌려준다. 
  - ```updated``` : ```update()``` 메소드를 실행하면서 넘겨준 업그레이드 대상 유저 오브젝트를 저장해뒀다가 검증을 위해 돌려주기 위한 용도로 사용된다. ```upgradeLevels()``` 메소드가 실행되는 동안 업그레이드 대상으로 선정된 사용자가 어떤 것인 지 확인하는 데 사용된다. 

> [참고사항]
>
> - 인터페이스 구현이므로 존재하는 모든 메소드를 만들어줘야 한다. 사용하지 않는 메소드에 대해서는 ```UnsupportedOperationException```을 던지도록 만들어준다. (지원하지 않는 기능이라는 예외 전달)
> - ```UserServiceTest``` 전용이기 때문에 스태틱 내부 클래스로 만들면 편하다. 

<br/>

#### 검증 로직 구현하기

```
@Test
public void upgradeLevels() throws Exception {
	UserServiceImpl userServiceImpl = new UserServiceImpl();
	
	//준비해둔 목 오브젝트를 사용하도록 수동 di 
	MockUserDao mockUserDao = new MockUserDao(this.users);
	userServiceImpl.setUserDao(mockUserDao); 
	
	//고립된 테스트를 위한 준비 끝
	//테스트 대상인 UserServiceImpl 오브젝트의 메소드를 실행시킨다. 
	//로직에 따라 업그레이드 대상을 선정해서 레벨 변경 후 MockDaoUser의 update() 메소드를 호출하게 된다. 
	userServiceImpl.upgradeLevels();
	
	//검증시작
	List<User> updated = mockDao.getUpdated();
	assertThat(updated.size(), is(2));
	checkUserAndLevel(updated.get(0), "joytouch", Level.SILVER);
	checkUserAndLevel(updated.get(1), "madnitel", Level.GOLD); 
}

private void checkUserAndLevel(User updated, String expectedId, Level expectedLevel) {
	assertThat(updated.getId(), is(expectedId)); 
	assertThat(updated.getLevel(), is(expectedLevel));
}
```

<br/>

### 목 프레임워크

단위 테스트는 스텁이나 목 오브젝트 사용이 필수적이지만 목 오브젝트를 만드는 일은 번거롭다. 이렇 번거로운 목 오브젝트를 편하게 작성할 수 있도록 도와주는 다양한 목 오브젝트 지원 프레임워크가 있다. 그 중 하나가 **Mockito 프레임워크**이다. 

<br/>
