# 2. 테스트

"스프링이 개발자에게 제공하는 가장 중요한 가치는 객체지향과 테스트다. "라고 토비가


## 2.1 UserDaoTest 다시 보기

### 2.1.1 테스트의 유용성 

테스트란 내가 예상하고 의도했던 대로 코드가 정확히 동작하는 지를 확인해서, 만든 코드를 확실할 수 있게 해주는 작업이다. 



### 2.1.2 UserDaoTest의 특징

#### 웹을 통한 DAO 테스트 방법의 문제점

UI부터 비즈니스 로직을 전부 다 구현한 후에야 테스트가 가능해서 번거롭기도 하고

이렇게 되면 어디서 문제가 발생했는 지 콕 찝어내는 것도 힘들다



#### 작은 단위의 테스트

<h4>테스트는 가능하면 작은 단위로 쪼개서 하나의 대상에 집중해서 할 수 있는 것이 좋다!</h4>

> 단위테스트란, 작은 단위의 코드에 대해 테스트를 수행한 것을 의미한다. 단위는 작을수록 좋고. 확인의 대상과 조건이 간단하고 명확할수록 좋다. 



#### 자동수행 테스트 코드

<h4>테스트는 자동으로 수행되도록 코드로 만들어지는 것이 좋다</h4> 

이 때 UserDaoTest 처럼 애플리케이션을 구성하는 클래스 안에 테스트코드를 포함시키는 것보다는, 별도로 테스트용 클래스를 만들어서 테스트코드를 넣는 편이 낫다. 



#### 지속적인 개선과 점진적인 개발을 위한 테스트

어쨌든 전부다 만들고나서 테스트하려고 하지 말 것. 코드 만들고 테스트 만들고를 반복하며 작은 단위로 테스트하는 것이 현명



### 2.1.3 UserTest의 문제점

- 얘는 결과를 찍기만한다. 결과가 원하는대로 나오는 지 개발자가 확인해야 함
- 매번 main 메소드를 실행하는 것은 번거롭다

#

## 2.2 UserDaoTest 개선

위에 두 문제점을 개선해보자,

### 2.2.1 테스트 검증의 자동화

테스트 결과를 프린트하는 부분을 검증 코드로 수정하기

```
if (!user.getName().equals(user2.getName()) {
	System.out.println("테스트 실패 (name)");
}
else if (!user.getPassword().equals(user2.getPassword())) {
	System.out.println("테스트 실패 (password)")
}
else {
	System.out.println("조회 테스트 성공")
}
```



### 2.2.2 테스트의 효율적인 수행과 결과 관리

main() 메소드를 이용한 테스트 작성 방법만으로는 애플리케이션 규모가 커지고 테스트 개수가 많아지면 테스트 수행하는 일이 부담이 될 것이다. 많은 테스트를 간단히 실행시킬 수 있으며, 테스트 결과를 종합해서 볼 수 있고, 테스트가 실패한 곳을 빠르게 찾을 수 있는 기능을 갖춘 테스트 지원도구와 그에 맞는 테스트 작성 방법이 필요하다. 자바의 테스트 도구 중에 JUnit이 있다. 자바 테스팅 프레임워크다. 



#### JUnit 테스트로 전환

JUnit은 프레임워크다! 프레임워크에서 동작하는 코드는 main() 메소드도 필요없고 오브젝트를 만들어서 실행시키는 코드를 만들 필요도 없다. 



#### 테스트 메소드 전환

앞의 예제에서 테스트가 main() 메소드로 만들어진 것은 '제어권을 직접 갖는다'는 의미이기 때문에 프레임워크에 적합하지 않다. JUnit 프레임워크가 요구하는 조건을 충족하는 일반 메소드로 변환이 필요하다. 

<B>요구하는 조건</B>

- 메소드가 public으로 선언되어야 한다.
- 메소드에 @Test 어노테이션을 붙여줘야 한다. 



#### 검증 코드 전환

테스트 결과를 검증하는 if/else 문장을 JUnit에서 제공하는 assertThat 스태틱 메소드로 변경한다. 

> assertThat() 메소드는 첫 번째 파라미터 값을 뒤에 나오는 매처라고 불리는 조건으로 비교해서 일치하면 다음으로 넘어가고, 아니면 테스트가 실패하도록 만들어준다. is()는 매처의 일종으로 equals()로 비교해주는 기능을 가졌다. JUnit은 예외가 발생하거나 assertThat()에서 실패하지 않고 테스트 메소드의 실행이 완료되면 테스트가 성공했다고 인식한다. assertThat()을 이용해 검증을 했을 때 기대한 결과가 아니면 AssertionError를 던진다. 



