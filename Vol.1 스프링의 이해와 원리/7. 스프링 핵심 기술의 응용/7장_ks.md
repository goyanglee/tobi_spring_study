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
