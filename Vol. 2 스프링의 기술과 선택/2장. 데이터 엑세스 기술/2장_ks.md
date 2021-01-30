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
