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

<br/>


# 7.3 서비스 추상화 적용

<br/>

> #### 서비스 추상화
>
> 로우레벤의 구체적인 기술과 API에 종속되지 않고 추상화된 레이어와 API를 제공해서 구현 기술에 대해 독립적인 코드를 작성할 수 있게 해준다. 

<br/>

### OXM 

스프링에서는 OXM 기술에 대해서 서비스 추상화 기능을 제공한다. OXM은 Object-XML Mapping의 약자로 XML과 자바 오브젝트를 매핑해서 상호 변환해주는 기술이다. 이 매핑 기술의 종류에는 아래와 같이 다섯 종류의 기술이 존재하고 스프링은 이것들에 대한 서비스 추상화 기능을 제공한다. 

- JAXB
- Caster XML
- JiBX
- XmlBeans
- Xstream

<br/>

스프링이 제공하는 OXM 추상화 서비스 인터페이스는 아래의 두 인터페이스가 있다. 

- **Marshaller** : 자바오브젝트를 XML로 변환
- **Unmarshaller** : XML을 자바오브젝트로 변환

OXM 기술에 따라 이 인터페이스를 구현한 다섯가지 클래스가 있다. 예를들어 JAXB를 이용하도록 만들어진 Unmarshaller 구현 클래스는 Jaxb2Marshaller다. @Autowired를 이용해서 Unmarshaller 타입의 인스턴스 변수를 선언해주면 빈을 가져올 수 있다. 

<br/>

### 리소스

스프링은 자바에 존재하는 일관성 없는 리로스 접근 API를 추상화해서 Resource라는 추상화 인터페이스를 정의한다. 

```
package org.springframework.core.io;
...
public interface Resource extends InputStreamSource {
	boolean exists();
	boolean isReadable();
	boolean isOpen();
	
	URL getURL() throws IOException;
	URI getURI() throws IOException;
	File getFile() throws IOException;
	
	Resource createRelative(String relativePath) throws IOException;
	
	long lastModified() throws IOException;
	String getFilename();
	String getDescription();
}

public interface InputStreameSource() {
	InputStream getInputStream() throws IOException;
}
```

스프링의 거의 모든 API는 외부의 리소스 정보가 필요할 때는 항상 이 Resource 추상화를 사용한다. Resource는 스프링에서 빈이 아니라 단순한 정보를 가진 '값'으로 취급된다. 

<br/>

#### 리소스 로더

스프링에서는 Resource 오브젝트를 접두어를 이용해서 선언한다. 문자열 안에 리소스의 종류와 리소스의 위치를 함께 표현한다. 이렇게 문자열로 정의된 리소스를 실제 Resource 타입 오브젝트로 변환해주는 ResourceLoader도 제공한다. 

```
package org.springframework.core.io;

public interface ResourceLoader {
	Resource getResource(String location);
	...
}
```

<br/>

**리소스 로더가 처리하는 접두어**

- file:
  - 예: file:/C:/temp/file.txt
  - 파일 시스템의 C:/temp 폴더에 있는 file.txt를 리소스로 만들어준다. 
- classpath:
  - 예: classpath:file.txt
  - 클래스패스의 루트에 존재하는 file.txt 리소스에 접근하게 해준다. 
- 없음
  - 예: WEB-INF/test.dat
  - 리소스 로더 구현에 따라 리소스의 위치가 결정된다. ServletResourceLoader라면 서블릿 컨텍스트 루트를 기준으로 해석한다. 
- http:
  - 예: http://www.myserver.com/test.dat
  - HTTP 프로토콜을 사용해 접근할 수 있는 웹 상의 리소스를 지정한다. ftp:도 사용할 수 있다. 

<br/>



## 7.4 인터페이스 상속을 통한 안전한 기능확장

<I>애플리케이션을 새로 시작하지 않고 특정 SQL의 내용만을 변경하고 싶다면 어떻게 해야할까?</I>

<br/>

### DI에 적합한 오브젝트 설계가 필요하다!

DI를 적용하려면 커다란 오브젝트 하나만 존재해서는 안되고 최소한 두 개 이상의, 의존관계를 가지고 서로 협력해서 일하는 오브젝트가 필요하다. 그래서 적절한 책임에 따라 오브젝트를 분리해줘야 한다. 그리고 항상 의존 오브젝트는 자유롭게 확장될 수 있다는 점을 생각해야 한다. 

<br/>

#### DI를 제대로 적용하려면 가능한 한 인터페이스를 사용하게 해야 한다. 

- 다형성을 얻을 수 있다. 하나의 인터페이스를 통해 여러 개의 구현을 바꿔가면서 사용할 수 있도록 하는 것

