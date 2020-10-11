스프링의 3대 기반 기술

1. AOP
2. IoC / DI
3. 서비스 추상화 

AOP의 적용 대상 중 가장 인기 있는 기술은 선언적 트랜잭션 기능이다.

## 트랜잭션 코드

### 메소드 분리

기존 코드를 살펴보면 트랜잭션 시작, 비즈니스 로직, 트랜잭션 종료 코드가 존재한다. 이 코드는 트랜잭션과 비즈니스 로직 간에 주고 받는 정보가 없으므로 완벽히 독립적인 코드이다.

```java
public void upgradeLevels() throws Exception {
TransactionStatus status = this.transactionManager.getTransaction(new DefaultTransactionDefinition());

try{ 
비즈니스 코드()
}catch{롤백~}

private void 비즈니스 코드() {
	비즈니스 코드~
}
}
```

### DI를 이용해서 클래스 분리

기존 코드를 살펴보면 UserService 클래스 구현체에서 트랜잭션 코드를 밖으로 빼버리면 UserService를 직접적으로 사용하고 있는 곳에서는 트랜잭션 코드가 제외된 UserService를 사용하게 된다.  직접 사용하는 것의 단점이다. 이럴 때에는 트랜잭션 코드를 밖으로 뺀 후 인터페이스 두개를 확장한다.

### UserService 인터페이스 도입

```java
public class UserServiceImpl implements UserService {

@Override
public void upgradeLevels() {
	비즈니스 코드 ~
}
}
```

### 분리된 트랜잭션

```java
public class UserServiceTx implements UserService {
UserService userService;
PlatformTransactionManager transactionManager;

@Override
public void add(User user){}

public void upgradeLevels() {
TransactionStatus status = this.trancationManager.getTransaction(new ~());
비즈니스 코드();
commit();
}

}
```

비즈니스 코드를 갖지 않으면서 UserService 를 DI를 받아 구현 오브젝트에 기능을 위임한다. 

### 테스트

```java
테스트 extends UserServiceImpl{
TestUserService testUserService = new TestUserService(users.get(3).getId());
info 세팅

UserServiceTx uxUserService = new UserServiceTx();
txUserService.setTransactionManager(transactionManager);
txUserService.setUserService(testUserService);

비즈니스 코드();
}
```

이렇게 관심사를 바깥으로 뺐을 때의 장점

1. 추 후 UserServiceImpl 의 코드를 작성할 때 트랜잭션과 같은 기술적인 내용은 신경쓰지 않아도 된다.
2. 비즈니스 로직에 대한 테스트를 손쉽게 만들어낼 수 있다.

## 고립된 단위 테스트

이전대로 하면 의존성을 가진 서비스를 테스트하면 테스트가 진행되는 동안 의존성을 가진 것들이 모두 실행이 된다. 더 많은 오브젝트 뿐만 아니라 네트워크까지 테스트를 하는 셈이다. 셋업되어있지 않으면 전부 실패하는 문제가 생긴다. 

#### 테스트 대상 오브젝트 고립시키기

테스트를 위한 대역을 사용하기. 트랜잭션 같은 경우는 이미 독립을 했기 때문에 메일링 서비스와 유저 제어 서비스만 테스트를 하면 된다.  다만 부가적인 검증 기능까지 가진 목 오브젝트로 만들어서 고립된 환경에서 동작하는 테스트 결과를 검증한다. void 라 데이터베이스에 직접 접근해서 결과를 확인하는 수밖에 없던 테스트 같은 경우에는 메소드를 호출했는지를 확인한다.

사용하지 않은 메소드를 구현해야하는 경우에는 UnSupportedOperationException을 던지거나 null로 return 하도록 한다. 다만 전자로 하는게 더 좋다.

데이터베이스에 저장할 목 오브젝트는 메모리에 가지고 있다가 돌려주는 형식으로 테스트한다.

기존에는 스프링 컨테이너에서 빈을 가져와 사용했지만, 고립된 테스트이기 때문에 그럴 필요가 없다.

```java
public void upgradeTest9) throws Exception{
MockUserDao mockUserDao = new MockUserDao(this.users);
userServiceImpl.setUserDao(mockUserDao); //목 오브젝트로 만든 userdao를 직접 di 해준다.

}
```

#### 단위 테스트와 통합 테스트 

단위 테스트 : 외부 리소스를 사용하지 않도록 고립시켜서 테스트하는 것.

통합 테스트 : 두 개 이상의 성격이나 계층이 다른 오브젝트가 연동하도록 만들어 테스트하거나 외부 데이터 베이스나 파일, 서비스 등의 리소스를 참조해서 하는 테스트.

어떤 방법을 쓸 까?

