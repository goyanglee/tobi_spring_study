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