- JUnit을 적용한 UserDaoTest

```
import static org.hamcrest.CoreMatchers.is;
import static org.junit.Assert.assertThat;
...
public class UserDaoTest {

	@Test
	public void addAndGet() throws SQLException {
		ApplicationContext context = new GenericXmlApplicationContext("applicationContext.xml");
		UserDao dao = context.getBean("userDao", UserDao.class);
		User user = new User(); 
		user.setId("itesh");
		user.setName("켱");
		user.setPassword("zudzud");
		
		dao.add(user);
		
		User user2 = dao.get(user.getId());
		assertThat(user2.getName().is(user.getName()));
		assertThat(user2.getPassword().is(user.getPassword()));
	}
	
}
```



#### JUnit 테스트 실행

JUnit은 프레임워크이기 때문에 JUnit을 시작시켜줘야 한다. main() 메소드에 JUnit을 시작시켜주는 클래스인 JUnitCore 클래스를 호출해주자

```
import org.junit.runner.JUnitCore;
...
public static void main(String[] args) {
	JUnitCore.main("com.sh.dao.UserDaoTest");
}
```

> org.junit.runner.JUnitCore
>
> - 테스트케이스는 JUnitCore 클래스를 사용해서 실행된다. 
> - JUnitCore는 테스트를 실행하기 위한 퍼사드(facade) 역할을 한다. 
> - JUnit4 테스트, JUnit 3.8.x 테스트 및 혼합 실행을 지원한다. 

#

## 2.3 개발자를 위한 테스팅 프레임워크 JUnit

스프링의 핵심 기능 중 하나인 스프링 테스트 모듈도 JUnit을 이용한다. 



### 2.3.1 JUnit 테스트 실행 방법

자바 IDE에 내장된 JUnit 테스트 지원도구를 사용한다. 

#### IDE

- 자바 IDE인 이클립스는 JUnit 테스트를 지원하는 기능을 제공한다. @Test가 들어있는 테스트 클래스를 선탤한 뒤에 Run As > JUnit Test를 선택하면 테스트가 자동으로 실행된다.
- JUnit은 한 번에 여러 테스트 클래스를 동시에 실행할 수도 있다. 패키지 우클릭해서 JUnit Test 실행! 



#### 빌드 툴

- 빌드 툴에서 제공하는 JUnit 플러그인이나 테스크를 이용해서 JUnit 테스트를 실행할 수 있다. 



### 2.3.2 테스트 결과의 일관성

UserDaoTest에는 유저 데이터가 중복되지 않도록 USER 테이블을 지워주는 사전작업이 필요했다. 즉, 별도의 준비작업 없이는 성공해야하는 테스트가 실패할 수도 있다. 

#### deleteAll()의 getCount() 추가

- 테스트가 시작될 때 deleteAll()을 통해 모든 USER 데이터를 삭제하고
- deleteAll()이 기대한대로 동작하는 지 확인하기 위해 getCount()를 적용하자.

```
@Test
public void addAndGet() throws SQLException {
	...
	dao.deleteAll();
	assertThat(dao.getCount(), is(0));
	
	User user = new User();
	user.setId("itesh");
	user.setName("켱");
	user.setPassword("zudzud");
	
	dao.add(user);
	assertThat(dao.getCount(), is(1));
	
	User user2 = dao.get(user.getId());
	
	assertThat(user2.getName(), is(user.getName()));
	assertThat(user2.getPassword(), is(user.getPassword()));
}
```



#### 동일한 결과를 보장하는 테스트

- 단위 테스트는 항상 일관성 있는 결과가 보장돼야 한다. 
  - DB에 남아있는 데이터와 같은 외부 환경에 영향을 받지 말아야 하고
  - 테스트 실행 순서를 바꿔도 동일한 결과가 나와야 한다



### 2.3.3 포괄적인 테스트

#### getCount() 테스트

User를 더 추가해서 갯수를 제대로 세는 지 검증을 더 많이

> JUnit 테스트 주의점!
>
> JUnit은 특정한 테스트 메소드의 실행 순서를 보장해주지 않는다. 테스트의 결과가 테스트 실행 순서에 영향을 받는다면 테스트를 잘못 만든 것이다. 



#### addAndGet() 테스트 보완

get()으로 가져오는 정보가 그 정보가 맞는지 더더더 확인



#### get() 예외조건에 대한 테스트 

get() 메소드에서 쿼리를 실행해 결과를 가져왔을 때 아무것도 없으면 예외를 던지도록 만들어보자,



예외가 던져지지 않고 정상적으로 작업을 마치면 테스트가 실패했다고 판단해야 한다. 예외 발생 여부는 메소드를 실행해서 리턴 값을 비교하는 방법으로 확인이 불가능하다. 

