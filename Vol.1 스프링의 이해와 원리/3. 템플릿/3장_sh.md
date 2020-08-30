# 3장 템플릿



## 3.1 다시보는 초난감 DAO

일반적으로 서버에서는 매번 새로운 DB 커넥션을 생성하지 않고, 미리 제한된 수의 커넥션을 풀에 만들어두고 필요할 때 가져가서 사용하고 반환하면 풀에 다시 넣는 방식으로 관리한다. 따라서 이러한 DB나 파일 같은 리소스들은 반환하는 과정이 반드시 필요하다. 특히 장시간 운영되는 다중 사용자를 다루는 서버는 무조건! 풀에 반환되지 않는 리소스가 늘어날 수록 서버가 뻗어버릴 가능성이 커진다. 

### "앞 단의 코드는 리소스를 반환하는 것에 대한 처리를 고려하지 않았다. 어떤 상황에서도 가져온 리소스를 반환하도록 즉, 예외상황에서도 리소스는 무조건 반환할 수 있도록 try/catch/finally를 적용하자. "

```
public void deleteAll() throws SQLException {
	Connection c = null;
	PreparedStatement ps = null;
	
	try {
		c = dataSource.getConnection();
		ps = c.preparedStatement("delete from users");
		ps.executeUpdate();
	} catch (Exception e) {
		throw e; 
	} finally {
		if (ps != null) {
			try {
				ps.close();
			} catch (SQLException e) {
				//로그기록
			}
		}
		
		if (c != null) {
			try {
				c.close();
			} catch (Exception e) {
				//로그기록
			}
		}
	}
}
```

#



## 3.2 변하는 것과 변하지 않는 것

위에 작성한 코드에는 또 다시 문제점이 존재한다. 위와 같은 dao가 많아질수록 try/catch/finally의 반복이 심해지고, 만약 close() 하는 부분을 빼먹고 반복되는 try/catch/finally 블록을 복붙해서 사용해버리면 할당된 커넥션 리소스들이 반환되지 않은 상태로 존재하게 되어 나중에는 DB 커넥션 풀이 꽉 찼다는 에러를 내면서 서버가 중단될 수도 있다. 

### "변하지 않는, 많은 곳에서 중복되는 코드와 로직에 따라 계속 확장되고 자주 변하는 코드를 잘 분리해내는 작업이 필요하다! 전략패턴을 적용하자. "



> ### 전략패턴
>
> 개방폐쇄원칙 관점에서 보면 확장에 해당하는 변하는 부분을 별도의 클래스로 만들어 추상화된 인터페이스를 통해 위임하는 방식이다. 



<B>전략패턴의 구조</B>

