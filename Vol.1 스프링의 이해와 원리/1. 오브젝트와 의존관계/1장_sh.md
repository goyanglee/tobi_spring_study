

# 1장 오브젝트와 의존관계 



## 1.1 초난감 DAO

> <B>DAO</B> (= Data Access Object)
>
> DB를 사용해서 데이터를 조회하거나 조작하는 기능을 전담하는 오브젝트



<I>- 난감한 코드를 작성하고, 이 코드를 객체지향 기술 원리에 충실한 스프링 스타일의 코드로 개선하는 작업을 할 예정 - </I>



## 1.2 DAO의 분리

개발자가 객체를 설계할 때 가장 염두해 둬야 할 사항은 <B>미래의 변화를 어떻게 대비할 것인가</B>이다. 객체지향 설계와 프로그래밍이 절차지향에 비해 초디에 좀 더 번거로운 작업을 요구하는 이유는 객체지향 기술 자체가 지니는, 변화에 효과적으로 대처할 수 있다는 기술적인 특징 때문이다. 객체 지향 기술은 흔히 실세계를 최대한 가깝게 모델링해낼 수 있기 떄문에 의미가 있다고 여겨진다. 하지만 그보다는 객체지향 기술이 만들어내는 가상의 추상세계 자체를 효과적으로 구성할 수 있고, 이를 자유롭고 편리하게 변경, 발전, 확장시킬 수 있다는 데 더 의미가 있다. 



<I>어떻게 변경이 일어났을 때 필요한 작업을 최소화하고, 그 변경이 다른 곳에 문제를 일으키지 않게 할 수 있을까?</I>

<B>"분리와 확장을 고려해서 설계해야 한다!"</B>



### 1.2.1 관심사의 분리 

프로그래밍의 기초 개념. 객체 지향에 적용해보면, <B>관심이 같은 것끼리는 하나의 객체 안으로 또는 친한 객체로 모이게</B> 하고, 관심이 다른 것은 가능한 한 따로 떨어져서 서로 영향을 주지 않도록 분리하는 것



### 1.2.2 커넥션 만들기의 추출

#### 중복 코드의 메소드 추출 <font color="red">(독립된 메소드를 만들어서 분리)</font>

하나의 클래스 안에 존재하는 중복코드를 하나의 메소드로 추출한다. 

- 전 : UserDao 클래스의 ```add()```, ```get()``` 메소드가 있고 메소드마다 "DB 커넥션 연결"이라는 관심사가 존재
- 후 : 각 메소드에 존재했던 "DB 커넥션 연결"이라는 관심을 분리해서 공통 메소드 ```getConnection()``` 에서 처리



### 1.2.3 DB 커넥션 만들기의 독립

<I>UserDao 소스코드를 N사와 D사에 공개하지 않고 고객 별로 원하는 DB 커넥션 생성 방식을 적용해서 UserDao를 사용하게 할 수 있을까? </I>

#### 상속을 통한 확장 <font color="red">(상속을 통해 분리)</font>

```getConnection()``` 안의 구현 코드를 제거하고 UserDao의 추상 메소드로 만들어둔다.

- N사와 D사는 UserDao 클래스를 상속해서 각각 서브클래스를 만들고 그 안에서 ```getConnection()``` 메소드를 원하는대로 구현해서 사용하면 된다. 커넥션 연결 기능을 자유롭게 확장할 수 있게 함

- 1.2.2에서 "커넥션 연결"이라는 관심을 같은 클래스에 있는 다른 메소드로 분리했지만 이번에는 그 기능을 상속을 통해 아예 다른 서브 클래스로 분리해버림
  - UserDao : 어떻게 데이터를 가져올 것인가? (SQL 작성, 파라미터 바인딩, 쿼리 실행, 검색정보 전달)
  - NUserDao/DUserDao : DB 연결은 어떻게 할 것인가? 



<B>[정리]</B> 1.2.2와 1.2.3의 "관심사 분리" 작업을 거쳐서 UserDao는 단순히 변경이 용이하다 라는 수준을 넘어서 손쉽게 확장된다라고 말할 수 있게되었다. 새로운 DB 연결 방법을 적용해야 할 때는 UserDao 상속을 통해 확장해주면 된다. 