1. 단위테스트를 먼저 고려한다.
2. 외부 리소스를 사용해야하면 통합테스트로 만든다.
3. dao같은 코드는 단위 테스트로 만들기 어렵다.
4. 데이터베이스라는 외부 리소스를 사용하는 경우는 통합 테스트로 분류한다.
5. 여러 의존관계를 갖고 동작하면 통합 테스트이다.
6. 단위 테스트로 만들기 복잡한 코드는 통합 테스트이다.
7. 스프링 테스트 컨텍스트 프레임워크를 사용하는 경우는 통합 테스트이다.

### 목 프레임워크

#### Mockito 프레임워크

```java
UserDao mockUserDao = mock(UserDao.class);
when(mockUserDao.getAll()).thenReturn(this.users);
verify(mockUserDao,times(2)).update(any(User.class));
```

Mockito 목 오브젝트는 다음의 네 단계를 거쳐서 사용한다.

1. 인터페이스를 이용해 목 오브젝트를 만든다.
2. 목 오브젝트가 리턴할 값이 있으면 지정한다.
3. 테스트 대상 오브젝트에 di 한다.
4. 목 오브젝트의 특정 메소드가 호출됐는지, 몇번 호출됐는지 검증한다.

## 다이나믹 프록시와 팩토리 빈

### 프록시와 프록시 패턴

핵심기능과 부가기능이 있던 기존의 패턴에서 부가기능을 제외시키고 난 후에는 부가기능이 핵심 기능을 사용하는 형태가 되어버렸다. 그래서 핵심기능을 사용할 때 부가기능이 사용되지 않는다. 이 점을 해결하기 위해서 클라이언트는 인터페이스로만 핵심기능을 사용할 수 있도록 해야한다. 그러면 클라이언트는 핵심기능을 사용하는 것 처럼 보이지만 사실 부가기능이 핵심 기능을 사용하는 형태는 변함이 없다. 바로 이 과정이 비즈니스 코드에 트랜잭션 기능을 부여해주는 것과 같다. 

**프록시** : 대리인처럼 위장해서 클라이언트의 요청을 받아줌.

**타깃/실체** : 요청을 위임받아 처리하는 실제 오브젝트

![그림6-10](url)

#### 목적

1. 클라이언트가 타깃에 접근하는 방법을 제어하기 위해
2. 타깃에 부가적인 기능을 부여해주기 위해

### 데코레이터 패턴

타깃에 부가적인 기능을 런타임 시 다이나믹하게 부여해주기 위해 프로시를 사용하는 패턴. 컴파일 시점에서는 어떤 방법과 순서로 연결되어 사용되는지는 모르고, 런타임 시에 알 수 있다. 제품을 여러 겹으로 포장하고 그 위에 장식을 붙이는 것처럼 실제 내용물은 동일하지만 부가적인 효과를 줄 수 있기 때문에 데코레이터 패턴이라 이름 붙였다. 따라서 프록시가 꼭 한 개로 제한하지 않는다. 

![그림6-11](url)

위임하는 대상에도 인터페이스로 접근해서 최종 타깃으로 위임하는지 다음단계 프록시인지 모른다. 대표적인 예로 **InputStream** 과 **OutputStream** 이 있다.

우리가 한 설정 중에 UserService 빈은 데코레이터다. UserService 타입의 오브젝트를 di 받아서 위임하고 트랜잭션 경계설정 기능을 부여한다. 인터페이스를 통해 위임하는 방식이다.

### 프록시 패턴

프록시를 사용하는 방법 중에서 타깃에 대한 접근 방법을 제어하려는 목적을 가진 경우다. 타깃의 기능을 확장하거나 추가하지 않고 클라이언트가 타깃에 접근하는 방식을 변경한다. 클라이언트에게 타깃에 대한 레퍼런스를 넘길 때 프록시를 대신 넘겨주고, 프록시의 메소드를 통해 타깃을 제어하려고 하면 프록시가 타깃 오브젝트를 생성하고 요청을 위임해준다. 

#### 장점

1. 레퍼런스는 갖고 있지만 늦게 사용되는 경우에 생성을 늦출 수 있다.
2. 원격 오브젝트를 사용할 때 편리하다.
3. 타깃에 대한 접근권한을 제어할 수 있다. 

### 다이나믹 프록시

#### 기능

1. 타깃과 같은 메소드를 구현하고 있다가 메소드가 호출되면 타깃 오브젝트로 위임한다.
2. 지정된 요청에 대해서는 부가기능을 수행한다.

```java
public class UserServiceTx implements UserService {
	UserService userService;
...
	public void add(User user){
		userService.add(user); //메소드 구현과 위임 
	}
	public void upgradeLevels() {
		TransactionStatus status = this.transactionoManager.getTransaction(new DefaultTransactionDefinition());//부가기능 수행 
	}

}
```

