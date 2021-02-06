# 2장 데이터 액세스 기술

<br/>

### 스프링은 주요 자바 데이터 액세스 기술을 모두 지원한다. 

> 자바는 JDBC API 외에도 JDBC 사용을 추상화해주는 iBatis SQLMapper나 오픈소스 ORM 대표 하이버네이트, JPA, JDO 등 다양한 데이터 액세스 기술을 제공한다. 

<br/>

#### 데이터 액세스 계층은 DAO 패턴으로 분리하기: DTO 또는 도메인 오브젝트만을 사용하는 인터페이스를 통해 데이터 액세스 기술을 외부에 노출하지 않는다.

- 인터페이스를 이용한 접근필요
  - DAO는 인터페이스를 이용해서 접근하고 DI되도록 생성해야 한다. 
    - DAO 클래스의 메소드를 생각없이 public으로 선언하지 않도록 주의
    - persist() 처럼 특정 데이터 액세스 기술에서 사용하는 메소드 이름을 사용하지 말자. 
- 예외처리 주의
  - DAO 밖으로 던지는 예외는 런타임예외여야 한다. (데이터 액세스 중에 발생하는 예외는 대부분 복구가 불가능하기 때문)
  - DAO 메소드 선언부에 throws SQLException 처럼 내부 기술을 드러내는 예외를 직접노출 X
  - throws Exception 같은 무책임한 선언도 X

<br/>

#### 스프링은 데이터 액세스 기술을 위한 템플릿을 제공한다: 템플릿/콜백 패턴 - 미리 만들어진 작업흐름을 담는다. 

- 중복코드 제거
- 트랜잭션 동기화기능 제공

<br/>

#### 다중사용자를 갖는 엔터프라이즈 시스템에서는 반드시 DB 연결 풀 기능을 지원하는 DataSource를 사용하기

> #### 스프링에서는 DataSource를 하나의 독립된 빈으로 등록하도록 강력하게 권장한다
>
> 스프링 데이터 기술의 다양한 서비스에서 DataSource를 필요로 하므로 공유가능한 스프링 빈으로 등록해줘야 한다. 스프링은 주요 데이터 액세스 기술이 자체적인 DataSource 생성방식 대신 스프링 빈으로 등록된 DataSource를 사용하는 데 필요한 방법을 제공해준다. 

<b>테스트를 위한 것</b> : 

- SimpleDriverDataSource : 매번 DB 커넥션 새로 만들고 풀 관리X
- SingleConnectionDataSource : 하나의 물리적인 DB 커넥션 만들어두고 사용. 순차적 통합테스트에 적합. 동시에 N개 스레드가 동작하면 사용X

<b>오픈소스 또는 상용 디비 커넥션 풀</b> :

- 아파치 Commons DBCP : 가장 보편적인 DB 커넥션 풀 라이브러리
- c3p0 JDBC/DataSource Resource Pool
- 상용 DB 커넥션 풀

<b>JDNI/WAS DB 풀</b> : 대부분의 자바 서버는 자체적으로 DB 풀 서비스 제공해준다. 

<br/>

### 종류들

- iBatis : 자바오브젝트와 SQL문 사이의 자동매핑 기능을 지원하는 ORM 프레임워크
- JPA : JavaEE와 JavaSE를 위한 영속성 관리와 O/R 매핑을 위한 표준 기술
- 하이버네이트 : POJO 프로그래밍 바탕의 오픈소스 ORM 프레임워크

<br/>

## 트랜잭션: 선언적 트랜잭션

선언적 트랜잭션 경계설정을 사용해서 트랜잭션이 시작되고 종료되는 지점을 별도설정을 통해 결정할 수 있다. 선언적 트랜잭션이 제공하는 트랜잭션 전파기능으로, 작은 단위로 분리되어 있는 데이터 액세스로직과 비즈니스 로직 컴포넌트와 메소드를 조합해서 하나의 트랜잭션에서 동작하게 만들 수 있다. 

