> 스프링의 3대 핵심 기술. 1. IoC/DI 2. 서비스 추상화 3. AOP

## SQL 과 DAO 의 분리
DAO 인터페이스, JDBC 처리 기술은 바뀌지 않을 테지만 테이블, 필드 명, sql이 바뀔수 있다. 따라서 sql과 dao를 분리해본다.

### xml 설정을 이용한 분리 
xml로 sql을 뺀다. 

#### 개별 sql 프로퍼티 방식 
```java
public class UserDaoJdbc implements UserDao {
	private String sqlAdd;
	public void setSqlAdd(String sqlAdd) {
		this.sqlAdd=sqlAdd;
	}
	public void add(User user) {
		this.jdbcTemplate.update(this.sqlAdd, user.getId(),..);
	}
}

//빈 설정 시
<property name=“sqlAdd” value=“insert into users~”/>
```

#### SQL 맵 프로퍼티 방식
sql을 하나의 컬렉션으로 사용할 수 있다.
```java
public class UserDaoJdbc implements UserDao{
	private Map<String, String> sqlMap;
	public void setSqlMap(Map<string, String> sqlMap) {
		this.sqlMap=sqlMap;
	}
	public void add(User user) {
		this.jdbcTemplate.update(this.sqlMap.get(“add”),~)
	}
}

//빈 설정 시
<entry key=“add” value=“inser~”/>
```

### SQL 제공 서비스 
```java
public void add(User user) {
	this.jdbcTemplate.update(this.sqlService.getSql(“userAdd”)~;
}
```
SqlService는 위의 맵을 사용하는 방식을 이용해서 설정파일에 만들어진 sql들을 가져와 프로퍼티로 등록해둔다.

## 인터페이스 분리와 자기 참조 빈 
### xml 파일 매핑 
#### JAXB (Java Architecture for XML Binding)
Java6에 있는 라이브러리로 DOM과 같은 전통적인 xml api와 비교했을 때 XML 정보를 거의 동일한 구조의 오브젝트로 직접 매핑해준다는 장점이 있다.  XML 문서의 구조를 정의한 스키마를 이용해 매핑할 오브젝트의 클래스까지 자동으로 만들어주는 컴파일러도 제공한다. 

**동작방식**
![그림7-1]() 

```xml
//SQL 맵 xml 문서
<sqlmap>
	<sql key=“”>insert into ~ ... </sql>
</sqlmap>
```

```xml
<schema xmlns~>
<element name=“sqlmap”>
	<complexType>
		<seqeunce>
			<element name=“sql” maxOccurs=“unbounded” type=“tns:sqlType” />
		</seqeunce>
	</complexType>
</element>
```
스키마 파일을 sqlmap.xsd 라는 이름으로 프로젝트 루트에 저장하고 JAXB 컴파일러로 컴파일하고 바인딩할 클래스 위치를 지정해준다.

```
//명령어 실행
xjc -p springbook.user.sqlservice.jaxb sqlmap.xsd -d src
```

```java
@XmlAccessorType(XmlAccessType.FIELD)
@XmlType(name=“”, propOrder={“sql”})
@XmlRootElement(name=“sqlmap”)
public class Sqlmap {
	@XmlElement(required=true)
	protected List<SqlType> sql;
	~~
}
```

#### 언마샬링 
XML 문서를 읽어서 자바의 오브젝트로 변환하는 것을 언마샬링(Unmarshalling) 이라고 하고, 반대로 바인딩 오브젝트를 XML 문서로 변환하는 것을 마샬링(Marshalling)이라고 한다.

### XML 파일을 이용하는 SQL 서비스 
#### XML SQL 서비스
요청이 들어왔을 때 sql을 읽어들이는 것을 우선 SQL 들을 가져와 Map 형태로 저장해보자.

```java
public class XmlSqlService implements SqlService {
	private Map<string, String> sqlMap = new HashMap<String, String>();
public XmlSqlService() {
	String contextPath = Sqlmap.class.getPackage().getName();
	try{
		JAXBContext context = JAXBContext.newInstance~;
		Unmashaller unmashaller = context.createUnmarsh~
		InputStream is =UserDao.class.getResourceAsStream(“~.xml”);
		Sqlmap sqlMap = (Sqlmap)unma~.unmarshal(is);
		
		for(sql:sqlmap.getSql()) {
			sqlMap.put(sql.getKey(), sql.getValue());
		}
	}catch~
}
}
```