- 인터페이스 분리 원칙을 통해 클라이언트와 의존 오브젝트 사이의 관계를 명확하게 해준다. 

  > #### 인터페이스 분리 원칙
  >
  > 오브젝트가 그 자체로 충분히 응집도가 높은 작은 단위로 설계됐더라고 목적과 관심이 각기 다른 클라이언트가 있다면 인터페이스를 통해 이를 적절하게 분리해줄 필요가 있는데, 이것을 객체지향 설계 원칙에서 인터페이스 분리 원칙이라고 부른다. 

<br/>

#### 관심사가 각기 다른 클라이언트들에게 상속을 통해 각자에 맞는 인터페이스를 제공할 수 있다.

-하나의 오브젝트가 구현하는 인터페이스를 여러 개 만들어서 구분하는 이유 중의 하나는, 오브젝트의 기능이 추가되는 과정에서 다른 관심을 가진 클라이언트가 나타날 수 있기 때문이다. 이 때 각 클라이언트에 맞는 인터페이스를 여러 개 만들어서 제공할 수도 있지만, 기존 인터페이스 상속을 통해 확장해서 제공할 수도 있다. 

<br/>

예를 들어, 아래와 같은 인터페이스를 통해 구성된 구조가 있다. 

![그림7-10](https://github.com/goyanglee/tobi_spring_study/blob/master/Vol.1%20%EC%8A%A4%ED%94%84%EB%A7%81%EC%9D%98%20%EC%9D%B4%ED%95%B4%EC%99%80%20%EC%9B%90%EB%A6%AC/7.%20%EC%8A%A4%ED%94%84%EB%A7%81%20%ED%95%B5%EC%8B%AC%20%EA%B8%B0%EC%88%A0%EC%9D%98%20%EC%9D%91%EC%9A%A9/7%EC%9E%A5_sh/7-10.jpeg)

- BaseSqlService는 SqlRegistry 라는 인터페이스를 통해 MySqlRegistry 클래스의 오브젝트에 접근한다. 따라서 MySqlRegistry의 구현 내용이 변경을 통해 기능이 추가될지라도 BaseSqlService 클래스는 변경 없이 유지될 수 있다. 

<br/>

만약에 MySqlRegistry에 업데이트 기능이 추가되면 그 기능을 사용하고자 하는 새로운 클라이언트가 나타날 수 있다. 이 클라이언트를 위해 SqlRegistry와는 다른 인터페이스가 필요하다. 이 때 인터페이스를 제공할 방법은 두 가지이다. 

1. SqlRegistry와 별개의 인터페이스를 만들어서 제공한다. 
2. 기존 SqlRegistry를 확장한 인터페이스를 이용한다. 

기억할 것은, 클라이언트의 목적과 용도에 적합한 인터페이스만을 제공한다는 인터페이스 분리 원칙을 지키기 위해서 SqlRegistry를 변경하면 안 된다는 것이다. 

<br/>

2번 방식대로 인터페이스를 구성한다면, 아래처럼 만들 수 있다.

```
public interface SqlRegistry {
	void registerSql(String key, String sql);
	String findSql(String key) throws SqlNotFoundException;
}
```

```
public interface UpdatableSqlRegistry extends SqlRegistry {
	public void updateSql(String key, String sql) throws SqlUpdateFailureException;
	public void updateSql(Map<String, String> sqlmap) throws SqlUpdateFailureException;
}
```

- BaseSqlService는 기존의 SqlRegistry 인터페이스를 통해 구현체에 접근하면 되고
- SQL 업데이트 작업이 필요한 새로운 클라이언트 오브젝트는 UpdatableSqlRegistry 인터페이스를 통해 SQL 레지스트리 오브젝트 구현체에 접근하면 된다. (이 클라이언트는 기존 기능인 조회, 등록도 필요한 클라이언트라고 가정한다)

<br/>

최종 의존관계는 아래처럼 된다. 

![그림7-11](https://github.com/goyanglee/tobi_spring_study/blob/master/Vol.1%20%EC%8A%A4%ED%94%84%EB%A7%81%EC%9D%98%20%EC%9D%B4%ED%95%B4%EC%99%80%20%EC%9B%90%EB%A6%AC/7.%20%EC%8A%A4%ED%94%84%EB%A7%81%20%ED%95%B5%EC%8B%AC%20%EA%B8%B0%EC%88%A0%EC%9D%98%20%EC%9D%91%EC%9A%A9/7%EC%9E%A5_sh/7-11.jpeg)

<br/>

#### 정리

BaseSqlService와 SqlAdminService는 동일한 오브젝트에 의존하고 있지만 각자의 관심과 필요에 따라서 다른 인터페이스를 통해 접근한다. 중요한 것은 클라이언트가 정말 필요한 기능을 가진 인터페이스를 통해 오브젝트에 접근하도록 만들었는 가이다. 

<br/>


