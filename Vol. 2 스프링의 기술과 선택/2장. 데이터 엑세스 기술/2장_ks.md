# 11장 데이터 엑세스 기술 
## 공통 개념 
### DAO 패턴 
데이터 엑세스 계층을 DAO 패턴으로 분리한다. 
1. DAO는 인터페이스를 이용해 접근하고 DI 되도록 만들어야한다. DAO 인터페이스에는 어떠한 API나 정보도 노출하지 않는다. 
2. DAO 밖으로 던져지는 예외는 런타임 예외여야 한다. 왜냐하면 대부분 예외는 직접 다뤄야할 이유가 없기 때문이다.
3. 템플릿과 콜백 패턴을 이용해 try/catch/finally 와 같이 판에 박힌 반복되는 코드를 작성하지 않도록 한다.

### DataSource 
Connection 타입의 DB 연결 오브젝트. 보통 미리 정해진 개수만큼 pool에 준비해두고 필요할 때마다 pool에서 꺼내서 할당하고 돌려받는 식으로 풀링기법을 사용한다. 스프링에서는 DataSource를 하나의 독립된 빈으로 등록할 것을 권장한다.
1. SimpleDriverDataSource : 풀을 관리하지 않으므로 보통 테스트용으로 사용한다.
2. SingleConnectionDataSource : 풀링기법을 사용하지만 멀티쓰레드가 동작하는 경우 하나의 커넥션을 공유하게 되므로 위험하다.

#### 커넥션 풀 
1. 아파치 Commons DBCP : 아파치의 commons 프로젝트의 오픈소스 커넥션 라이브러리 
2. c3p0 JDBC/DataSource Resource pool : jdbc 3.0 스펙을 준수하는 라이브러리 
3. 상용 db 커넥션 풀 

#### JDNI/WAS DB 풀 
자바 서버 자체 DB 풀 서비스 

## JDBC
자바 데이터 엑세스 기술의 기본이 되는 로우레벨의 API 다. SQL의 호환성만 유지된다면 DB가 변경되어도 사용할 수 있다. 최신 ORM 기술도 내부적으로는 JDBC를 사용한다. 
### 스프링 JDB 기술과 동작원리 
#### 접근방법 
1. SimpleJdbcTemplate
2. SimpleJdbcInsert, SimpleJdbcCall 

#### JDBC가 해주는 작업 
1. Connection open, close : 스프링 트랜잭션과 맞물려서 열고 닫는다.
2. Statement 준비, 닫기, 실행 : Statement / preparedStatement를 생성하고 필요한 준비 작업을 해준다.
3. ResultSet 루프 : 루프를 만들어 반복
4. 예외처리와 변환 
5. 트랜잭션 처리 

### SimpleJdbcTemplate
```java
SimpleJdbcTemplate simpleJdbcTemplate;
public void setDataSource(DataSource dataSource) {
	this.simpleJdbcTemplate = new SimpleJdbcTemplate(datasource);
}
```

#### SQL 파라미터 
```sql
insert into member(id, name) values (?,?);

-> 실행하는 경우 

SimpleJdbcTemplate.update("insert into ~", 1, "spring");
```

```java
Map<String, Object> map = new HashMap<String, Object>();
map.put("id",1);
map.put("name","spring");
-> 실행하는 경우 
simpleJdbcTemplate.update("insert into ~ valued(:id,:name)", map);

or

MapSqlParameterSource map = new MapSqlParameterSource()
.addValue("id",1);
.addValue("name","spring);
-> 실행하는 경우 
simpleJdbcTemplate.update("insert ~ values(:id, :name)", map);
or

BeanPropertySqlParameter params = new BeanPropertySqlParameter(new Member(1,"spring));
-> 실행하는 경우 
simpleJdbcTemplate.update("insert ~ values(:id, :name)", params);
```

## iBatis SqlMaps

## JPA
Java Persistent API의 약자로, 영속성 관리와 ORM 을 위한 표준 기술. 오브젝트를 가지고 정보를 다루면 ORM 프레임워크가 이를 적절한 형태로 변환해주는 기술. 

### EntityManagerFactory 등록 
EntityManager는 
1. 애플리케이션이 관리하는 EntityManager
2. 컨테이너가 관리하는 EntityManager
가 있다.