- 코드중복 제거 가능
- 작은 단위의 컴포넌트로 쪼개서 개발한 후 조합 가능

<br/>

### 트랜잭션 추상화

핵심 인터페이스는 "PlatformTransactionManager"

모든 스프링의 트랜잭션 기능과 코드는 이 인터페이스를 통해서 로우레벨의 트랜잭션 서비스를 이용한다. 트랜잭션 경계를 지정하는 데에 사용한다. 

```
public interface PlatformTransactionManager {
	TransactionStatus getTransaction(TransactionDefinition definition) throws TransactionException; 
	
	void commit(TransactionStatus status) throws TransactionException;
	
	void rollback(TransactionStatus status) throws TransactionException; 
}
```

<br/>

#### 종류

- DataSourceTransactionManager
  - Connection의 트랜잭션 API를 이용해서 트랜잭션을 관리해주는 트랜잭션 매니저. 이 트랜잭션 매니저를 사용하려면 트랜잭션을 적용할 DataSource가 스프링의 빈으로 등록되어야 한다. 
- JpaTransactionManager 
  - JPA를 이용하는 DAO에 사용하는 매니저
- HibernateTransactionManager
  - 하이버네이트 DAO에 사용하는 매니저
- JmsTransactionManager, CciTransactionManager
  - 트랜잭션이 지원되는 JMS와 CCI를 위한 트랜잭션 매니저
- JtaTransactionManager
  - 하나 이상의 DB 또는 트랜잭션 리소스가 참여하는 글로벌 트랜잭션 가능

<br/>

#### 경계설정 전략

1. 코드에 의한 설정: 트랜잭션을 다루는 코드를 직접 만든다. 
2. AOP를 이용한 선언적 설정: AOP를 이용해 기존코드에 트랜잭션 경계설정 기능을 부여한다. 특정 메소드 실행 전 후에 트랜잭션이 시작되고 종료되거나 기존 트랜잭션에 참여하도록 만들 수 있다. 
   - 전용 태그를 사용하거나
   - @Transational 애노테이션 사용

> #### 스프링의 AOP는 기본적으로 프록시 방식이다: 인터페이스를 이용하는 JDK 다이내믹 프록시 / 클래스에 바로 프록시를 만드는 CGLib 모두 프록시 오브젝트를 타깃 오브젝트 앞에두고 호출과정을 가로채서 트랜잭션과 같은 부가적인 작업을 진행해준다. 
>
> - 스프링의 프록시 AOP 대신 전용 프레임워크인 AspectJ의 AOP를 사용할 수 있다. 	
>   - AspectJ의 AOP는 프록시는 타깃 오브젝트 앞에 두지 않고 타깃 오브젝트 자체를 조작해서 부가기능을 직접 넣는 방식을 사용한다. 
>   - 메소드 실행지점만 조인포인트로 사용할 수 있는 스프링 AOP에서는 불가능한 다양한 조인포인트와 고급기능을 이용할 수 있다. 
>   - 대신 별도의 빌드과정이나 바이트코드 조작을 위한 로드타입 위버 설정과 같은 부가적인 작업이 필요하다. 
>   - 트랜잭션 AOP를 적용하기 위해 굳이 번거롭게 AspectJ를 사용할 필요는 X

<br/>

#### 트랜잭션 속성

[트랜잭션 전파: propagation] : 트랜잭션을 시작하거나 기존 트랜잭션에 참여하는 방법을 결정하는 속성

[트랜잭션 격리수준: isolation] : 동시에 여러 트랜잭션이 진행될 때 트랜잭션의 작업 결과를 여타 트랜잭션에 어떻게 노출할 것인지를 결정하는 기준

[트랜잭션 제한시간: timeout] : 초 단위 트랜잭션 제한시간

[읽기전용 트랜잭션: read-only, readOnly]

[트랜잭션 롤백 예외: rollback-for, rollbackFor, rollbackForClassName]

[트랜잭션 커밋 예외: no-rollback-for, noRollbackFor, noRollbackForClassName]



