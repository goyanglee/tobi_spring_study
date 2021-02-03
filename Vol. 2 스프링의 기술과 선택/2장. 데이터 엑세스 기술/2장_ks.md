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
