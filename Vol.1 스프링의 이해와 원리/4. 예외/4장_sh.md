# 4장 예외

<br/>

## 4.1 사라진 SQLException

### 예외처리의 핵심원칙

모든 예외는 적절하게 복구되든지 작업을 중단시키고 운영자 또는 개발자에게 분명하게 통보되어야 한다. 

- catch 블록을 사용해 화면에 메시지를 출력하는 것은 에러를 처리한 것이 아니다. 
- 무책임한 throw 선언도 심각한 문제가 있다. 

<br/>

### 예외의 종류와 특징

#### 에러

- 시스템에 비정상적인 상황이 발생했을 경우에 사용된다. 
- 주로 JVM에서 발생한다. 
- 애플리케이션에서는 처리하지 않아도 된다. 

예) OutOfMemoryError, ThreadDeath

<br/>

#### 체크 예외

- 개발자들이 만든 애플리케이션 코드의 작업 중에 예외상황이 발생했을 경우에 사용된다. 
- Exception 클래스의 서브클래스이면서 RuntimeException을 상속하지 않은 클래스들이다.  
- 체크 예외가 발생할 수 있는 메소드를 사용할 경우 반드시 예외를 처리하는 코드를 함께 작성해야 한다. 
- 그렇지 않으면 컴파일 에러가 발생한다. 

예) IOException, SQLException

<br/>

#### 언체크 예외 (런타임 예외)

- 개발자들이 만든 애플리케이션 코드의 작업 중에 예외상황이 발생했을 경우에 사용된다. 
- RuntimeException을 상속한 클래스들이다. 
- 코드에서 미리 조건을 체크하도록 주의 깊게 만들면 피할 수 있다. 개발자가 부주의해서 발생할 수 있는 경우에 발생할 수 있는 경우에 발생하도록 만든 것이다. 
- 따라서, 예상하지 못했떤 예외상황에서 발생하는 것이 아니기 때문에, catch 문으로 잡거나 throws로 선언하지 않아도된다. 

예) NullPointerException, IllegalArgumentException 

<br/>

### 예외처리 방법

#### 예외 복구

- 예외상황을 파악하고 문제를 해결해서 정상 상태로 돌려놓는 것
- 예외로 인해 기본 작업 흐름이 불가능하면 다른 작업 흐름으로 자연스럽게 유도해주는 것이다. 

예) 정상적인 파일로 변경, 서버 접속 재시도 while문 작성 등

<br/>

#### 예외처리 회피

- 예외처리를 자신이 담당하지 않고 자신을 호출한 쪽으로 던져버리는 것이다. 
- throws 문으로 선언해서 예외가 발생하면 알아서 던져지게 하거나 catch 문으로 일단 예외를 잡은 후에 로그를 남기고 다시 예외를 던진다.

<br/>

#### 예외 전환

- 회피처럼 예외를 메소드 밖으로 던지는 것이다. 차이점은 발생한 예외를 그대로 넘기는 것이 아니라 적절한 예외로 전환해서 던진다. 

- 보통 전환하는 예외에 원래 발생한 예외를 담아서 **중첩 예외**로 만드는 것이 좋다. 

  ```
  #중첩예외1
  catch(SQLException e) {
  	...
  	throw DuplicateUserIdException(e);
  }
  
  #중첩예외2
  catch(SQLException e) {
  	...
  	throw DuplicateUserIdException().initCause(e);
  }
  ```

  

- 예외를 전환하는 목적

  1) 의미를 분명하게 해줄 수 있는 예외로 바꿔주기 위해

  2) 예외를 처리하기 쉽고 단순하게 만들기 위해 : 런타임 예외로 만들어서 굳이 필요하지 않은 catch/throws를 줄여준다.

<br/>

### "스프링 API 메소드에 정의되어 있는 대부분의 예외는 런타임 예외이다. 따라서 발생가능한 예외가 있다고 하더라도 처리하도록 강제하지 않는다. "

스프링의 JdbcTemplate은 예외처리 전략을 따르고 있다. JdbcTemplate 템플릿과 콜백 안에서 발생하는 모든 SQLException을 런타임 예외인 DataAccessException으로 포장해서 던져준다. 

JdbcTemplate의 update(), queryForInt(), query() 메소드는 모두 throws DataAccessException이라고 되어 있다. throws로 선언되어 있긴 하지만 DataAccessCeption이 런타임 예외이므로 update() 메소드에서 이를 잡거나 다시 던질 의무는 없다. 

<br/>

## 4.2 예외 전환

### DataAccessException 계층구조

스프링은 자바의 다양한 데이터 액세스 기술을 사용할 때 발생하는 예외들을 추상화해서 DataAccessExceptin 계층구조 안에 정리해뒀다. 추상화되어 있는 예외들의 타입은 다음과 같다. 

- 데이터 액세스 기술에 상관없이 공통적으로 발생하는 예외

  예) InvalidDataAccessResourceUsageException - 거의 대부분 프로그램을 잘못 작성해서 발생하는 오류



- 일부 데이터 액세스 기술에서만 발생하는 예외

  예) ObjectOptimisticLockinFailureException - JDO/JPA/Hibernate 마다 별도로 발생시키는 낙관적인 락킹예외를 이 예외로 통일시킴

  > 낙관적인 락킹? 
  >
  > 같은 정보를 두 명 이상의 사용자가 동시에 조회하고 순차적으로 업데이트할 때 뒤늦게 업데이트한 것이 먼저 업데이트한 것을 덮어쓰지 않도록 막아주는 데 쓸 수 있는 편리한 기능



- 템플릿 메소드나 DAO 메소드에서 직접 활용할 수 있는 예외

  예) IncorrectResultSizeDataAccessException - 한 개 로우만 돌려주는 쿼리에 사용하는 queryForObject()를 작성했을 때 여러 개의 로우를 가져온 경우 발생하는 예외