<h4>JUnit은 예외조건 테스트를 위한 특별한 방법을 제공해준다</h4>

```
@Test(expected=EmptyResultDataAccessException.class)
public void getUserFailure() throws SQLExeption {
	ApplicationContext context = new GenericXmlApplicationContext("applicationContext.xml");
	
	UserDao dao = context.getBean("userDao", UserDao.class);
	dao.deleteAll();
	assertThat(dao.getCount(), is(0));
	
	dao.get("unknown_id"); //여기서 예외발생해야 함
}
```

- Expected: 테스트 메소드 실행 중에 발생하리라 기대하는 예외 클래스를 넣어준다. 정상적으로 테스트 메소드를 마치면 테스트가 실패하고 expected에서 지정한 예외가 던져지면 테스트가 성공한다. 
  - 예외가 반드시 발생해야하는 경우를 테스트하고자 할 떄 유용하다. 



#### 테스트를 성공시키기 위한 코드의 수정

위 테스트가 성공하도록 get을 수정해보자

```
public User get(String id) throws SQLException {
	...
	ResultSet rs = ps.excuteQuery();
	
	User user = null;
	if (rs.next()) {
		user = new User();
		user.setId(rs.getString("id"));
		user.setName(rs.getString("name"));
		user.setPassword(rs.getString("password"));
	}
	
	rs.close();
	ps.close();
	c.close();
	
	if (user == null) throws new EmptyResultDataAccessException(1);
	
	return user; 
}
```



#### 포괄적인 테스트

신경을 써서 내 코드에서 발생할 수 있는 다양한 상황과 입력 값을 고려하는 테스트를 만들어야 한다. 

<h4>테스트를 작성할 때 부정적인 케이스를 먼저 만들자!</h4> 예를 들어, 존재하는 아이디가 주어졌을 때 동작보다는 존재하지 않는 아이디가 주어졌을 때 어떻게 반응할 지를 결정하고 테스트하는 것을 먼저하자. 



### 2.3.4 테스트가 이끄는 개발

#### 기능설계를 위한 테스트

테스트에는 만들고 싶은 기능에 대한 조건과 행위, 결과에 대한 내용이 잘 포현되어 있다. 그래서 보통 기능설계, 구현, 테스트라는 일반적인 개발 흐름의 기능설계에 해당하는 부분을 이 테스트 코드가 일부분 담당하고 있다고 볼 수도 있다. 



#### 테스트 주도 개발 (TDD. Test Driven Development)

"만들고자 하는 기능의 내용을 담고 있으면서 만들어진 코드를 검증도 해줄 수 있도록 테스트 코드를 먼저 만들고, 테스트를 성공하게 해주는 코드를 작성하는 방식의 개발 방법" "테스트 우선 개발" 

- 테스트를 먼저 만들고 그 테스트가 성공하도록 하는 코드만 만드는 식으로 진행하기 떄문에 테스트를 빼먹지 않고 꼼꼼하게 만들어낼 수 있다.
- 코드를 만들어 테스트를 실행하는 그 사이의 간격이 매우 짧다. 코드를 작성하면 바로바로 테스트 가능. 코드에 있는 오류를 빠르게 발견할 수 있다. 



<h4>테스트는 코드를 작성한 후에 가능한 빨리 실행할 수 있어야 한다. </h4> 테스트 없이 한 번에 너무 많은 코드를 만들지 말자. 이게 불편하다면 일정 분량의 코딩을 먼저 해놓고 빠른 시간 안에 테스트 코드를 만들어 테스트해도 상관없다. 



### 2.3.5 테스트 코드 개선

#### @Before

> 테스트를 실행할 때마다 반복되는 준비작업을 별도의 메소드에 넣게 해주고 이를 매번 테스트 메소드를 실행하기 전에 실행시켜주는 기능



<h4>JUnit 프레임워크가 테스트 메소드를 어떻게 실행하는가?</h4>

1. 테스트 클래스에서 @Test가 붙은 public이고 void형이며 파라미터가 없는 테스트 메소드를 모두 찾는다. 
2. 테스트 클래스의 오브젝트를 하나 만든다. 
3. @Before가 붙은 메소드가 있으면 실행한다.
4. @Test가 붙은 메소드를 하나 호출하고 테스트 결과를 저장해둔다.
5. @After가 붙은 메소드가 있으면 실행한다.
6. 나머지 테스트 메소드에 대해 2~5번을 반복한다.
7. 모든 테스트의 결과를 종합해서 돌려준다.



