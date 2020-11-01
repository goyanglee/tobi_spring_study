# 7장. 스프링 핵심 기술의 응용

# 7.1 SQL과 DAO의 분리

DB테이블과 필드정보를 고스란히 담고 있는 SQL 문장들의 분리

## 7.1.1 XML 설정을 이용한 분리

### 개별 SQL 프로퍼티 방식

xml로 sql를 만들어서, xml을 이용하는 방식 

### SQL 맵 프로퍼티 방식

userDao에서 SQL을 주입받기 위해 개별적으로 정의한 프로퍼티를 모두 제거하고, Map타입의 sqlMap프로퍼티를 대신 추가한다.

map에서는 xml을 이용

```xml
<bean id="userDao" class="springbook.user.dao.UserDaoJdbc">
  <property name="dataSource" ref="dataSource" />
    <property name="sqlMap">
      <map>
        <entry key="get" value=" select * from users order by id" />
        .
        .
        .
      </map>
    </property>
</bean>
```

새로운 sql문이 필요하면 <entry>만 추가하면 된다.

## 7.1.2 SQL 제공 서비스

### SQL 서비스 인터페이스

```java
public interface SqlService {
    String getSql(String key) throws SqlRetrievalFailureException;
}
```

어떤 이유에서는 sql을 가져오다가 실패하는 경우에는 SqlRetrievalFailureException 예외를 던지도록 정의한다.

```java
public class SqlRetrievalFailureException extends RuntimeException {
    public SqlRetrievalFailureException() {
    }

    public SqlRetrievalFailureException(String message) {
        super(message);
    }

    public SqlRetrievalFailureException(String message, Throwable cause) {
        super(message, cause);
    }
}
```

### 스프링 설정을 사용하는 단순 SQL 서비스

어떤 방법을 사용하든 상관없이 DAO가 요구하는 SQL을 돌려주기만 하면된다.

맵에서 sql을 읽어서 돌려주도록 sqlService의 getSql() 메소드를 구현해보자.

```java
public class SimpleSqlService implements SqlService {

    private final Map<String, String> sqlMap;

    public SimpleSqlService(Map<String, String> sqlMap) {
        this.sqlMap = sqlMap;
    }

    @Override
    public String getSql(String key) throws SqlRetrievalFailureException {
        String sql = sqlMap.get(key);
        if (sql == null) {
            throw new SqlRetrievalFailureException(key + "에 대한 SQL을 찾을수 없습니다");
        }
        return sql;
    }
}
```

그리고 sql 정보는 맵을 이용해 등록하여 사용한다.

# 7.2 인터페이스 분리와 자기 참조 빈

## 7.2.1 XML 파일 매핑

검색용 키와 SQL 문장 두 가지를 담을 수 있는 간단한 XML 문서를 설계해보고, 이 XML 파일에서 SQL을 읽어뒀다가 DAO에게 제공해주는 SQL 서비스 구현 클래스를 만들어보자.

### JAXB

XML에 담긴 정보를 파일에서 읽어오는 방법으로 JAXB(Java Architecture for XML Binding를 이욯가ㅔㅆ다.

JDK 6 라면 java.xml.bind 패키지 안에서 JAXB의 구현 클래스를 찾을 수 있다. 

JAXB의 장점은 XML 문서 정보를 거의 동일한 구조의 오브젝트로 직접 매핑해준다는 것이다.

전통적인 XML API인 DOM과 비교했을 때, DOM은 XML 정보를 마치 자바의 리플렉션 API를 사용해서 오브젝트를 조작하는 것처럼 간접적으로 접근해야 하는 불편이 있지만,

그에 비해 JAXB는 XML의 정보를 그대로 담고 있는 오브젝트 트리 구조를 만들어주기 때문에 XML 정보를 오브젝트처럼 다룰 수 있어 편리하다.

JAXB는 XML 문서의 구조를 정의한 스키마를 이용해서 매핑할 오브젝트의 클래스까지 자동으로 만들어주는 컴파일러도 제공해준다. 스키마 컴파일러를 통해 자동생성된 오브젝트에는 매핑정보가 어노테이션으로 담겨 있다.

JAXB API는 어노테이션에 담긴 정보를 이용해서 XML과 매핑된 오브젝트 트리 사이의 자동변환 작업을 수행해준다.

![7%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%89%E1%85%B3%E1%84%91%E1%85%B3%E1%84%85%E1%85%B5%E1%86%BC%20%E1%84%92%E1%85%A2%E1%86%A8%E1%84%89%E1%85%B5%E1%86%B7%20%E1%84%80%E1%85%B5%E1%84%89%E1%85%AE%E1%86%AF%E1%84%8B%E1%85%B4%20%E1%84%8B%E1%85%B3%E1%86%BC%E1%84%8B%E1%85%AD%E1%86%BC%20884f0f0f28ae45f894c33a139e2b08d7/Untitled.png](7%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%89%E1%85%B3%E1%84%91%E1%85%B3%E1%84%85%E1%85%B5%E1%86%BC%20%E1%84%92%E1%85%A2%E1%86%A8%E1%84%89%E1%85%B5%E1%86%B7%20%E1%84%80%E1%85%B5%E1%84%89%E1%85%AE%E1%86%AF%E1%84%8B%E1%85%B4%20%E1%84%8B%E1%85%B3%E1%86%BC%E1%84%8B%E1%85%AD%E1%86%BC%20884f0f0f28ae45f894c33a139e2b08d7/Untitled.png)

### SQL 맵을 위한 스키마 작성과 컴파일

```xml
<sqlmap>
   <sql key="userAdd"> insert into users(...) ... </sql>
   <sql key="userGet"> select * from users ... </sql>
</sqlmap>
```

```xml
<element name="sqlmap">
   <complexType>
      <sequence>
         <element name="sql" maxOccurs="unbounded" type="tns:sqlType" />
      </sequence>
   </complexType>
</element>
```

<sql> 태그를 가진 XML 문서와 XML 문서 구조를 정의하고 있는 XML 스키마.

sqlService를 정의한 패키지 아래에 jaxb패키지를 추가한 뒤에 이를 사용하자.

셸이나 도스창에서 프로젝트 루트 폴더로 이동한 뒤에 다음 명령을 사용해 컴파일하면 된다.

### 언마샬링

언마샬링 : xml문서를 읽어서 자바의 오브젝트로 변환하는 것

마샬링 : 바인딩 오브젝트를 XML 문서로 변환하는 것

직렬화 : 자바오브젝틀 바이트 스트림으로 바꾸는 것

## 7.2.2 XML 파일을 이용하는 SQL 서비스

### SQL맵 XML 파일