### 빈 초기화 
생성자에서 예외가 발생할 수 있는 초기화 작업을 다루는 것은 좋지 않다. 코드의 로직과 파일의 위치 등 바뀔 수 있는 정보들은 별도의 메소드로 추출하고 di로 설정할 수 있도록 한다.

```java
InputStream is = UserDao.class.getResourceAsStream(this.sqlmapFile);
```

**초기화**
```java
XmlSqlService sqlProvider = new XmlSqlService();
sqlProvider.setSqlmapFile(“~.xml”);
sqlProvider.loadSql();
```

@PostContruct : 이 어노테이션을 메소드 위에 붙여주면 빈 오브젝트를 생성하고 주입작업이 끝난 후 자동으로 실행해준다.
**스프링 컨테이너 초기 작업 순서**
1. xml 빈 설정을 읽는다.
2. 빈의 오브젝트를 생성한다.
3. 프로퍼티에 의존 오브젝트 또는 값을 주입한다.
4. 빈이나 태그로 등록된 후처리기를 동작시킨다. 

### 인터페이스 분리
책임에 따라서
#### SqlRegistry 인터페이스 
#### SqlReader 인터페이스

### 자기참조 빈
위에 분리한 인터페이스와 더불어 본인도 인터페이스로 만든 다음 이 세 인터페이스를 구현한 인터페이스를 사용한다.

### 디폴트 의존관계
setter, 생성자로 값을 세팅해주기 전 디폴트로 수행할 값을 세팅해준다.
```java
public class JaxbXmlSqlReader implements SqlReader {
	private static final String DEFAULT_FILE=“sqlmap.xml”;
	public setter(~);
}
```

## 서비스 추상화 
OXM(Object-XML Mapping) 라이브러리는 여러 개가 있다. 
1. jaxb
2. castor xml
3. jibx
4. xmlbeans
5. xstream
이들 별로 확장하는 방법은
```java
public class OxmlSqlService implements SqlService {
	private final OxmlSqlReader reader = new OxmSqlReader(); //디폴트

public setter();
}
```

### 리소스 추상화
리소스는 빈이 아니라 값 그 자체로 사용되는 클래스다. 거의 모든 api는 외부의 리소스 정보가 필요할 때에는 항상 이 리소스 추상화를 이용한다.

## 인터페이스 상속을 통해 기능 확장 
서버가 재시작하지 않고 긴급하게 sql을 변경해 반영해보자. 우리가 이전에 했던 작업들처럼 di 확장을 사용해보자

### 인터페이스를 확장하는 이유
1. 다형성을 얻기 위해.
2. 인터페이스 분리 원칙을 통해 의존 관계를 명확하게 하기 위해.

## DI를 이용한 다양한 구현 방법 
지금까지의 SqlRegistry는 쓰레드에서 공유하기 때문에 동시에 CUD를 할 경우 문제가 생길 수 있다. 
### ConcurrentHashMap 
데이터 조작 시 전체 데이터에 대해 락을 걸지 않는다(?). 멀티쓰레드 환경에서 보다 안전해진다. 
### 내장형 데이터베이스 사용 
1. Derby
2. HSQL
3. H2 
JDBC 방식이라고 해서 기존 DataSource, Dao를 사용하는 것 보다는 내장형 디비용 JDBC드라이버를 사용하면 별도의 초기화 과정을 하지 않아도 되어서 좋다. 
EnbeddedDatabaseBuiklder는 직접 빈으로 등록한다고 바로 사용할 수 있는게 아니다. 적절한 메소드를 호출해주는 초기화 코드가 필요하고, 팩토리 빈으로 만드는 것이 좋다.(?)
### 트랜잭션 적용
한번에 여러 sql을 사용할 경우 반드시 같은 트랜잭션 내에서 일어나야 하고 관리가 되어야한다. 
PlatformTransactionManager는 직접 빈으로 만들어 관리해야하지만 지금 경우엔 템플릿/콜백 패턴을 적용한 TransactionTemplate을 쓰면 내부에서 자동으로 트랜잭션을 관리해주기 때문에 좋다. AOP를 통해 만들어지는 트랜잭션 프록시가 같은 트랜잭션 매니저를 공유해야하기 때문에 PlatformTransactionManager는 싱글톤 빈으로 만들어 관리한다.
