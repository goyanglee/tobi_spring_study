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