#### EntityManagerFactory 등록 방법 1. LocalEntityManagerFactoryBean
PersistentProvider 자동 감지 기능을 이용해 프로바이더를 찾고 META-INF/persistence.xml 에 담긴 퍼시스턴스 유닛의 정보를 활용해 EntityManagerFactory를 생성한다. 
단점 : 스프링 빈으로 등록한 DataSource를 사용할 수 없다. 바이트코드 위빙 기법도 사용할 수 없다.
따라서 실전에선 사용하지 않는다.
#### EntityManagerFactory 등록 방법 2. JAVAEE 5 가 제공하는 EntityManagerFactory 
JNDI를 통해 EntityManager, EntityManagerFactory를 제공받을 수 있고, JTA를 이용해 트랜잭션 관리 기능을 활용할 수 있다. META-INF/persistenct.xml 으로 퍼시스턴스 유닛을 설정한 후 
```xml
<jee:jndi-lookup id="emf" jndi-name="persistenct/myPersistenctUnit"/>
```
로 빈 등록 기능을 이용한다.
장점 : JavaEE에서 사용되도록 개발된 JPA 모듈 그대로 사용할 수 있다.
#### EntityManagerFactory 등록 방법 3. LocalContainerEntityManagerFactoryBean 
JavaEE 가 아니어도 컨테이너에서 동작하는 JPA 와 그 이상 확장된 기능들을 사용할 수 있다.
빈으로 등록된 dataSource를 프로퍼티에 지정해주면 된다. 
* persistenctUnitName : 많은 퍼시스턴스 유닛들 중에 사용할 유닛을 지정 
* persistenceXmlLocation : 디폴트 위치(META-INF) 외의 경로를 사용할 경우 사용 
* jpaProperties, jpaPropertyMap
* jpaVendorAdapter : vendor 설정 
* loadtimeWeaver : 엔티티 클래스의 바이트코드를 직접 조작해서 확장된 기능 추가 = 바이트코드 향상 기법 
1. 바이트코드를 빌드 중에 변경 
2. 런타임 시에 바이트코드를 메모리에 로딩하면서 변경  = 로드타임 위버. 로드타임 위빙
#### 트랜잭션 매니저 
```xml
<bean id="transactionManager" class="org.springframework.orm.jpa.JpaTransactionManager">
<property name="entityManagerFactory" ref="emf"/>
</bean>
```
JpaTransactionManager를 사용하면 같은 DataSource를 공유하는 DAO 와 트랜잭션을 공유할 수도 있다.

### EntityManager 와 JpaTemplate
#### JpaTemplate
#### EntityManager 와 @PersistenceUnit
```java
EntityManager em = entityManagerFactory.createEntityManager();
em.getTransaction().begin();
//작업 
em.getTransaction().commit();
```
* @Autowired, @Resource 로 EntityManagerFactory DI
* @PersistenctUnit
```java
@PersistenctUnit EntityManagerFactory emf;
```


#### 컨테이너 관리 EntityManager와 @PersistenceContext 
@PersistenctContext를 사용하면 컨테이너가 사용하는 EntityManager를 직접 주입받을 수 있다. EntityManager는 빈으로 등록되지 않고 이를 생성하는 LocalContainerEntityManagerFactory이기 때문에 @Autowired같은 스프링의 di 방법으로는 주입받을 수 없고 @PersistenctContext를 사용한다.
```java
public class MemberDao {
	@PersistenctContext EntityManager em;
	public void addMember(Member member) {
		em.persist(member)
	}
}
```

하지만 EntityManager를 생성하고 재사용하는 개념은 아니고 트랜잭션에 연결되는 퍼시스턴스 컨텍스트를 갖는 일종의 프록시를 주입받는다. 그래서 그 범위 안에서 존재하다가 제거된다. 트랜잭션마다 다른 EntityManager를 사용하게 된다.
```java
@PersistenceContext(type=PersistenceContextType.TRANSACTION) EntityManager em;
```

#### @PersistenctContext와 확장된 퍼시스턴스 컨텍스트 
PersistenctContextType.EXTENDED로 지정하면 확장된 스코프를 가진 EntityManager가 만들어진다. 이는 상태유지 세션빈에 바인딩되어 사용자별로 독립적으로 만들어지고 장기간 보존된다. 

#### JPA 예외 변환 
자동으로 예외변환이 일어나지 않지만 런타임 예외를 발생시키기 때문에 불필요한 try/catch 블록은 필요없다.