> <B>템플릿 메소드 패턴</B>
>
> 슈퍼클래스에 기본적인 로직의 흐름을 만들고, 그 기능의 일부를 추상 메소드나 오버라이딩이 가능한 protected 메소드 등으로 만든 뒤 서브클래스에서 이런 메소드를 필요에 맞게 구현해서 사용하도록 하는 디자인 패턴. 스프링에서 애용되는 패턴
>
> - 상속을 통해 슈퍼클래스의 기능을 확장할 때 사용하는 방법
> - 변하지 않는 기능은 슈퍼클래스에 만들어두고 자주 변경되며 확장할 기능은 서브클래스에서 만들도록 한다. 



> <B>팩토리 메소드 패턴</B>
>
> 서브클래스에서 구체적인 <B>오브젝트 생성방법을 결정</B>하게 하는 디자인 패턴
>
> - ```getConnection()``` 메소드는 Connection 타입 오브젝트를 생성한다는 기능을 정의해놓은 추상 메소드이고 UserDao 서브 클래스의 ```getConnection()``` 메소드는 어떤 Connection 클래스의 오브젝트를 어떻게 생성할 것인 지를 결정하는 방법이라고 볼 수 있다. 
> - 템플릿 메소드 패턴과 마찬가지로 상속을 통해 기능을 확장하게 하는 방법



#### 단점

- 상속 자체가 가지고 있는 한계점
  - 자바는 클래스의 다중상속을 허용하지 않는다. 
    - NUserDao 클래스가 이미 다른 클래스를 상속 중이라면 UserDao를 상속할 수 없음 
  - 상속을 통한 상하위 클래스의 관계는 생각보다 밀접하다. 
    - 슈퍼클래스의 변경이 있을 때 모든 서브 클래스를 함께 수정하거나 다시 개발해야할 수도 있다. 
  - 확장된 기능인 DB 커넥션을 생성하는 코드를 다른 DAO 클래스에 적용할 수 없다. 
    - UserDao 외의 DAO 클래스들이 계속 만들어지면 상속을 통해 만들어진 getConnection() 구현 코드가 매 DAO 클래스마다 중복된다. 



## 1.3 DAO의 확장

상속 말고 다른 방법으로 확장해보자!



### 1.3.1 클래스의 분리

- DB 커넥션 관심을 서브클래스가 아니라 별도의 클래스에 작성하고, 이 클래스를 UserDao가 이용하게함

```
public class UserDao {
	private SimpleConnectionMaker simpleConnectionMaker;
	
	public UserDao() {
		simpleConnectionMaker = new SimpleConnectionMaker();
		//각 메소드에서 매번 생성하지않고 한 번만 만들어서 저장해두고 이것을 계속 사용함
	}
	
	public void add(User user) throws ClassNofFoundException, SQLException {
		Connection c = simpleConnectionMaker.makeNewConnection();
		...
	}
	
	public User get(String id) throws ClassNotFoundException, SQLException {
		Connection c = simpleConnectionMaker.makeNewConnection();
		...
	}
}

public class SimpleConnectionMaker {
//상속을 이용한 확장방식을 사용하지 않으므로 추상 클래스로 만들 필요가 없다 
	public Connection makerNewConnection() thrwos ClassNotFoundExceiption, SQLException {
		Class.forName("com.mysql.jdbc.Driver");
		Connection c = DriverManager.getConnection("jdbc:mysql://localhost/sh", "sh", "ite");
		return c;
	}
}
```

<B>[문제점]</B> N사와 D사에 UserDao 클래스만 제공하고 상속을 통해 DB 커넥션 기능을 확장해서 사용하게 했던 것이 다시 불가능해졌다. UserDao의 코드가 SimpleConnectionMaker라는 특정 클래스에 종속되어 있기 때문에 상속을 사용했을 때처럼 UserDao 코드의 수정 없이 DB 커넥션 생성 기능을 변경할 방법이 없다. 



### 1.3.2 인터페이스의 도입

> <B>추상화</B>
>
> 어떤 것들의 공통적인 성격을 뽑아내어 이를 따로 분리해내는 작업
>
> 이러한 추상화를 위해 자바에서 제공하는 도구 - 인터페이스



> <B>인터페이스</B>
>
> 어떤 일을 하겠다는 기능만 정의해 놓음
>
> 어떻게 하겠다는 구현 방법은 인터페이스를 구현한 클래스들이 알아서 결정함



