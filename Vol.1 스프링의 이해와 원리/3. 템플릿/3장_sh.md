# 3장. 템플릿



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

![image](Vol.1 스프링의 이해와 원리/3. 템플릿/3-2_전략패턴의_구조.jpeg)



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

(그림 3-3)



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