#### JPA 예외 변환 AOP
JPA 예외를 스프링의 DataAccessException 예외로 전환시킬 수는 있다.
##### @Repository 
DAO 에 @Repository를 선언한다.
##### PersistenceExceptionTranslationPostProcessor
@Repository 가 붙은 빈을 찾아서 AOP 어드바이스를 적용시켜주는 후처리기가 필요한데 PersistenceExceptionTranslationPostProcessor 를 빈으로 등록해주면 된다.

## 하이버네이트 
ORM 프레임워크 중 가장 성공한 오픈소스. 
### SessionFactory 등록 
JPA 의 EntityManagerFactory 같은 SessionFactory가 있다. 
#### LocalSessionFactoryBean 
스프링이 제공하는 트랜잭션 매니저와 연동될 수 있도록 설정된 SessionFactory 생성 

* mappingLocations : 매핑 파일 위치 
* hibernateProperties : hibernate.cfg.xml 안의 설정값 

#### AnnotationSessionFactoryBean 

* annotatedClasses : 매핑 어노테이션이 부여된 클래스 목록 
* packagesToScan : 자동 스캔 패키지 지정 

#### 트랜잭션 매니저 
##### HibernateTransactionManager 
단일 DB에 JTA를 이용할 필요가 없다면 이 빈을 사용한다.

##### JtaTransactionManager
여러 DB에 대한 작업을 하나의 트랜잭션으로 묶으려면 JTA를 통해 글로벌 트랜잭션 기능을 사용해야 한다.

### Session과 HibernateTemplate 
Session은 SessionFactory로 부터 만들어지며 트랜잭션과 동일한 스코프를 갖는다.
#### HibernateTemplate 
스프링의 템플릿/콜백 패턴이 적용된다. 그다지 권장되지는 않는다.
#### SessionFactory.getCurrentSession() 
현재 트랜잭션이 연결되어있는 하이버네이트 Session을 돌려준다. 

### Session 과 HibernateTemplate 
Session은 하이버네이트의 핵심 API 이며 트랜잭션과 동일한 스코프를 가진다. 
#### Session을 사용하는 방법 1. HibernateTemplate 
```java
private HibernameTemplate hibernateTemplate;

@AUtowired
public void setSessionFactor(SessionFactory sf) {
	hibernateTemplate = new HibernateTemplate(sf);
}
```
그리 권장되진 않는다.
#### Session을 사용하는 방법 2. SessionFactory.getCurrentSession()
현재 트랜잭션에 연결되어 있는 하이버네이트 Session을 돌려준다.  반드시 트랜잭션이 시작된 후에만 사용할 수 있고, 없다면 현재 스레드에 바인딩된 Session이 없다는 예외가 발생한다. 

## 트랜잭션 
### 트랜잭션 추상화와 동기화 
#### PlatformTransactionManager 
```java
public interface PlatformTransactionManager {
	TransactionStatus getTransaction(TransactionDefinition definition) throws TransactionException;
	void commit(TransactionStatus status) throws TransactionException;
	void rollback(TransactionStatuas status) throws TransactionExcepion;
}
```
트랜잭션 경계를 지정. 어디서 시작하고 종료하는지, 정상종료인지 비정상종료인지를 결정한다. 
TransactionDefinition은 트랜잭션의 네 가지 속성을 나타내는 인터페이스이다. 
#### 트랜잭션 매니저의 종류 
##### DataSourceTransactionManager 
Connection의 트랜잭션 API 를 이용해 트랜잭션을 관리해준다. DataSource를 DI 해주어야 한다. ThreadLocal 등을 이용해 트랜잭션을 저장해두고 돌려주는 기능을 가진 DataSource를 사용하면 안된다. 
##### JpaTransactionManager 
JPA를 이용하는 DAO의 경우 사용한다. LocalContainerEntityManagerFactoryBean을 프로퍼티로 등록해줘야한다. 
##### HibernateTransactionManager
하이버네이트 DAO에 사용한다.
##### JmsTransactionManager, CciTransactionManager 
JMS ,CCI를 위핳 ㄴ트랜잭션 매니저 
##### JtaTransactionManager
하나 이상의 DB 또는 트랜잭션 리소스가 참여하는 글로벌 트랜잭션을 적용할 때 사용한다.
DB가 하나라면 트랜잭션 매니저도 하나만 등록되어야 하고, DB가 여러개라도 JTA를 이용해 글로벌 트랜잭션을 적용할 거라면 JtaTransactionManager 하나만 등록한다. 단, 두개 이상의 DB를 독립적으로 이용하는 경우라면 두 개 이상의 트랜잭션 매니저를 등록할 수는 있다. 
### 트랜잭션 경계설정 전략 
#### 코드에 의한 트랜잭션 경계 설정 
```java 
public void addMembers(List<Member> members) {
	this.transactionTemplate.execute(new TransactionCallback {
		public Object doInTransaction(TrnasactionStatus statue) {
			memberdao.addMember~
			return null;
		}
		
	};
}
```

