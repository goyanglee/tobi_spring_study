# 7장 스프링 핵심 기술의 응용

<br/>

## SQL과 DAO의 분리

운영 최적화를 위해 DAO에서 SQL 로직은 분리하는 것이 좋다. 

<br/>

### 분리방식

#### -XML 설정을 이용한 분리

설정파일의 ```<bean>``` 태그 안에 ```<property>``` 태그 형식으로 sql이 포함되어 있는 형태이다. 

- 개별 SQL 프로퍼티 방식
- SQL 맵 프로퍼티 방식

> SQL 문장이 ```<bean>``` 과 같은 설정정보 안에 함께 섞여서 존재하는 건 비효율적이다. 

<br/>

#### -SQL 서비스 인터페이스 설계를 통한 분리

sql을 독립적으로 다루는 인터페이스를 별도로 만들어서 설정파일에 선언하는 방식이다. 

```
public interface SqlService {
	String getSql(String key) throws SqlRetrievalFailureException;
}
```

```
public class SimpleSqlService implements SqlService {
	private Map<String, String> sqlMap; 
	
	public void setSqlMap(Map<String, String> sqlMap) {
		this.sqlMap = sqlMap;
	}
	
	public String getSql(String key) throws SqlRetrievalFailureException {
		String sql = sqlMap.get(key);
		if (sql == null) 
			throw new SqlRetrievalFailureException(key + "에 대한 SQL을 찾을 수 없습니다");
		else
			return sql;
	}
}
```

<br/>

```
public class UserDaoJdbc implements UserDao {
	...
	private SqlService sqlService;
	public void setSqlService(SqlService sqlService) {
		this.sqlService = sqlService; 
	}
}
```

```
public class UserDao {
	...
	public void add(User user) {
		this.jdbcTemplate.update(this.sqlService.getSql("userAdd"), 
		user.getId, user.getName(), user.getPassword(), user.getEmail());
	}
	...
}
```

<br/>

```
<bean id="userDao" class="com.sh.UserDaoJdbc">
	<property name="dataSource" ref="dataSource" />
	<property name="sqlService" ref="sqlService" />
</bean>

<bean id="sqlService" class="com.sh.sqlservice.SimpleSqlService">
	<property name="sqlMap">
		<map>
			<entry key="userAdd" value="insert into users(id, name, password, email) values (?,?,?,?)" />
			<entry key="userGet" value="select * from users where id = ?" />
			...
		</map>
	</property>
</bean>
```

<br/>

## 인터페이스의 분리와 자기참조 빈 (SqlService 인터페이스 구현방법)

<br/>

<I>-검색용 키와 SQL 문장 두 가지를 담을 수 있는 간단한 XML 문서를 설계해보고</I>

<I>-이 XML 파일에서 SQL을 읽어뒀다가 DAO에게 제공해주는 SQL 서비스 구현 클래스를 만든다. </I>

<br/>

### XML 파일 매핑

스프링의 XML 설정파일에서 <bean> 태그 안에 SQL 정보를 넣어두고 활용하는 건 좋은 방법이 아니다. 그보다는 SQL을 저장해두는 전용 포맷을 가진 독립적인 파일을 이용하는 것이 좋다. JAXB를 사용해서 XML 변환. 

<br/>

> #### JAXB (Java Architecture for XML Binding)
>
> - XML의 정보를 그대로 담고 있는 오브젝트 트리구조로 만들어준다. XML정보를 오브젝트로 관리할 수 있다. 
> - SQL 스키마 파일(sqlmap.xml)을 만들고 JAXB 컴파일러로 컴파일한다. 

<br/>

### XML SQL 서비스

SQL 스키마 파일에 있는 SQL을 가져와 DAO에 제공해주는 SqlService 인터페이스의 구현클래스를 만든다. XML 문서에서 SQL을 가져올 때도 JAXB API를 사용하면된다.

<br/>

**[구현내용]**

- 생성자에서 JAXB를 이용해 XML로 된 SQl 문서를 읽어들이고
- 변환된 Sql 오브젝트들을 맵으로 옮겨서 저장해뒀다가
- DAO 요청에 따라 SQL을 찾아서 전달

<br/>

### 빈의 초기화 작업

생성자에서 문서를 읽어들이는 부분을 수정할 필요가 있다. 생성자에서 예외가 발생할 수도 있는 복잡한 초기화 작업을 다루는 것은 좋지 않다. 일단 초기 상태를 가진 오브젝트를 만들어두고 별도의 초기화 메소드를 사용하는 것이 좋다. ```@PostConstruct```

> #### @PostConstruct
>
> - 빈 오브젝트의 초기화 메소드를 지정하는 데 사용하는 어노테이션

<br/>

### 책임에 따른 인터페이스 정의

분리 가능한 관심사를 구분해서 정의하는 것이 좋다. SqlService의 구현 클래스가 변경 가능한 책임을 가진 SqlReader와 SqlRegistry 두 가지 타입의 오브젝트를 사용하도록 만든다.

- SQL 정보를 외부의 리소스로부터 읽어오는 부분 -> 별도 인터페이스화 (SqlRegistry)
- 읽어온 SQL을 보관해두고 있다가 필요할 때 제공해주는 부분 -> 별도 인터페이스화 (SqlReader)

<br/>



### 자기참조 빈

XmlSqlService 클래스 하나가 SqlService, SqlReader, SqlRegistry라는 세 개의 인터페이스를 구현하게 만들어도 상관없다. 