> JUnit은, 
>
> - @Test가 붙은 메소드를 실행하기 전과 후에 각각 @Before와 @After가 붙은 메소드를 자동으로 실행한다. 이것들을 테스트 메소드에서 직접 호출하지 않기 떄문에 서로 주고받을 정보나 오브젝트가 있다면 인스턴스 변수를 이용해야 한다. 
> - 각 테스트 메소드를 실행할 때마다 테스트 클래스의 오브젝트를 새로 만든다. 한번 만들어진 테스트 클래스의 오브젝트는 하나의 테스트 메소드를 사용하고 나면 버려진다. 
>   - 각 테스트가 서로 영향을 주지 않고 독립적으로 실행될 수 있도록 매번 새로운 오브젝트를 만들게 함. 따라서 인스턴스 변수도 부담없이 사용할 수 있다. 다음 테스트 메소드가 실행되면 초기화되니까
> - 테스트 메소드의 일부에서만 공통적으로 사용되는 코드가 있다면 @Before를 이용하기보다는, 일반적인 메소드 추출 방법을 써서 메소드를 분리하고 테스트 메소드에서 직접 호출해 사용하도록 만드는 편이 낫다. 아니면 아예 공통적인 특징을 지닌 테스트 메소드를 모아서 별도의 테스트 클래스로 만드는 방법도 있다. 



#### 픽스처

> 픽스처란 테스트를 수행하는 데 필요한 정보나 오브젝트를 의미한다. 

#

## 2.4 스프링 테스트 적용

테스트는 가능한 한 매번 새로운 오브젝트를 생성해서 사용하는 것이 원칙이지만 경우에 따라서는 테스트 전체가 공유해서 사용할 수 있는 오브젝트를 만들기도 한다. 이를 위해 JUnit에서는 @BeforeClass 스태틱 메소드를 지원한다. 이것은 테스트 클래스 전체에 걸쳐 단 한번만 실행된다. 이 때, 대상이 애플리케이션 컨텍스트인 경우에는 스프링이 제공하는 지원 기능을 사용하는 것이 더 편리하다. 



### 2.4.1 테스트를 위한 애플리케이션 컨텍스트 관리

스프링은 간단한 어노테이션 설정을 통해 모든 테스트에서 애플리케이션 컨텍스트를 공유할 수 있도록 해준다. 



#### 스프링 테스트 컨텍스트 프레임워크 적용

- ApplicationContext 타입의 인스턴스 변수를 선언하고, @Autowired 어노테이션을 붙여준다. 
- 클래스 레벨에 @RunWith와 @ContextConfiguration 어노테이션을 추가한다. 



#### 테스트 메소드의 컨텍스트 공유

"테스트 클래스 안의 테스트 메소드들은 애플리케이션 컨텍스트를 공유한다. "

>  스프링의 JUnit 확장기능은 테스트가 실행되기 전에 단 한 번만 애플리케이션 컨텍스트를 만들어두고, 테스트 오브젝트가 만들어질 때마다 특별한 방법을 이용해 애플리케이션 컨텍스트 자신을 테스트 오브젝트의 특정 필드에 주입해준다. 



#### 테스트 클래스의 컨텍스트 공유

"모든 테스트 클래스들도 애플리케이션 컨텍스트를 공유한다. "



#### @Autowired

스프링의 DI에 사용되는 어노테이션

- 변수에 "할당 가능한" 타입을 가진 빈을 자동으로 찾는다. 따라서 특정 클래스의 인터페이스 타입으로 선언해도 된다. 

  (*) 테스트에서도 가능한 한 타입을 인터페이스로 선언해서 코드를 느슨하게 해두는 것이 좋다.

- 단, 같은 타입의 빈이 두 개 이상 존재하는 경우에는, 변수의 이름을 보고 이름이 동일한 빈을 가져온다. 

- 변수 이름으로도 찾을 수 없는 경우에는 예외가 발생한다. 



<h4>애플리케이션 컨텍스트를 빈으로 가져올 수 있었던 이유?</h4>

스프링 애플리케이션 컨텍스트는 초기화할 때 자신도 빈으로 등록한다. 즉, 애플리케이션 컨텍스트에는 ApplicationContext 타입의 빈이 존재하는 것이고 DI도 가능한 것이다. 



### 2.4.2 DI와 테스트

TODO

#

## 2.5 학습 테스트로 배우는 스프링

<B>학습 테스트란?</B> 

내가 만들지 않은 프레임워크나 라이브러리에 대해서 테스트를 작성해보는 것

목적은 사용할 API나 프레임워크의 기능을 테스트로 보면서 사용방법을 익히고, 테스트 코드를 작성하면서 빠르고 정확하게 사용법을 익히기 위함

