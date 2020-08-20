# 2장. 테스트

[https://www.notion.so/2-74d34ab67c0a48e8a6d4621322b19228](https://www.notion.so/2-74d34ab67c0a48e8a6d4621322b19228)

스프링 개발자에게 제공하는 가장 중요한 가치

: 객체지향과 테스트

# 2.1 UserDaoTest 다시보기

## 2.1.1 테스트의 유용성

테스트 해라!

테스트란, 결국 내가 예상하고 의도했던 대로 코드가 정확히 동작하는 지를 확인해서, 만든 코드를 확신할 수 있게 해주는 작업.

## 2.1.2 UserDaoTest의 특징

- main() 메소드 사용
- 직접 테스트할 대상인 UserDao를 불러서 사용하는 테스트 작성

### 웹을 통한 DAO 테스트 방법의 문제점

- 서비스 클래스, 컨트롤러, jsp 뷰 등 모든 레이어 기능을 만들고나서야 테스트가 가능하다.
- 에러가 나면 어디서 발생했는 지 수고가 필요하다

### 작은 단위의 테스트(Unit Test)

- "관심사의 분리" 라는 원리

### 자동수행 테스트 코드

테스트는 자동 으로 수행되도록 코드로 만들어지는 것이 중요하다. ⇒ 자주 반복할 수 있다.

### 지속적인 개선과 점진적인 개발을 위한 테스트

테스트를 안하면서 코드를 짜면 막판에 굉장히 질린다⇒그래서 내가질리나보다

## 2.1.3 UserDaoTest의 문제점

### 수동 확인 작업의 번거로움

출력 값 확인의 번거로움

### 실행 작업의 번거로움

아무리 간단히 실행 간단한 main() 메소드라해도, 매번 그것을 실행하는 것은 번거롭다. 전체 기능을 확인하기 우이해 수백번 실행해보는 수고가 필요하다. 

⇒ 좀 더 편리하고 체계적으로 테스트를 실행하고 그 결과를 확인하는 방법이 절실히 필요하다. 

# 2.2 UserDaoTest 개선

## 2.2.1 테스트 검증의 자동화

테스트 결과의 검증 부분을 자동화 해보자.

⇒ 객체 비교 출력

xUnit : 포괄적인 테스트는 개발자를 안심할 수 있게 한다.

## 2.2.2 테스트의 효율적인 수행과 결과 관리

테스트 도구 사용으로 단위 테스트 가능

### Junit 테스트로 전환

Junit은 테스트를 위한 프레임워크 

### 테스트 메소드 전환

1. 메소드는 public으로 선언 
2. @Test 애노테이션 사용

### 검증 코드 전환

- junit 라이브러리 추가  : com.springsoure.org.junit-4.7.0.jar

Junit 제공 assertThat 메소드 사용.

= 일치하면 다음으로 넘어가고, 아니면 test 실패 

is() : equals()로 비교해주는 기능

```java
assertThat(user2.getName(), is(user.getName());
```

### Junit 테스트 실행

@Test 가 붙은 메소드를 가진 클래스의 이름을 넣어준다.

```java
JunitCore.main("springbook.user.dao.UserDaoTest");
```

# 2.3 개발자를 위한 테스팅 프레임워크 Junit

### 2.3.1 JUnit 테스트 실행 방법

### IDE

이클립스 > run > Run As > JUnit Test

### 빌드 툴

ANT, Maven 하면 html이나 텍스트 파일의 형태로 보기 좋게 만들어진다.

## 2.3.2 테스트 결과의 일관성

### deleteAll()의 getCount() 추가

- deleteAll() : test 한 데이터는 전부 삭제해준다.
- getCount(): User 테이블의 레코드 개수를 돌려준다.

### deleteAll() 과 getCount()의 테스트

```java
dao.deleteAll()l
assertThat(dao.getCount(),is(0));
```

### 동일한 결과를 보장하는 테스트

db의 동일성은 당연하고, 테스트를 실행하는 순서를 바꿔도 동일한 결과가 보장되도록 만들어야 한다.

## 2.3.3 포괄적인 테스트

성의 없는 테스트는 안하느니만 못하다. 한가지 결과만 확인하는 것은 위험하다.

### getCount()테스트

```java
dao.add(user1);
assertThat(dao.getCount(),is(1));
```

JUnit은 특정한 테스트 메소드의 실행 순서를 보장해주지 않는다. ⇒ 모든 테스트는 실행 순서에 상관 없이 독립적으로 항상 동일한 결과를 낼 수 있도록 해야 한다.

### addAndGet() 테스트 보완

```java
User userget1 = dao.get(user1.getId());
assertThat(userget1.getName(),is(user1.getName()));
```

### get() 예외 조건에 대한 테스트

EmptyResultDataAccessException 이용

JUnit 예외 조건 테스트

```java
@Test(expected=EmptyResultDataAccessException.class) 
-> 테스트 중에 발생할 것으로 기대하는 예외 클래스를 지정해준다.

dao.get("unkown")// 아무 id나 get해서 exception 이 제대로 발생하는 지 확인한다.
```

### 테스트를 성공시키기 위한 코드의 수정

```java
if(user == null) throw new EmptyResultDataaccessException(1);
```

### 포괄적인 테스트

테스트는 성공할 상황이 아닌, 예외적인 상황에 대한 테스트를 해봐야 한다.