대개는 선언적 트랜잭션 방식을 사용하지만 의도적으로 테스트코드에서 만들고 종료해야하는 상황이 생긴다면 유용하다.

#### 선언적 트랜잭션 경게 설정 
##### aop와 tx 네임스페이스 
트랜잭션 어드바이스가 필요하고 어떤 대상에게 부여할지를 선정해야한다.
```xml
//어드바이스 
<tx:advice id="" transaction-manager="">
	<tx:attributes>
		<tx:method name="*"/>
	</tx:attributes>
</tx:advice>
```

##### @Transactional 
<tx:annotation-driven> 태그 한줄만 있으면 사용가능하다.
```java
@Transactional
public interface MemberDao {
	~
}
```
인터페이스 내에 선언된 모든 메소드에 적용된다. 메소드 레벨에도 적용할 수 있는데 이는 인터페이스 선언보다 우선한다.
우선순위 : 메소드 > 클래스  > 인터페이스의 메소드 > 인터페이스 

#### 프록시모드 : 인터페이스와 클래스 
스프링에서는 jdk 다이나믹 프록시외에도 CGLib 라이브러리가 제공해주는 클래스 레벨의 프록시도 사용 가능하다.
##### aop/tx 스키마 태그의 클래스 프록시 설정 
```xml
<aop:config proxy-target-class="true">
	<aop:pointcut id="txPointcut" expression=~/>
	<aop:advisor advice-ref="txAdvice" pointcut-ref="txPointcut"/>
</aop:config>
```
##### @Transactional 클래스 프록시 설정 
```xml
<tx:annotation-driven proxy-target-class="true"/>
```
1. 클래스에 부여해야한다. 
2. final 클래스에는 적용할 수 없다. 클래스 프록시를 적용하면 생성자가 두 번 호출된다. 상속을 통해 만들기 때문이다.
3. 클래스에 적용하는 경우 모든 public 메소드에 적용된다. 수정자같은 메소드에도 적용이 되기 때문에 시간과 리소스에 낭비가 발생한다. 
#### AOP 방식: 프록시와 AspectJ 
스프링 aop 대신 aop 전용 프레임워크인 AspectJ를 사용할 수도 있다. 프록시를 앞에 두는 방식 대신에 오브젝트 자체를 조작해서 부가기능을 직접 넣기 때문에 고급 기능을 이용할 수도 있다. 
프록시가 적용되면 타깃 오브젝트 메소드 호출 전에 트랜잭션을 시작하고 호출 후에 커밋하거나 롤백 해준다. 타깃 오브젝트의 메소드가 자기 자신의 다른 메소드를 호출할 때에는 프록시가 동작하지 않는다. 이미 거쳐서 진행됐기 때문이다. 
```java 
@Transactional
public class MemberService {
	@Transactional(propagation = Propogation.REQUIRES_NEW) 
	public void add(Member member) {}
	public void complex() {
		this.add(new Member~);
	}
}
```
위의 경우에서 complex()를 호출했을 때 add()는 어떻게 될까? 이미 프록시를 지나서 호출됐기 때문에 add에 붙은 트랜잭션 속성은 반영되지 못하고 complex() 에서 시작된 트랜잭션에 그냥 참여하게 된다. 만약 이 상황에서 add()를 호출할 때에도 프록시를 거치게 하고 싶다면, 

##### AopContext.currentProxy()
##### AspectJ AOP