#### 문제점

1. 메소드가 변경될 때마다 함께 수정해줘야한다.
2. 부가기능 코드가 중복될 가능성이 많다.

### 해결 : 리플렉션

자바의 코드 자체를 추상화해서 접근하도록 만든 것.

```java
Method lengthMethod = String.class.getMethod("length");
//string이 가진 메소드 중에서 length라는 이름을 갖고 있고 파라미터는 없는 메소드의 정보를 가지고 온다.
int length = lengthMethod.invoke(name);
//실행시킬 대상 오브젝트와 파라미터 목록을 받아 메소드를 호출한 뒤 objec 타입으로 돌려준다.
```

### 프록시 클래스

```java
interface Hello {
	String sayHello(String name);
}

public class HelloTarget implements Hello {
	public String sayHello(String name) {
		return "Hello" + name;
	}
}

public class HelloUpperCase implements Hello {
	Hello hello;
	public HelloUpperCase(Hello hello){
		this.hello = hello;
	}
	public String sayHello(String name) {
		return hello.sayHello(name).toUpperCase();
	}
}

@Test
public void proxy() {
	Hello hello = new HelloTarget();
	hello.sayHello("Toby")//Hello Toby
	Hello proxyHello = new HelloUpperCase(new HelloTarget());
	proxyHello.sayHello("Toby"); //HELLO TOBY
}
```

![그림6-13](url)

**다이나믹 프록시** : 런타임 시 프록시 팩토리에 의해 다이나믹하게 만들어지는 오브젝트. 타깃의 인터페이스와 같은 타입으로 만들어진다. 

![그림6-14](url)

```java
public class UpperCaseHandler implements InvocationHandler {
	Hello terget;
	public UpperCaseHandler(Hello target) {
			this.target = target;//타깃 오브젝트 주입
	}
	public Object invoke(Object proxy, Method method, Object[] args) {
		String ret = (String) method.invoke(target, args);//타깃으로 위임. 
		return ret.toUpperCase();
	}
}
```

다이나믹 프록시는 클라이언트로부터 받는 모든 요청을 invoke() 메소드로 전달하고, 리플렉션 API 를 이용해 타깃 오브젝트의 메소드를 호출해서 리턴값을 반환한다.

```java
Hello proxiedHello = (Hello) Proxy.newProxyInstance (
getClass().getClassLoader(),//다이나믹 프록시 클래스의 로딩에 사용할 클래스 로더
new Class[] {Hello.class},//구현할 인터페이스
new UpperCaseHandler(new HelloTarget()));//부가기능과 위임 코드를 담은 invocationHandler
)
```

다이나믹 프록시 방식이 직접 정의한 프록시보다 유연하다.

1. 메소드가 늘어날 떄마다 다이나믹 프록시는 만들어질때 자동으로 포함되기 때문에 편리하다.

## 트랜잭션 부가기능을 다이나믹 프록시로 구현

```java
public class TransactionHandler implements InvocationHandler {

	private Object target;
	private PlatformTransactionManager transactionManager;
	private String pattern;
	
	//target setting
	//transactionManager setting
	//pattern setting
	
	/**
	* 트랜잭션 적용 대상 메소드 선별  
	**/
	public Object invoke(Object proxy, Method method, Object[] args) {
		//메소드 이름 판별 후
		return invokeInTransaction(method, args);
		return method.invoke(target, args);
	}

	public Object invokeInTransaction(Method method, Object[] args) {
		TransactionStatus status = this.transactionManager.getTransaction(new DefaultTransacionDefinition());
		Object ret = method.invoke(target, args); //트랜잭션 시작 후 타깃 오브젝트의 메소드 호출
		this.transactionManager.commit(status);
		return ret;
	}
}
```

### 다이나믹 프록시를 위한 팩토리 빈

프록시를 일반적인 스프링의 빈으로 등록할 수 없고, 아래와 같은 방법을 사용한다.

```java
Date now = (Date) Class.forName("java.util.Date").newInstance();
```

스프링은 내부적으로 리플렉션 API를 이용해서 클래스 이름을 가지고 빈 오브젝트를 생성한다. 다만 다이나믹 프록시는 Proxy 클래스의 newProxyInstance() 라는 스태틱 팩토리 메소드를 통해서만 만들 수 있다.

#### 팩토리 빈

```java
public interface FactoryBean<T> {
	T getObject() throws Exception;//빈 오브젝트 생성 후 반환
	Class<? extends T> getObjectType();//생성되는 오브젝트의 타입
	boolean isSingleton();//오브젝트가 싱글톤인지 확인
}
```
