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