### 트랜잭션 속성 
#### 트랜잭션 전파 : propagation
트랜잭션을 시작하거나 기존 트랜잭션에 참여하는 방법을 결정하는 속성. 여러 트랜잭션 적용 범위를 묶어서 커다란 트랜잭션 경계를 만들 수 있다. 
1. REQUIRED : 디폴트 속성. 이미 시작된 트랜잭션이 있으면 참여하고 없으면 새로 시작한다. 
2. SUPPORTS : 이미 시작된 트랜잭션이 있으면 참여하고 없으면 없는대로 진행한다.
3. MANDATORY : 이미 시작된 트랜잭션이 있으면 참여한다. 없으면 예외 발생
4. REQUIRED_NEW : 항상 새로운 트랜잭션을 시작한다.
5. NOT_SUPPORTED : 트랜잭션을 사용하지 않는다. 이미 시작된 트랜잭션이 있으면 보류시킨다.
6. NEVER : 트랜잭션을 사용하지 않도록 강제한다.
7. NESTED : 이미 진행 중인 트랜잭션이 있으면 중첩 트랜잭션을 시작한다.  먼저 시작된 부모 트랜잭션에는 영향을 받지만 부모에게 영향을 주지 않는다. 

#### 트랜잭션 격리 수준 : isolation
여러 트랜잭션이 동시에 진행될 때 작업 결과를 다른 트랜잭션에 어떻게 노출할 것인지 결정하는 기준 
1. DEFAULT : 데이터 엑세스 기술 또는 DB 드라이버의 디폴트 설정을 따른다. 대부분의 DB는 READ_COMMITED를 갖는다.
2. READ_UNCOMMITED : 트랜잭션이 커밋되기 전의 변화가 노출된다. 
3. REAS_COMMITED : 가장 많이 사용되는 격리 수준. 읽은 데이터를 다른 트랜잭션이 수정할 수 있다. 
4. REPEATABLE_READ : 하나의 트랜잭션이 읽은 로우를 다른 트랜잭션이 수정하는 것을 막아준다. 
5. SERIALIZABLE : 트랜잭션을 순차적으로 진행시켜 가장 안전하지만 성능이 가장 떨어진다. 

#### 트랜잭션 제한시간 : timeout 
초 단위로 지정해서 트랜잭션에 제한시간을 지정한다. 
#### 읽기전용 트랜잭션 : read-only, readOnly
트랜잭션을 읽기 전용으로 설정한다. 일부 트랜잭션 매니저는 읽기전용 속성을 무시하고 쓰기 작업을 허용할수도 있어서 주의해야한다.
#### 트랜잭션 롤백 예외 : rollback-for
런타임 예외가 발생하면 롤백한다. 체크예외가 발생하면 커밋된다.
#### 트랜잭션 커밋 예외 : no-rollback-for
런타임 예외를 트랜잭션 커밋 대상으로 지정해준다.

### 데이터 엑세스 기술 트랜잭션의 통합 
하나의 트랜잭션 매니저가 여러 개의 데이터 액세스 기술의 트랜잭션 기능을 지원해주도록 만든다. 
#### 트랜잭션 매니저별 조합 가능 기술 
##### DataSourceTransactionManager 
JDBC, iBatis 두 가지 기술을 함께 사용할 수 있다. 항상 동일한 DataSource를 사용해야 한다. 
##### JpaTransactionManager 
JPA, jdbc, ibatis 를 함께 사용할 수 있다. 
##### HibernateTransactionManager 
하이버네이트, jdbc, ibatis 와 트랜잭션을 공유할 수 있다. 
##### JtaTransactionManager 
하나이상의 디비를 통합해서 하나의 트랜잭션으로 관리하려고 할때에는 JTA가 반드시 필요하다. 

#### ORM 과 비ORM 을 함께 사용할 때 주의사항 
orm은 flush()를 통해 캐시에 저장되어있던 실행 쿼리문을 실행해서 db에 반영한다. 이렇듯 각 프레임워크의 동작 원리를 알고 수행해야 한다.
### JTA를 이용한 글로벌/분산 트랜잭션 
```xml
<jee:jndi-lookup id="dataSource1" jndi-name=""/>
<bean memberDao~/>

<jee:jndi-lookup id="dataSource2" jndi-name=""/>
<bean userDao~/>
```

#### 독립형 JTA 트랜잭션 매니저 
서버 없이 앱 안에 jta 기능을 내장하는 방식을 이용할 수도 있다. ObjectWeb의 jta 엔진인 jotm, Atomikos 의 TransactionalEssentials가 대표적이다. 
#### WAS 트랜잭션 매니저의 고급 기능 사용하기 
1. WebSphereUowTransactionManager
2. WebLogicJtaTransactionManager
3. OC3JJtaTransactionManager
