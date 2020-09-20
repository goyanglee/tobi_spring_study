# 4장 예외
## 예외의 종류
### Error
시스템에 뭔가 비정상적인 상황이 발생한 경우. 주로 JVM에서 발생시킨다. 대응방법이 없고, 시스템 레벨의 개발이 아니라면 무시해도 된다.

### Exception
1. 체크 예외 : 예외 처리를 강제한다. Exception 의 서브클래스이면서 RuntimeException 을 상속하지 않는 Exception
2. 언체크 예외 : 명시적인 예외 처리를 강제하지 않는다. 피할 수 있는 에러이며 주로 개발자의 부주의로 발생할 수 있는 예외들이다. Exception 의 서브클래스이면서 RuntimeException을 상속하는 Exception
ex) NullPointerException, IllegalArgumentException,..

RuntimeException도 Exception을 상속하지만 RuntimeException부터 상속하는 모든 Exception을 언체크예외라고 한다.

## 예외의 처리
### 예외 복구
사용자에게 알려서 다른 작업의 흐름을 유도하는 방법. 또는 일정시간 재시도를 하기도 한다. 복구 가능성이 있는 경우 사용한다.

### 예외 회피 
호출한 메소드로 책임을 throw한다. 무분별하게 던질 부작용이 있다.

### 예외 전환 
적절한 예외를 전환해서 던진다. 의미를 분명하게 하는 구체적인 Exception 을 던져서 받는 쪽에서 예외를 짐작할 수 있기 한다.
사용 목적 : 
1. 내부에서 발생한 예외를 그대로 던졌을 때 예외 상황에 대한 설명이 정확하게 되지 않을 경우. 예를 들어서 같은 아이디의 사용자를 등록하고자 할 때 데이터베이스 Exception 이 발생하지만 이것 만으로는 데이터베이스의 문제인지, 중복되는 사용자를 등록하려고 해서 발생한 것인지 분명하지 않다. 그래서 다른 예외로 전환시켜주는 것이다.
2. 강제로 체크예외를 언체크예외로 바꾸는 경우. 복구할 수 없는 수준의 체크 예외를 언체크예외로 변환해서 다른 메소드에서 throws 선언 하는 것을 줄인다.

## 예외처리 전략
### 런타임 예외의 보편화
예외 처리를 강제하면서 무분별한 예외 처리를 해야만 했던 문제점이 존재해왔다.  서버는 수많은 요청들을 수행하다가 예외가 발생하면 해당 요청만 중단시키면 되고, 원상복구 시키기 위해 사용자와 커뮤니케이션할 수도 없다보니 다른 방법이 없다. 그래서 강박적으로 처리해야했던 예외처리들을 하지 않도록 런타임 예외로 변환한다.

### 체크예외를 명시적으로 던지기
의미있는 체크예외도 더 앞단에서 처리할 수도 있다면 런타임 예외로 만든다. 대신 메소드는 명시적으로 어떤 예외를 던진다고 선언해야한다.
```java
public class DuplicateUserIdException extends RuntimeException {
	public DuplicateUserIdException(throwable cause) {super(cause);}
}
```
런타임 익셉션을 생성한 후,
```java
public void add() throws DuplicationUserIdException {
	try{~~~}{}
	catch(SQLException e){if(e.errorcode==error){throw new DuplicationUserIdException(e);}}
}
```
로 메소드에 명시적으로 던질 예외를 선언한다.
단, 런타임 예외가 되었기 때문에 주의를 기울일 필요는 있다.

### 애플리케이션 예외
애플리케이션 자체 로직에 의도적으로 발생시켜서 반드시 조치를 취해야 하는 예외도 있다. 이를 **애플리케이션 예외**라고 한다.
설계하는 방법 :
1. 각 시나리오에 맞는 다른 종류의 리턴값을 반환한다. 호출한 쪽은 리턴값을 보고 판단한다. 단, 명확하게 코드화하고 관리되어야 한다.
2. 정상적인 코드 외에 예외 시나리오에서 예외를 던진다. 

### 직접 DB를 자유롭게 바꿔 사용할 수 있는 프로그램 작성하기
#### 문제점 
1. JDBC 코드에서 사용하는 SQL은 어느정도 표준화되어 있지만 대부분의 DB는 비표준 문법과 기능도 제공한다. 이 때문에 DB 에 종속될 수 밖에 없다.
해결책 : 표준 SQL 만 사용하거나 DB 별로 DAO 를 만든다.
2. SQLException, 에러의 종류와 원인은 DB마다 다르다. SQLException 은 상태 코드를 제공해서 DB별로 달라지는 에러코드를 대신할 수 있도록 하지만 드라이버에서 정확하게 만들어주지는 않는다.

#### 해결책
1. DB 업체마다 제공하는 DB 전용 에러 코드를 참고한다. 예를 들어 아이디 중복 값으로 에러가 발생하면 MySQL 은 1062, 오라클은 1을 반환한다. 
2. 스프링은 SQLException 을 대체할 수 있는 세분화된 Exception 을 제공한다. 예를 들어서 BadSqlGrammarException, DataAccessResourceFailureException 등을 제공한다. 
3. 스프링은 DB 마다 다른 에러코드들과 매핑되는 예외클래스를 정의해두고 이용한다.

#### DAO를 왜 분리할까?
데이터 엑세스 로직을 담은 코드를 성격이 다른 코드에서 분리해놓기 위해서이다. 분리된 DAO 는 전략 패턴을 적용해 구현 방법을 변경해서 사용할 수 있게 한다. 
##### 문제점 
```java 
public void add(User user) throws PersistentException //JPA
public void add(User user) throws HibernateException // Hibernate
public void add(User user) throws JdoException; //JDO
```
위와 같이 DB 별로 예외가 발생한걸 던진다면 DAO를 분리한 이점을 얻을 수 없다. 그러나 다행이도 JDBC 보다 늦게 등장한 이 기술들은 체크 예외 대신 런타임 예외를 사용해서 throws를 하지 않아도 된다.
```java
public void add(User user);
```
##### 개선점
클라이언트 입장에서는 DAO 사용 기술에 따라 예외 처리 방법이 달라져야 해서 DAO 에 의존적이 되므로 체크예외를 런타임 예외로 변환시키는 것 만으로는 불충분하다.

##### 해결책 
그래서 이러한 예외들을 모아 DataAccessException 계층 구조 안에 정리했다. 이는 자바의 주요 데이터 엑세스 기술에서 발생할 수 있는 대부분의 예외를 추상화하고 있다.

> **낙관적인 락킹 Optimistic locking**
> 같은 정보를 두 명 이상의 사용자가 동시에 조회하고 업데이트 할 때 뒤늦게 업데이트 한 것이 먼저 업데이트한 것을 덮어쓰지 않도록 막아준다.

낙관적인 락킹이 발생했을 때 DAO 마다 다른 종류를 발생시킨다. 그러나 DataAccessException 의 서브클래스로 통일시킬 수 있다.

#### 기술에 독립적인 UserDao를 만들자
##### 인터페이스 적용
UserDao의 인터페이스를 만들고 이를 확장한다.
##### 테스트하기
```java
@Test(expected=DataAccessException.class) //DataAccessException가 발생하는 것을 검증한다.
public void duplicateKey() {
	dao.deleteAll();
	dao.add(user1);
	dao.add(user1);
}
```

#### 주의사항
스프링이 최종적으로 DataAccessException 으로 변환해주긴 하지만 세분화되어 있지 않은 예외들도 있다. Dao마다 같은 상황에서 각기 다른 Exception 을 발생시킨다. 그러니 구현 시 미리 확인할 필요가 있다.



