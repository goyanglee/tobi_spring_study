# 1장. 오브젝트와 의존관계

스프링 핵심 철학 = 객체지향 기술의 진정한 가치를 회복시키고 그로부터 객체지향의 기본을 돌아가자는 것.

앞으로 주의 깊게 살펴봐야 할 것

- 애플리케이션에서 오브젝트가 생성되고 다른 오브젝트와 관계를 맺고, 사용되고, 소멸하기까지의 전 과정
- 오브젝트는 어떻게 설계되어야 하는 지
- 어떤 단위로 만들어지며 어떤 과정을 통해 자신의 존재를 드러내고 등장해야 하는 지

## 1.1 초난감 DAO

### DAO(Data Access Object)

: DB를 사용해 데이터를 조회하거나 조작하는 기능을 전담하도록 만든 오브젝트를 말한다.

### 1.1.1 User

### 자바빈(빈)

요즘의 자바빈은 두가지 관례에 따라 만들어진 오브젝트를 가리킨다.

- 디폴트 생성자 : 자바빈은 파라미터가 없는 디폴트 생성자를 갖고 있어야 한다. 툴이나 프레임워크에서 리플렉션을 이용해 오브젝트를 생성하기 때문에 필요하다.
- 프로퍼티 : 자바빈이 노출하는 이름을 가진 속성을 프로퍼티라고 한다. 프로퍼티는 set으로 시작하는 수정자 메소드(setter)와 get으로 시작하는 접근자 메소드(getter)를 이용해 수정 또는 조회할 수 있다.

### 1.1.2 UserDao

JDBC를 이용하는 작업의 일반적인 순서

1. DB 연결을 위한 connection을 가져온다.
2. SQL을 담은 Statement(또는 PreparedStatement)를 만든다.
3. 만들어진 Statement를 실행한다.
4. 조회의 경우 SQL 쿼리의 실행 결과를 ResultSet으로 받아서 정보를 저장할 오브젝트(ex.User)에 옮겨준다.
5. 작업 중에 생성된 Connection, Statement,ResultSet 같은 리소스는 작업을 마친 후 반드시 닫아준다.
6. JDBC API가 만들어내는 예외(Exception)을 잡아서 직접 처리하거나, 메소드에 throws를 선언해서 예외가 발생하면 메소드 밖으로 던지게 한다.

예외는 메소드 밖으로 던져버리는 편이 간단하다.

```java
public void add(User user) throws ClassNotFoundException, SQLException {
	Class.forName("com.mysql.jdbc.Driver");
  Connection c = DriverManager.getConnection(
            "jdbc:mysql://localhost/spring","spring","book");

  PreparedStatement ps = c.prepareStatement(
       "insert into users(id,name,password) values(?,?,?)");
 
  ps.setString(1, user.getId());
  ps.setString(2, user.getName());
  ps.setString(3, user.getPassword();

  ps.executeUpdate();

  ps.close();
  c.close();
}
```

### 1.1.3 main()을 이용한 DAO 테스트 코드

static main() : 모든 클래스에 자신을 엔트리 포인트로 설정해 직접 실행이 가능하게 해주는 것

사용할 DB의 드라이버를 클래스패스에 넣어주는 일 : db connector jar 파일.

스프링을 내가 공부해야 하는 이유

1. 코드에 문제가 많은가?
2. 많다면 왜 많은가?
3. 잘 동작하는 코드를 굳이 수정하고 개선해야 하는 이유는 뭘까?
4. 개선했을 때의 장점은 무엇일까?
5. 장점들이 당장에, 미래에 주는 유익은 무엇인가?
6. 또, 객체지향 설계의 원칙과는 무슨 상관이 있을까?
7. 개선하는 경우와 그대로 사용하는 경우, 스프링을 사용하는 개발에는 무슨 차이가 있을까?

와 같은 문제 제기와 의문에 대한 답을 찾아가는 과정.

## 1.2 DAO 분리

### 1.2.1 관심사의 분리

> 분리와 확장을 고려한 설계

* 분리 :  관심사의 분리

: 관심이 같은 것끼리는 하나의 객체 안으로 또는 친한 객체로 모이게 하고, 관심이 다른 것은 가능한 따로 떨어져서 서로 영향을 주지 않도록 분리해야 하는 것 

ex) 모든 dao에 db접속 정보코드를 놓는다면, db 접속 비밀번호가 바뀌면 dao 전부를 수정해야 한다.

### 1.2.2 커넥션 만들기의 추출