![이미지1](https://github.com/goyanglee/tobi_spring_study/blob/master/Vol.1%20%EC%8A%A4%ED%94%84%EB%A7%81%EC%9D%98%20%EC%9D%B4%ED%95%B4%EC%99%80%20%EC%9B%90%EB%A6%AC/3.%20%ED%85%9C%ED%94%8C%EB%A6%BF/3%EC%9E%A5_sh/3-2_%E1%84%8C%E1%85%A5%E1%86%AB%E1%84%85%E1%85%A3%E1%86%A8%E1%84%91%E1%85%A2%E1%84%90%E1%85%A5%E1%86%AB%E1%84%8B%E1%85%B4_%E1%84%80%E1%85%AE%E1%84%8C%E1%85%A9.jpeg)



### 1) 변하는 코드를 분리해서 전략으로 만들기 : PrepareStatement를  생성하는 부분을 분리한다.  

```
public interface StatementStrategy {
	PreparedStatement makePreparedStatement(Connection c) throws SQLException;
}
```

```
public class DeleteAllStatement implements StatementStrategy {
	public PreparedStatement makePreparedStatement(Connection c) throws SQLException {
		PreparedStatement ps = c.prepareStatement("delete from users");
		return ps; 
	}
}
```



### 2) 'deleteAll()' 컨텍스트는 'DeleteAllStatement' 전략을 사용해서 구현된다. 

```
public void deleteAll() throws SQLException {
	Connection c = null;
	PreparedStatement ps = null;
	
	try {
		c = dataSource.getConnection();
		
		//전략을 사용하는 코드로 변경
		//ps = c.preparedStatement("delete from users");
		StatementStategy strategy = new DeleteAllStatement();
		ps = strategy.makePreparedStatement(c);
		
		ps.executeUpdate();
	} catch (Exception e) {
		throw e; 
	} finally {
		if (ps != null) {
			try {
				ps.close();
			} catch (SQLException e) {
				//로그기록
			}
		}
		
		if (c != null) {
			try {
				c.close();
			} catch (Exception e) {
				//로그기록
			}
		}
	}
}
```



### 3) 전략패턴에 따르면 컨텍스트가 어떤 전략을 사용하게 할 것인가는 컨텍스트를 사용하는 앞단의 클라이언트가 결정하는 게 일반적이다. '2)'의 코드를 컨텍스트/클라이언트로 분리한다. 즉, DeleteAllStatement 전략을 생성하는 부분을 클라이언트의 책임으로 넘긴다. 

![이미지2](https://github.com/goyanglee/tobi_spring_study/blob/master/Vol.1%20%EC%8A%A4%ED%94%84%EB%A7%81%EC%9D%98%20%EC%9D%B4%ED%95%B4%EC%99%80%20%EC%9B%90%EB%A6%AC/3.%20%ED%85%9C%ED%94%8C%EB%A6%BF/3%EC%9E%A5_sh/3-3_%E1%84%8C%E1%85%A5%E1%86%AB%E1%84%85%E1%85%A3%E1%86%A8%E1%84%91%E1%85%A2%E1%84%90%E1%85%A5%E1%86%AB%E1%84%8B%E1%85%A6%E1%84%89%E1%85%A5_%E1%84%8F%E1%85%B3%E1%86%AF%E1%84%85%E1%85%A1%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%A5%E1%86%AB%E1%84%90%E1%85%B3%E1%84%8B%E1%85%B4_%E1%84%8B%E1%85%A7%E1%86%A8%E1%84%92%E1%85%A1%E1%86%AF.jpeg)



```
/////////////////////////////////////////컨텍스트/////////////////////////////////////////

public void jdbcContextWithStatementStrategy(StatementStrategy stmt) throws SQLException {
	Connection c = null;
	PreparedStatement ps = null;
	
	try {
		c = dataSource.getConnection();
		
		//클라이언트로부터 넘겨받은 전략을 사용하도록 변경
		//ps = strategy.makePreparedStatement(c);
		ps = stmt.makePreparedStatement(c);
		
		ps.executeUpdate();
	} catch (Exception e) {
		throw e; 
	} finally {
		if (ps != null) {
			try {
				ps.close();
			} catch (SQLException e) {
				//로그기록
			}
		}
		
		if (c != null) {
			try {
				c.close();
			} catch (Exception e) {
				//로그기록
			}
		}
	}
}
```

```
/////////////////////////////////////////클라이언트/////////////////////////////////////////
public void deleteAll() throws SQLException {
	StatementStrategy st = new DeleteAllStatement(); 
	jdbcContextWithStatementStrategy(st);
}
```

#



## 3.3 JDBC 전략 패턴의 최적화

> ### 중첩 클래스 (nested class)
>
> 다른 클래스 내부에 정의되는 클래스
>
> - 스태틱 클래스 (static class) : 독립적으로 오브젝트로 만들어질 수 있는 클래스
> - 내부 클래스 (inner class) : 자신이 정의된 클래스의 오브젝트 안에서만 만들어질 수 있는 클래스
>   - 멤버 내부 클래스 : 멤버 필드처럼 오브젝트 레벨에 정의됨
>   - 로컬 클래스 : 메소드 레벨에 정의됨
>   - 익명 내부 클래스 : 이름을 갖지 않으며, 선언된 위치에 따라 범위가 다르다. 



### 위에서 만든 전략패턴 구조의 문제점?

- DAO 메소드마다 새로운 StatementStrategy 구현 클래스를 만들어야 한다는 점
- DAO 메소드에서 StatementStrategy에 전달할 부가적인 정보가 있는 경우(도메인 등), 이를 위해 오브젝트를 전달받는 생성자와 이를 저장해둘 인스턴스 변수를 전략클래스에 번거롭게 만들어둬야 한다는 점 



### "익명 내부 클래스를 사용해서 코드를 최적화하자. "

```
public void deleteAll() throws SQLException {
	jdbcContextWithStatementStrategy(
		new StatementStrategy() {
			public PreparedStatement makePreparedStatement(Connection c) throws SQLException {
				return c.preparedStatement("delete from users");
			}
		}
	)
}
```

- 전략클래스로 파라미터 전달이 필요하다면, deleteAll() 메소드가 선언된 클래스에서 사용하는 값을 전달해줄 수 있다. 
#

# 3.4 컨텍스트와 DI

jdbcContextWithStatementStrategy()는 UserDao 외에 다른 DAO에서도 사용할 수 있기 때문에 별도 클래스로 분리했고, 필요한 파일을 정리하면 아래와 같다. 

- JdbcContext.java : JDBC 작업흐름을 담고 있음
- UserDao.java : 분리된 JdbcContext를 주입받아서 사용
- applicationContext.xml : 설정파일



! UserDao는 인터페이스를 거치지 않고 코드에서 바로 JdbcContext를 사용하고 있다. 둘은 클래스 레벨에서 의존관계가 설정된다. 즉, 스프링 기본 의도와는 다르게 인터페이스를 사용하지 않고 DI를 적용하고 있다. 이것은 엄밀히 말해 온전한 DI라고 할 수는 없지만, DI의 기본은 따른 것으로 본다. 

! 이렇게 인터페이스가 없다는 것은 UserDao와 JdbcContext가 매우 긴밀하게 결합되어 항상 함께 사용되어야 한다는 의미이고, 종종 사용되는 방법이긴 하지만 가장 마지막 단계에 고려해야 하는 방법임을 주의해야한다. 



### "인터페이스를 사용하지 않고 DAO와 밀접한 관계를 갖는 클래스를 DI에 적용하는 방법은 두 가지 존재한다. " 



#### 방법 1 : JdbcContext를 스프링 빈으로 등록해서 UserDao에 주입한다. 

> ### JdbcContext를 UserDao와 DI 구조로 만들어야 하는 이유? 
>
> - JdbcContext가 스프링 컨테이너의 싱글톤 레지스트리에서 관리되는 싱글톤 빈이기 때문이다. 
>   - JdbcContext는 무상태 빈으로, JDBC 컨텍스트 메소드를 제공해주는 일종의 서비스 오브젝트이므로 싱글톤으로 등록돼서 여러 오브젝트가 공유해서 사용하는 것이 이상적이다. 
> - JdbcContext가 DI를 통해 다른 빈(DataSource)에 의존하고 있기 때문이다. 
>   - DI를 위해서는 주입되는 오브젝트와 주입받는 오브젝트 양쪽 모두 스프링 빈으로 등록되어야 한다. 스프링이 생성하고 관리하는 IoC 대상이어야 DI에 참여할 수 있다. 



#### 방법 2 : JdbcContext를 UserDao 내부에서 직접 주입한다. 그러려면?

- 싱글톤으로 만들지 못한다. DAO 마다 하나의 JdbcContext 오브젝트를 갖고 있게 한다. UserDao가 JdbcContext 즉, 자신이 사용할 오브젝트를 직접 만들고 초기화한다. 
- JdbcContext를 주입하는 부분도 UserDao가 진행한다. 이 때, JdbcContext에 주입해줄 의존 오브젝트인 DataSource는 UserDao가 대신 주입받도록 하면된다. 



#### * 각각의 장단점 * 

<스프링 빈으로 등록하는 방법>

- 장점) 오브젝트 사이의 실제 의존관계가 설정파일에 명확하게 드러난다. 
- 단점) DI의 근본적인 원칙에 부합하지 안는 구체적인 클래스와의 관계가 설정에 직접 노출된다.



<DAO 코드 내에서 직접 주입하는 방법>

- 장점) 관계가 외부에 드러나지 않는다. 
- 단점) JdbcContext를 여러 오브젝트가 사용하더라도 싱글톤으로 만들 수 없고, DI 작업을 위한 부가적인 코드가 필요하다. 