<I>UserDao가 인터페이스를 사용하게 한다면 인터페이스의 메소드를 통해 알 수 있는 기능에만 관심을 가지면 되지, 그 기능을 어떻게 구현했는 지에는 관심을 둘 필요가 없다!</I>



```
public interface ConnectionMaker {
	public Connection makeConnection() throws ClassNotFoundException, SQLException; 
}
```



```
public Class DConnectionMaker implements ConnectionMaker {
	public Connection makeConnection() throws ClassNotFoundException, SQLException {
		//D사의 독자적인 방법으로 Connection을 생성하는 코드
		...
	}
}
```



```
public class UserDao {
	private ConnectionMaker connectionMaker;
	//인터페이스를 통해 오브젝트에 접근하므로 구체적인 클래스 정보를 알 필요가 없음
	
	public UserDao() {
		connectionMaker = new DConnectionMaker(); 
		//(!) 클래스이름이 나오는 문제
	}
	
	public void add(User user) throws ClassNotFoundException, SQLException {
		Connection c = connectionMaker.makeConnection(); 
		//인터페이스에 정의된 메소드를 사용하므로 클래스가 바뀐다고 해도 메소드 이름이 변경될 걱정은 없다. 
		...
	}
	
	public User get(String id) throws ClassNofFoundException, SQLException {
		Connection c = connectionMaker.makeConnection();
		...
	}
}
```

<B>[문제점]</B> UserDao에 DConnection 클래스의 생성자를 호출해서 오브젝트를 생성하는 코드가 여전히 남아있다. 여전히 UserDao 소스코드를 함께 제공해서, 필요할 때마다 UserDao 생성자 메소드를 직접 수정하라고 하지 않고는 고객에게 자유로운 DB 커넥션 확장 기능을 가진 UserDao를 제공할 수가 없다. 



### 1.3.3 관계설정 책임의 분리

1.3.2에는 여전히 UserDao에 어떤 ConnectionMaker 구현 클래스를 사용할 지 결정하는 코드가 남아 있다. 즉, UserDao 안에 분리되지 않은, 또 다른 관심사항(UserDao와 UserDao가 사용할 ConnectionMaker의 특정 구현 클래스 사이의 관계를 설정해주는 것에 대한 관심)이 존재하고 있다. 따라서 UserDao 변경 없이는 DB 커넥션 기능의 확장이 자유롭지 못하다. 



- 오브젝트 사이의 관계가 만들어지려면?
  - 직접 생성자를 호출해서 오브젝트를 만드는 방법
  - 외부에서 만들어준 것을 가져오는 방법
    - UserDao 오브젝트가 다른 오브젝트와 관계를 맺으려면 관계를 맺을 오브젝트가 있어햐 하는데 이것을 꼭 UserDao 코드 내에서 만들 필요는 없다. 
    - 메소드 파라미터나 생성자 파라미터를 이용해서 관계를 맺을 오브젝트를 받아온다. 
      - 이 때 파라미터 타입을 전달받을 오브젝트의 인터페이스로 선언해뒀다고 가정하면, 이 경우 파라미터로 전달되는 오브젝트 클래스는 해당 인터페이스를 구현했어야 한다. 



<I>UserDao의 모든 코드는 ConnectionMaker 인터페이스 외에 어떤 클래스와도 관계를 가지지 않도록 만들어보자!</I>

- 결국 UserDao 오브젝트는 ConnectionMaker 인터페이스를 구현한 특정 클래스의 오브젝트와 관계를 맺어야 한다. 하지만 이것은 클래스 사이에 관계가 만들어지는 것은 아니고 단지 <B>오브젝트 사이에 다이내믹한 관계가 만들어지는 것</B>이다. (클래스 사이의 관계는 코드에 다른 클래스 이름이 나타나기 때문에 만들어지는 것임. 클래스 사이의 관계와 오브젝트 사이의 관계를 구분할 줄 알아야 함)
- 오브젝트 사이의 관계: 코드에서 특정 클래스를 알지 못하더라도 해당 클래스가 구현한 인터페이스를 사용했다면, 그 클래스의 오브젝트를 인터페이스 타입으로 받아서 사용할 수 있다. 다형성이라는 객체지향 프로그래밍의 특징을 통해 가능



<I>위 내용을 참고해서 UserDao 오브젝트가 DconnectionMaker 오브젝트를 사용하게 해보자!</I>

- 오브젝트 사이에 런타임 사용관계(또는 링크/또는 의존관계)라고 불리는 관계를 맺어주면 된다. 
- <B>런 타임에 둘 사이의 관계를 만들어주는 것을 외부에서 제 3의 오브젝트가 하도록</B>



```
public UserDao(ConnectionMaker connectionMaker) {
	this.conntectionMaker = connectionMaker; 
	//생성자에서 구체적인 클래스 생성코드 사라짐
	//UserDao와 특정 ConnectionMaker 구현 클래스의 오브젝트 간 관계를 맺는 책임을 담당하는 코드를 UserDao의 클라이언트 즉, 제3의 오브젝트에게 넘겨버림
}
```

```
public class UserDaoTest {
//이 클래스가 UserDao와 ConnectionMaker 구현클래스와의 런타임 오브젝트 의존관계를 설정하는 책임을 담당!!
//N사와 D사는 각각 자신들만의 DB 접속 클래스를 만들어서 UserDao를 사용할 수 있게 됨. UserDao 변경 없이
	public static void main(String[] args) throws ClassNotFoundException, SQLException {
		ConnectionMaker connectionMaker = new DConnectionMaker();
		UserDao dao = new UserDao(connectionMaker); 
		...
	}
}
```



<B>[정리]</B> 인터페이스를 도입하고 클라이언트의 도움을 얻는 방법은 상속을 사용해 비슷한 시도를 했을 경우에 비해 훨씬 유연하다. DAO가 아무리 많아져도 DB 접속 방법에 대한 관심은 오직 한 군데에 집중될 수 있고, DB 접속 방법을 변경해야 할 때도 오직 한 곳의 코드만 수정하면 된다. 



### 1.3.4 원칙과 패턴

#### 개방 폐쇄 원칙 (OCP, Open-Closed Principle)

- 클래스나 모듈은 확장에는 열려있어야 하고 변경에는 닫혀있어야 한다.
- Ex. UserDao
  - UserDao에 영향을 주지 않고 DB 연결 방법이라는 기능을 확장하는 데 열려 있다.
  - UserDao 자신의 핵심 기능을 구현한 코드는 그런 변화에 영향을 받지 않고 유지할 수 있으므로 변경에는 닫혀 있다. 
  - 인터페이스
    - 인터페이스를 통해 제공되는 확장 포인트는 확장을 위해 활짱 개방됨
    - 인터페이스를 이용하는 클래스는 자신의 변화가 불필요하게 일어나지 않도록 굳게 폐쇄됨



#### 높은 응집도와 낮은 결합도 (High conherence and low coupling)

- 높은 응집도
  - 하나의 모듈, 클래스가 하나의 책임 또는 관심사에만 집중됨
  - 결과적으로 변화가 일어날 때 해당 모듈에서 변하는 부분이 큼
- 낮은 결합도 
  - 결합도: 하나의 오브젝트가 변경이 일어날 때 관계를 맺고 있는 다른 오브젝트에게 변화를 요구하는 정도
  - 책임과 관심사가 다른 오브젝트 또는 모듈과는 느슨하게 연결된 형태를 유지하는 게 좋다.
  - 느슨한 연결 - 관계를 유지하는 데 꼭 필요한 최소한의 방법만 간접적인 형태로 제공, 나머지는 서로 독립적이고 알 필요 없게 만든다. 
- Ex. UserDao
  - UserDao와 ConnectionMaker의 관계는 인터페이스를 통해 매우 느슨하게 연결되어 있음
  - UserDao 입장에서는 구체적인 ConnectionMaker 구현 클래스의 구현 방법이나 전략, 뒤에서 사용하는 오브젝트에 대해서 알 필요 없음



#### 전략 패턴 (Strategy Pattern)

- 자신의 기능 맥락에서, 필요에 따라 변경이 필요한 알고리즘을 통째로 외부에 분리시키고, 이를 구현한 구체적인 알고리즘 클래스를 필요에 따라 바꿔서 사용할 수 있게 하는 디자인 패턴



스프링의 역할 - 위의 객체지향적인 설계 원칙과 디자인 패턴에 나타난 장점을 자연스럽게 개발자들이 활용할 수 있게 해주는 프레임워크
