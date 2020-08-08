

# 1장 오브젝트와 의존관계 



## 1.1 초난감 DAO

> <B>DAO</B> (Data Access Object)
>
> DB를 사용해서 데이터를 조회하거나 조작하는 기능을 전담하는 오브젝트

- <I>난감한 코드를 작성하고, 이 코드를 객체지향 기술 원리에 충실한 스프링 스타일의 코드로 개선하는 작업을 할 예정.</I>



## 1.2 DAO의 분리

개발자가 객체를 설계할 때 가장 염두해 둬야 할 사항은 "<B>미래의 변화를 어떻게 대비할 것인가</B>"이다. 

- 객체지향 설계와 프로그래밍이 절차지향에 비해 초기에 좀 더 번거로운 작업을 요구하는 이유는, 객체지향 기술 자체가 지니는 변화에 효과적으로 대처할 수 있다는 기술적인 특징 때문이다. 
- 객체 지향 기술은 흔히 실세계를 최대한 가깝게 모델링해낼 수 있기 떄문에 의미가 있다고 여겨진다. 하지만 그보다는 객체지향 기술이 만들어내는 가상의 추상세계 자체를 효과적으로 구성할 수 있고, 이를 자유롭고 편리하게 변경, 발전, 확장시킬 수 있다는 데 더 의미가 있다. 



<I>어떻게 변경이 일어났을 때 필요한 작업을 최소화하고, 그 변경이 다른 곳에 문제를 일으키지 않게 할 수 있을까?</I>

<h4>"분리와 확장을 고려해서 설계해야 한다"</h4>





### 1.2.1 관심사의 분리 

프로그래밍의 기초 개념. 객체 지향에 적용해보면, <B>관심이 같은 것끼리는 하나의 객체 안으로 또는 친한 객체로 모이게</B> 하고, 관심이 다른 것은 가능한 한 따로 떨어져서 서로 영향을 주지 않도록 분리하는 것



### 1.2.2 커넥션 만들기의 추출

#### 중복 코드의 메소드 추출 : 독립된 메소드를 만들어서 분리

하나의 클래스 안에 존재하는 중복코드를 하나의 메소드로 추출한다. 

- 추출 전: UserDao 클래스의 ```add()```, ```get()``` 메소드가 있고 메소드마다 "DB 커넥션 연결"이라는 관심사가 존재
- 추출 후: 각 메소드에 존재했던 "DB 커넥션 연결"이라는 관심을 분리해서 공통 메소드 ```getConnection()``` 에서 처리



### 1.2.3 DB 커넥션 만들기의 독립

<I>UserDao 소스코드를 N사와 D사에 공개하지 않고 고객 별로 원하는 DB 커넥션 생성 방식을 적용해서 UserDao를 사용하게 할 수 있을까? </I>

#### 상속을 통한 확장 : 상속을 통해 분리

```getConnection()``` 안의 구현 코드를 제거하고 UserDao의 추상 메소드로 만들어둔다.

- N사와 D사는 UserDao 클래스를 상속해서 각각 서브클래스를 만들고 그 안에서 ```getConnection()``` 메소드를 원하는대로 구현해서 사용하면 된다. 커넥션 연결 기능을 자유롭게 확장할 수 있게 함

- 1.2.2에서 "커넥션 연결"이라는 관심을 같은 클래스에 있는 다른 메소드로 분리했지만 이번에는 그 기능을 상속을 통해 아예 다른 서브 클래스로 분리해버림
  - UserDao : 어떻게 데이터를 가져올 것인가? (SQL 작성, 파라미터 바인딩, 쿼리 실행, 검색정보 전달)
  - NUserDao/DUserDao : DB 연결은 어떻게 할 것인가? 



<B>[정리]</B> 1.2.2와 1.2.3의 "관심사 분리" 작업을 거쳐서 UserDao는 단순히 변경이 용이하다 라는 수준을 넘어서 손쉽게 확장된다라고 말할 수 있게되었다. 새로운 DB 연결 방법을 적용해야 할 때는 UserDao 상속을 통해 확장해주면 된다. 



> <h4>템플릿 메소드 패턴</h4>
>
> 슈퍼클래스에 기본적인 로직의 흐름을 만들고, 그 기능의 일부를 추상 메소드나 오버라이딩이 가능한 protected 메소드 등으로 만든 뒤 서브클래스에서 이런 메소드를 필요에 맞게 구현해서 사용하도록 하는 디자인 패턴. 스프링에서 애용되는 패턴
>
> > 상속을 통해 슈퍼클래스의 기능을 확장할 때 사용하는 방법
> >
> > 변하지 않는 기능은 슈퍼클래스에 만들어두고 자주 변경되며 확장할 기능은 서브클래스에서 만들도록 한다. 



> <h4>팩토리 메소드 패턴</h4>
>
> 서브클래스에서 구체적인 <B>오브젝트 생성방법을 결정</B>하게 하는 디자인 패턴
>
> > ```getConnection()``` 메소드는 Connection 타입 오브젝트를 생성한다는 기능을 정의해놓은 추상 메소드이고 UserDao 서브 클래스의 ```getConnection()``` 메소드는 어떤 Connection 클래스의 오브젝트를 어떻게 생성할 것인 지를 결정하는 방법이라고 볼 수 있다. 
> >
> > 템플릿 메소드 패턴과 마찬가지로 상속을 통해 기능을 확장하게 하는 방법



#### 단점

- 상속 자체가 가지고 있는 한계점
  - 자바는 클래스의 다중상속을 허용하지 않는다. 
    - NUserDao 클래스가 이미 다른 클래스를 상속 중이라면 UserDao를 상속할 수 없음 
  - 상속을 통한 상하위 클래스의 관계는 생각보다 밀접하다. 
    - 슈퍼클래스의 변경이 있을 때 모든 서브 클래스를 함께 수정하거나 다시 개발해야할 수도 있다. 
  - 확장된 기능인 DB 커넥션을 생성하는 코드를 다른 DAO 클래스에 적용할 수 없다. 
    - UserDao 외의 DAO 클래스들이 계속 만들어지면 상속을 통해 만들어진 getConnection() 구현 코드가 매 DAO 클래스마다 중복된다. 



## 1.3 DAO의 확장

<I>상속 말고 다른 방법으로 확장해보자!</I>



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
```

```
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

> <h4>추상화</h4>
>
> 어떤 것들의 공통적인 성격을 뽑아내어 이를 따로 분리해내는 작업
>
> 이러한 추상화를 위해 자바에서 제공하는 도구 - 인터페이스



> <h4>인터페이스</h4>
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


## 1.4 제어의 역전

### 1.4.1 오브젝트 팩토리

앞의 리팩토링 과정에서는 UserDao에 여러 개의 관심이 집중되지 않도록 '어떤 DB Connection 구현체'를 사용할 것인 지에 대한 결정을 하는 역할을 UserDaoTest에게 분리시켰다. 하지만 UserDaoTest는 원래 UserDao의 기능이 잘 동작하는 지 테스트하기 위해 만들어진 것이었다. 기존의 책임 외에 또 다른 책임을 갖게 된 것이다. 다시 한번 분리하는 과정이 필요할 것 같다! 



분리된 후의 기능은 아래처럼 나뉘어져야할 것이다. 

- 기능1. UserDao 오브젝트를 만드는 것

- 기능2. ConnectionMaker 구현 클래스의 오브젝트를 만드는 것

- 기능3. 이렇게 만들어진 두 개의 오브젝트가 연결돼서 사용될 수 있도록 관계를 맺어주는 것



#### 팩토리

"객체의 생성 방법을 결정하고 그 오브젝트를 돌려주는 역할을 하는 오브젝트. 즉, 오브젝트를 생성하는 쪽과 사용하는 쪽이 분리되어 있다."

> <h5>오브젝트 vs 인스턴스</h5>
>
> 오브젝트(객체) : 클래스의 타입으로 선언한 것
>
> 인스턴스: 그 객체가 메모리에 할당(new)되어진 것



```
public class DaoFactory {
	//팩토리의 메소드는 
	public UserDao userDao() {
	
		//userDao 객체의 생성방법을 결정하고
		ConnectionMaker connectionMaker = new DConnectionMaker();
		UserDao userDao = new UserDao(connectionMaker);
		
		//그렇게 만들어진 userDao 객체를 돌려준다. 
		return userDao; 
		
	}
}
```

```
public class UserDaoTest {
	public static void main(String[] args) throws ClassNotFoundException, SQLException {
	
		//UserDaoTest는 UserDao가 어떻게 만들어지는 지 어떻게 초기화되어 있는 지에 신경쓰지 않고 팩토리로부터 UserDao 객체를 받아서, 자신의 관심사인 테스트만 수행하면 된다. 
		UserDao userDao = new DaoFactory().userDao();
		...
		
	}
}
```



#### 설계도로서의 팩토리

위 결과를 정리하면

UserDao - 핵심적인 데이터 로직 담당 책임

ConnectionMaker - 핵심적인 기술 로직 담당 책임

DaoFactory - 이런 애플리케이션의 오브젝트들을 구성하고 그 관계를 정의하는 책임 

위 두개는 실질적인 로직은 담당하는 컴포넌트, 팩토리는 컴포넌트 구조와 관계를 정의한 설계도 같은 역할!

"팩토리는 어떤 오브젝트가 어떤 오브젝트를 사용하는 지 정의해놓은 코드라고 생각하자."



### 1.4.2 오브젝트 팩토리의 활용

DaoFactory에 userDao()와 같은 DAO 생성메소드가 늘어나면, 중복되는 부분이 나타난다. 이것도 분리해주자. 

```
public class DaoFactory {
	public UserDao userDao() {
		return new UserDao(connectionMaker());
	}
	
	public AccountDao accountDao() {
		return new AccountDao(connectionMaker());
	}
	
	public MessageDao messageDao() {
		return new MessageDao(connectionMaker());
	}
	
	public ConnectionMaker connectionMaker() {
		return new DConnectionMaker(); //분리함 
	}
}
```



### 1.4.3 제어권의 이전을 통한 제어관계 역전

"제어의 역전이란, 프로그램의 제어 흐름 구조가 뒤바뀌는 것이다."

제어의 역전에서는 오브젝트가 자신이 사용할 오브젝트를 스스로 선택하지 않고 생성하지도 않는다. 또 자신도 어떻게 만들어지고 어디서 사용되는 지를 알 수 없다. 모든 제어 권한을 자신이 아닌 다른 대상에게 위임하기 때문이다. 프로그램의 시작을 담당하는 main()과 같은 엔트리 포인트를 제외하면 모든 오브젝트는 이렇게 위임받은 제어 권한을 갖는 특별한 오브젝트에 의해 결정되고 만들어진다. 



<B>프레임워크는 제어의 역전 개념이 적용된 대표적인 기술이다.</B> 

- 애플리케이션 코드가 프레임워크게 의해 사용된다. 

  > 라이브러리의 경우에는 라이브러리를 사용하는 애플리케이션 코드가 흐름을 직접 제어한다. 단지 동작하는 중에 필요한 기능이 있을 때 능동적으로 라이브러리를 사용할 뿐이다. 

  프레임워크 위에 개발한 클래스를 등록해두고 프레임워크가 흐름을 주도하는 중에 개발자가 만든 애플리케이션 코드를 사용하도록 만드는 방식이다. 즉, 애플리케이션 코드는 프레임워크가 짜 놓은 틀에서 수동적으로 동작한다. 



<B>UserDao와 DaoFactory에도 제어의 역전이 적용되어 있다. </B>

- 앞에서 관심을 분리하고 책임을 나누고 확장가능한 구조로 만들기 위해 DaoFactory를 도입했던 과정이 제어의 역전을 적용하는 작업이었다. 

  원래 ConnectionMaker의 구현 클래스를 결정하고 오브젝트를 만드는 제어권은 UserDao에게 있었지만 지금은 DaoFactory에게 있다. 즉, UserDao는 자신도 팩토리에 의해 수동적으로 만들어지고 자신이 사용할 오브젝트도 DaoFactory가 공급해주는 것을 수동적으로 사용하게되었다. 

- 그러고보면, 스프링 프레임워크 없이도 IoC 개념을 적용한 셈이다. 단순한 적용이라면 이처럼 DaoFactory 같은 IoC 제어권을 가진 오브젝트를 분리해서 만드는 방법을 사용해도 충분하겠지만

- IoC를 애플리케이션 전반에 걸쳐 적용하고 싶다면 스프링과 같은 IoC 프레임워크의 도움을 받는 편이 훨씬 유리하다. 제어의 역전에서는 애플리케이션 컴포넌트의 생성과 관계설정, 사용, 생명주기 관리 등을 관장하는 존재가 필요하다. 



## 1.5 스프링의 IoC

"스프링의 핵심을 담당하는 건, 빈 팩토리 또는 애플리케이션 컨텍스트라고 불리는 것이다. 이 두가지는 DaoFactory가 하는 일을 좀 더 일반화한 것이라고 할 수 있다."

### 1.5.1 오브젝트 팩토리를 이용한 스프링 IoC

#### 애플리케이션 컨텍스트와 설정정보

- 스프링에서는 스프링이 제어권을 가지고 직접 만들고 관계를 부여하는 오브젝트를 빈(Bean)이라고 부른다. 동시에 스프링 빈은 스프링 컨테이너가 생성과 관계설정, 사용 등을 제어해주는 제어의 역전이 적용된 오브젝트를 가리키는 말이다. 
- 스프링에서는 빈의 생성과 관계설정 같은 제어를 담당하는 IoC 오브젝트를 빈 팩토리(Bean factory)라고 부른다. 보통 빈 팩토리보다는 이를 좀 더 확장한 애플리케이션 컨텍스트(Application Context)를 주로 사용한다. 이것은 IoC 방식을 따라 만들어진 일종의 빈 팩토리라고 생각하면 된다. 
  - 애플리케이션 컨텍스트는 별도의 설정 정보를 참고해서 빈의 생성, 관계설정 등의 제어 작업을 총괄한다. 
  - 설정 정보를 만드는 방법은 다양하게 존재한다. 



#### DaoFactory를 사용하는 애플리케이션 컨텍스트

DaoFactory를 스프링의 빈 팩토리가 사용할 수 있는 본격적인 설정정보로 만들어보자!

```
@Configuration //이 클래스가 빈 팩토리를 위한 오브젝트 설정을 담당하는 클래스라고 스프링이 인식할 수 있도록 추가
public class DaoFactory {

	@Bean //오브젝트를 만들어주는 메소드에 추가 
	public UserDao userDao() {
		return new UserDao(connectionMaker());
	}
	
	@Bean
	public ConnectionMaker connectionMaker() {
		return new DConnectionMaker(); 
	}

}
```

- @Configuration과 @Bean 어노테이션을 추가해줌으로써 스프링 프레임워크의 빈 팩토리 또는 애플리케이션 컨텍스트가 IoC 방식의 기능을 제공할 때 사용할 설정정보가 완성되었다. 



이번에는 DaoFacotry를 설정정보로 사용하는 애플리케이션 컨텍스트를 만들어보자!

```
public class UserDaoTest {
	public stagic void main(String[] args) throws ClassNotFoundException, SQLException {
	
		//애플리케이션 컨텍스트는 ApplicationContext 타입의 오브젝트이다. 
		//ApplicationContext를 구현한 클래스는 여러개 존재하는데 @Configuration이 붙은 자바 코드를 설정정보로
		//사용하려면 AnnotationConfigApplicationContext를 사용하면 된다. 
		//애플리케이션 컨텍스르를 만들 때 생성자 파라미터로 DaoFactory 클래스를 넣어준다. 
		ApplicationContext context = new AnnotationConfigApplicationContext(DaoFactory.class);
		
		//준비된 ApplicationContext의 getBean()이라는 메소드를 호출해서 UserDao의 오브젝트를 가져온다. 
		//getBean(): AppliationContext가 관리하는 오브젝트를 요청하는 메소드
		//첫 인자는 등록된 빈의 이름이다. 메소드 이름이 빈 이름이다. 
		//두번쨰 인자는 리턴 타입이다. getBean()은 기본적으로 Object 타입으로 리턴하게 되어 있어서 캐스팅 부담을 줄이
		//기 위해 타입을 지정해준다. 
		UserDao dao = context.getBean("userDao", UserDao.class);
	
	}
}
```



### 1.5.2 애플리케이션 컨텍스트의 동작방식

> 스프링에서는 애플리케이션 컨텍스트를 IoC 컨테이너라 하기도 하고, 간단히 스프링 컨테이너라고 부르기도 한다. 또는 빈 팩토리라고 부를 수도 있다. 애플리케이션 컨텍스트를 DaoFactory와 달리 직접 오브젝트를 생성하고 관계를 맺어주는 코드가 없고 그런 생성정보와 연관관계 정보를 별도의 설정정보를 통해 얻는다. 



<B>동작방식</B>

- 애플리케이션 컨텍스트는 @Configuration 어노테이션이 붙은 DaoFactory 클래스를 설정정보로 등록해두고 
- @Bean이 붙은 메소드의 이름을 가져와 빈 목록을 만들어둔다. 
- 클라이언트가 애플리케이션 컨텍스트의 getBean() 메소드를 호출하면 자신의 빈 목록에서 요청한 이름이 있는 지 찾고
- 있다면 빈을 생성하는 메소드를 호출해서 오브젝트를 생성시킨 후
- 클라이언트에게 돌려준다. 



<B>DaoFactory 오브젝트 팩토리로 사용했을 때에 비해 애플리케이션 컨텍트스를 사용했을 때 좋은점</B>

- 클라이언트는 구체적인 팩토리 클래스를 알 필요가 없다. 
- 애플리케이션 컨텍스트는 종합 IoC 서비스를 제공해준다. 
- 애플리케이션 컨텍스트는 빈을 검색하는 다양한 방법을 제공한다. 



### 1.5.3 스프링 IoC 용어 정리

#### 빈

스프링이 IoC 방식으로 관리하는 오브젝트. 스프링이 직접 그 생성과 제어를 담당하는 오브젝트만을 빈이라고 부른다.



#### 빈 팩토리

스프링의 IoC를 담당하는 핵심 컨테이너. 빈을 등록하고 생성하고 조회하고 돌려주고 그 외에 부가적인 빈을 관리하는 기능을 담당한다. 



#### 애플리케이션 컨텍스트

빈 팩토리를 확장한 IoC 컨테이너. 빈을 등록하고 관리하는 기본적인 기능은 빈 팩토리와 동일하다. 여기에 스프링이 제공하는 각종 부가 서비스를 추가로 제공한다. 

> 빈 팩토리는 주로 빈의 생성과 제어의 관점에서 이야기하는 것이고, 애플리케이션 컨텍스트는 스프링이 제공하는 애플리케이션 지원 기능을 모두 포함해서 이야기하는 것이라고 보면 된다. ApplicationContext는 BeanFactory를 상속한다. 



#### 설정정보/설정 메타정보

스프링의 설정정보란 애플리케이션 컨텍스트 또는 빈 팩토리가 IoC를 적용하기 위해 사용하는 메타정보. 컨테이너에 어떤 기능을 세팅하거나 조정하는 경우에도 사용하지만, 그보다는 IoC 컨테이너에 의해 관리되는 애플리케이션 오브젝트를 생성하고 구성할 때 사용된다. 



#### 컨테이너 또는 IoC 컨테이너

IoC 방식으로 빈을 관리한다는 의미에서 애플리케이션 컨텍스트나 빈 팩토리를 컨테이너 또는 IoC 컨테이너라고 부른다. 

> IoC 컨테이너는 주로 빈 팩토리의 관점에서 이야기하는 것이고, 그냥 컨테이너 또는 스프링 컨테이너라고 할 떄는 애플리케이션 컨텍스트를 가리키는 것이라고 보면 된다. 

> 애플리케이션 컨텍스트는 하나의 애플리케이션에서 보통 여러 개가 만들어져 사용되는데, 이를 통틀어서 스프링 컨테이너라고 부를 수 있다. 



#### 스프링 프레임워크

IoC 컨테이너, 애플리케이션 컨텍스트를 포함해서 스프링이 제공하는 모든 기능을 통틀어 말할 떄 주로 사용한다. 그냥 스프링이라고 줄여서 말하기도 한다. 



## 1.6 싱글톤 레지스트리와 오브젝트 스코프

DaoFactory를 직접 사용하는 것과 @Configuration 어노테이션을 추가해서 스프링의 애플리케이션 컨텍스트를 통해 사용하는 것은 테스트 결과는 동일하다. 뭐가 다른걸까? 



<B>DaoFactory의 userDao()를 여러 번 호출했을 떄 동일한 오브젝트가 돌아오는가?</B>

- 매번 다른 오브젝트가 만들어져서 돌아온다. 

<B>스프링의 애플리케이션 컨텍스트에 DaoFactory를 설정정보로 등록하고 getBean()을 통해 오브젝트를 가져오면?</B>

- 동일한 오브젝트를 가져온다. 



"스프링은 여러 번에 걸쳐 빈을 요청하더라도 매번 동일한 오브젝트를 돌려준다!"



### 1.6.1 싱글톤 레지스트리로서의 어플리케이션 컨텍스트

애플리케이션 컨텍스트는

- 앞에서 만들었던 오브젝트 팩토리와 비슷한 방식으로 돌아가는 IoC 컨테이너이고
- 싱글톤을 저장하고 관리하는 싱글톤 레지스트리이기도 하다. 스프링은 기본적으로 별다른 설정을 하지 않으면 내부에서 생성하는 빈 오브젝트를 모두 싱글톤으로 만든다. 



#### 서버 애플리케이션과 싱글톤

<B>왜 스프링은 싱글톤으로 빈을 만드는 것일까?</B>

- 스프링이 주로 적용되는 대상이 '자바 엔터프라이즈 기술을 사용하는 서버환경'이기 때문이다. 

  > 대규모 엔터프라이즈 서버환경은 서버 하나 당 최대로 초당 수십에서 수백 번씩 여러 요청을 받아 처리할 수 있는 높은 성능이 요구된다. 

- 매번 클라이언트에서 요청이 올 때마다 각 로직을 담당하는 오브젝트를 새로 만들어서 사용한다면..?

- 아무리 자바의 오브젝트 생성과 가비지컬렉션의 성능이 좋아졌다고 해도 이렇게 부하가 걸리면 서버가 감당하기 힘들다!

- 그래서 앤터프라이즈 분야에서는 "서비스 오브젝트"라는 개념을 일찍부터 사용했다. 그 대표적인 것이 "서블릿"이다.

- 서블릿은 대부분 멀티스레드 환경에서 싱글톤으로 동작한다. 서블릿 클래스 당 하나의 오브젝트만 만들어두고, 사용자의 요청을 담당하는 여러 스레드에서 하나의 오브젝트를 공유해 동시에 사용한다. 



#### 싱글톤 패턴의 한계

<B>싱글톤을 구현하는 일반적인 방법부터 보면</B>

```
public class UserDao {
	//생성된 싱글톤 오브젝트를 저장할 수 있는 자신과 같은 타입의 스태틱 필드를 정의한다. 
	private static UserDao INSTANCE;
	...
	
	//클래스 밖에서는 오브젝트를 생성하지 못하도록 생성자를 private로 만든다. 
	private UserDao(ConnectionMaker connectionMaker) {
		this.connectionMaker = connectionMaker;
	}
	
	//스태틱 팩토리 메소드를 만들고 이 메소드가 최초로 호출되는 시점에서 한 번만 오브젝트가 만들어지게 한다. 
	//생성된 오브젝트는 스태틱 필드에 저장된다. 또는 스태틱 필드의 초기값으로 오브젝트를 미리 만들어둘 수도 있다. 
	//한번 오브젝트가 만들어지고 난 후에는 getInstance() 메소드를 통해 이미 만들어져 스태틱 필드에 저장해둔 오브젝트를 넘겨준다. 
	public static synchronized UserDao getInstance() {
		if (INSTANCE == null) INSTANCE = new UserDao(???);
		return INSTANCE;
	}
}
```



<B>문제점이 있다</B>

- private 생성자를 갖고 있기 때문에 상속할 수 없다.
- 싱글톤은 테스트하기가 힘들다.
- 서버환경에서는 싱글톤이 하나만 만들어지는 것을 보장하지 못한다.
- 싱글톤의 사용은 전역 상태를 만들 수 있기 때문에 바람직하지 못하다. 

이렇게 자바의 기본적인 싱글톤 패턴의 구현 방식은 여러가지 단점이 있다. 



#### 싱글톤 레지스트리

"스프링은 직접 싱글톤 형태의 오브젝트를 만들고 관리하는 기능을 제공하는데, 이것이 싱글톤 레지스트리이다. "

> 스프링은 
>
> - IoC 컨테이너이면서 
> - 싱글톤을 만들고 관리해주는 싱글톤 레지스트리이다. 



<B>장점</B>

스태틱 메소드와 private 생성자를 사용해야 하는 비정상적인 클래스가 아니라 평범한 자바 클래스를 싱글톤으로 활용하게 해준다. 평범한 자바 클래스라도 IoC 방식의 컨테이너를 사용해서 생성과 관계설정, 사용 등에 대한 제어권을 컨테이너에게 넘기면 손쉽게 싱글톤 방식으로 만들어져 관리되게 할 수 있다. 



### 1.6.2 싱글톤과 오브젝트의 상태

싱글톤은 멀티스레드 환경이라면 여러 스레드가 동시에 접근해서 사용할 수 있다. 따라서 싱글톤은 상태정보를 내부에 갖고 있지 않은 무상태(Stateless) 방식으로 만들어져야 한다. 



<B>상태가 없는 방식으로 클래스를 만들 때 각 요청에 대한 정보나, DB나 서버의 리소스로부터 생성한 정보는 어떻게 다뤄야 할까?</B>

- 파라미터와 로컬 변수, 리턴 값 등을 이용하면 된다. 메소드 파라미터나, 메소드 안에서 생성되는 로컬 변수는 매번 새로운 값을 저장할 독립적인 공간이 만들어지기 때문에 싱글톤이라 해도 여러 스레드가 변수의 값을 덮어쓸 일은 없다. 



인스턴스 변수를 사용하면..

```
public class UserDao {
	//초기에 설정하면 사용 중에는 바뀌지 않는 읽기전용 인스턴스 변수 
	//얘는 읽기전용 인터페이스이므로 문제발생 안 함. 이 변수에는 ConnectionMaker 타입의 싱글톤 오브젝트가 들어있음
	private ConnectionMaker connectionMaker; 
	
	//매번 새로운 값으로 바뀌는 정보를 담는 인스턴스 변수
	//**싱글톤으로 만들어져서 멀티스레드 환경에서 사용하면 문제가 발생한다!!
	private Connection c; 
	private User user; 
	
	public User get(String id) throws ClassNotFoundException, SQLException {
		this.c = connectionMaker.makeConnection();
		...
		this.user = new User();
		this.user.setId(rs.getString("id"));
		this.user.setName(rs.getString("name"));
		this.user.setPassword(rs.getString("password"));
		...
		return this.user;
	}
}
```



### 1.6.3 스프링 빈의 스코프

스코프란 스프링이 관리하는 오브젝트, 즉 빈이 생성되고 존재하고 적용되는 범위를 말한다. 

스프링 빈의 기본 스코프는 "싱글톤"이다. 싱글톤 스코프는 컨테이너 내에 한 개의 오브젝트만 만들어져서 강제로 제거하지 않는 한 스프링 컨테이너가 존재하는 동안 계속 유지된다. 

경우에 따라서는 싱글톤 외의 스코프를 가질 수 있다. (10장 참고)

- 프로토타입 스코프: 컨테이너에 빈을 요청할 때마다 매번 새로운 오브젝트를 만들어준다.
- 요청 스코프: 웹을 통해 새로운 HTTP 요청이 생길때마다 생성된다.
- 세션 스코프: 웹의 세션과 스코프가 유사하다. 

## 1.7 의존관계 주입

### 1.7.1 제어의 역전(IoC)과 의존관계 주입

스프링 IoC 기능의 대표적인 동작원리는 주로 의존관계 주입이라고 불린다. 이것이 스프링이 다른 프레임워크와 차별화돼서 제공해주는 기능이다. 요즘은 스프링을 DI 컨테이너라고 부르기도 한다. 



### 1.7.2 런타임 의존관계 설정

#### 의존관계

두 개의 클래스가 의존관계에 있다고 말할 때는 방향성을 부여해줘야 한다. 



A -------------> B 

- A가 B에 의존하고 있음을 나타낸다. 
- 의존한다는 건 의존대상, 여기서는 B가 변하면 그것이 A에 영향을 미친다는 뜻이다. 
- 대표적인 예는 A가 B를 사용하는 경우, 예를 들어서 A에서 B에 정의된 메소드를 호출해서 사용하는 경우이다. 



#### UserDao의 의존관계

UserDao가 ConnectionMaker 인터페이스에 의존하고 있는 형태에서, 

- ConnectionMaker 인터페이스가 변하면 UserDao도 직접적으로 영향을 받는다.
- 하지만 ConnectionMaker를 구현한 DConnectionMaker가 변했을 경우에는 UserDao가 영향을 받지 않는다.

"인터페이스에 대해서만 의존관계를 만들어두면 인터페이스 구현 클래스와의 관계는 느슨해지면서 변화에 영향을 덜 받는 상태가 된다. "



인터페이스를 통해 설계 시점에 느슨한 의존관계를 갖는 경우에는 UserDao의 오브젝트가 런타임 시에 사용할 오브젝트가 어떤 클래스로 만든 것인 지 미리 알 수가 없다. 이 때 프로그램이 실행되고 UserDao의 오브젝트가 만들어지고 나서 런타임 시에 의존관계를 맺는 대상, 즉 실제 사용대상인 오브젝트를 <B>의존 오브젝트</B>라고 한다. 



<B>의존관계 주입은 구체적인 의존 오브젝트(:DConnectionMaker)와 그것을 사용할 주체, 보통 클라이언트(UserDao)라고 부르는 오브젝트를 런타임 시에 연결해주는 작업을 말한다. </B>

의존관계 주입이란, 아래 조건을 충족하는 작업을 말한다. 

- 클래스 모델이나 코드에는 런타임 시점의 의존관계가 드러나지 않는다. 그러기 위해서는 인터페이스에만 의존하고 있어야 한다.
- 런타임 시점의 의존관계는 컨테이너나 팩토리 같은 제 3의 존재가 결정한다. 
- 의존관계는 사용할 오브젝트에 대한 레퍼런스를 외부에서 제공(주입)해줌으로써 만들어진다. 



#### UserDao의 의존관계 주입

DaoFactory를 만든 시점에서 의존관계주입을 이용한 셈이다. 

DaoFactory는 

- 두 오브젝트 사이의 런타임 의존관계를 설정해주는 의존관계 주입작업을 주도하는 존재이며
- 동시에 IoC 방식으로 오브젝트의 생성과 초기화, 제공 등의 작업을 수행하는 컨테이너이다. 
- 아무튼 DI 컨테이너다!
- DI 컨테이너는 UserDao를 만드는 시점에서 생성자의 파라미터로 이미 만들어진 DConnectionMaker의 오브젝트를 전달한다. 정확히는 DConnectionMaker 오브젝트의 레퍼런스가 전달되는 것이다. 



### 1.7.3 의존관계 검색과 주입

의존관계 검색(DL)이란, 의존관계를 맺으려 할 때 스스로 검색해서 자신이 필요로하는 의존 오브젝트를 능동적으로 찾는 방식을 말한다. 런타임 시 의존관계를 맺을 오브젝트를 결정하는 것과 오브젝트의 생성 작업은 외부 컨테이너에게 IoC로 맡기지만 이를 가져올 때는 메소드나 생성자를 통한 주입 대신 스스로 컨테이너에게 요청을 해서 가져온다. 

```
public UserDao() {
	UserFactory daoFactory = new DaoFactory();
	this.connectionMaker = daoFactory.connectionMaker();
}
```



스프링의 애플리케이선 컨텍스트의 경우라면 미리 정해놓은 이름을 전달해서 그 이름에 해당하는 오브젝트를 찾게 된다. 애플리케이션 컨텍스트는 getBean()이라는 의존관계 검색에 사용되는 메소드를 제공한다. 

```
public UserDao() {
	AnnotationConfigApplicationContext context = new AnnotationConfigApplication(DaoFactory.class);
	this.connectionMaker = context.getBean("connectionMaker", ConnectionMaker.class);
}
```



의존관계 검색보다는 의존관계 주입이 훨씬 단순하고 깔끔해서 주입을 사용하는 것이 낫지만, 의존관계 검색 방식을 사용해야할 경우가 있다. 애플리케이션 기동 시점에서는 적어도 한 번 의존관계 검색 방식을 사용해서 오브젝트를 가져와야 한다. 스태틱 메소드인 main()에서는 DI를 이용해서 오브젝트를 주입받을 방법이 없기 때문이다. 



<B>DI vs DL</B>

의존관계 검색 방식에서는 검색하는 오브젝트는 자신이 스프링의 빈일 필요가 없다. 

의존관계 주입 방식에서는 주입을 원하는 오브젝트는 자신이 먼저 스프링의 빈이 되어야 한다. 



### 1.7.4 의존관계 주입의 응용

코드에는 런타임 클래스에 대한 의존관계가 나타나지 않고, 인터페이스를 통해 결합도가 낮은 코드를 만들므로, 다른 책임을 가진 사용 의존관계에 있는 대상이 바뀌거나 변경되더라도 자신은 영향을 받지 않으며 벼경을 통한 다양한 확장 방법에는 자유롭다는 장점이 있다. 
#### 기능 구현의 교환
#### 부가기능 추가


### 1.7.5 메소드를 이용한 의존관계 주입

- 수정자 메소드를 이용한 주입
  - set으로 시작해야 하고 한 번에 한 개의 파라미터만 가질 수 있는 메소드.
  - 외부에서 오브젝트 내부의 애트리뷰트 값을 변경하려는 용도로 주로 사용된다. 
  - 부가적으로 입력 값에 대한 검증이나 그 밖의 작업을 수행할 수도 있다. 

- 일반 메소드를 이용한 주입
  - 여러 개의 파라미터를 가질 수 있는 메소드



<B>수정자 메소드를 가장 많이 사용한다.</B>

DaoFactory 같은 자바 코드 대신 XML을 사용하는 경우에는 자바빈 규약을 따르는 수정자 메소드가 가장 사용하기 편하다. 가능한 의미있고 단순한 이름을 사용하거나, 메소드를 통해 DI 받을 오브젝트의 타입 이름을 따르는 것이 가장 무난하다. 

```
public class UserDao {
	private ConnectionMaker connectionMaker; 
	
	public void setConnectionMaker(ConnectionMaker connectionMaker) {
		this.connectionMaker = connectionMaker;
	}
}
```

```
...

@Bean
public User userDao() {
	UserDao userDao = new UserDao();
	userDao.setConnectionMaker(connectionMaker());
	return userDao; 
}
```



## 1.8 XML을 이용한 설정

스프링은 DaoFactory와 같은 자바 클래스를 이용하는 것 외에도, 다양한 방법을 통해 DI 의존관계 설정정보를 만들 수 있다. 가장 대표적인 것이 XML이다. 



### 1.8.1 XML 설정

@Configuration을 <beans>, @Bean을 <bean>에 대응해서 생각하면 된다. 

> @Bean 메소드에서 얻을 수 있는 빈의 DI 정보
>
> - 빈의 이름: @Bean 메소드 이름이 빈의 이름이다. 이 이름은 getBean()에서 사용된다.
> - 빈의 클래스: 빈 오브젝트를 어떤 클래스를 이용해서 만들지를 정의한다.
> - 빈의 의존 오브젝트: 빈의 생성자나 수정자 메소드를 통해 의존 오브젝트를 넣어준다. 의존 오브젝트도 하나의 빈이므로 이름이 있을 것이고, 그 이름에 해당하는 메소드를 호출해서 의존 오브젝트를 가져온다. 의존 오브젝트는 하나 이상일 수도 있다. 



#### connectionMaker() 전환

```
@Bean -----------------------------------> <bean
public ConnectionMaker
connectionMaker() { --------------> id="connectionMaker"
	return new DConenctionMaker(); -> class="com.sh.dao.DConnectionMaker" />
}
```



#### userDao() 전환

```
<bean id="userDao" class="com.sh.dao.UserDao">
	<property name="connectionMaker" ref="connectionMaker" />
</bean>
```



#### XML의 의존관계 주입 정보

```
<beans>
	<bean id="connectionMaker" class="com.sh.dao.DConnectionMaker" />
	<bean id="userDao" class="com.sh.dao.UserDao">
		<property name="connectionMaker" ref="connectionMaker" />
		//name: DI에 사용할 수정자 메소드의 프로퍼티 이름
		//ref: 주입할 오브젝트를 정의한 빈의 ID
	</bean>
</beans>
```



### 1.8.2 XML을 이용하는 애플리케이션 컨텍스트

```
### applicationContext.xml

<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/shcma/beans/spring-beans-3.0.xsd">
	<bean id="connectionMaker" class="com.sh.dao.DConnectionMaker" />
	<bean id="userDao" class="com.sh.dao.UserDao">
		<property name="connectionMaker" ref="connectionMaker" />
		//name: DI에 사용할 수정자 메소드의 프로퍼티 이름
		//ref: 주입할 오브젝트를 정의한 빈의 ID
	</bean>
</beans>
```

```
ApplicationContext context = new GenericXmlApplicationContext("applicationContext.xml")
```



### 1.8.3 DataSource 인터페이스로 변환

#### DataSource 인터페이스 적용

자바에서는 DB 커넥션을 가져오는 오브젝트의 기능을 추상화해서 비슷한 용도로 사용할 수 있게 만들어진 DataSource라는 인터페이스가 이미 존재한다. 다양한 방법으로 DB 연결과 풀링 기능을 갖춘 많은 DataSource 구현 클래스가 존재하고, 이를 가져다 사용하면 된다. 

```
pakage javax.sql

public interface DataSource extends CommonDataSource, Wrapper {
	Connection getConnection() throws SQLException;
	...
}
```

```
import javax.sql.DataSource;

public class UserDao {
	private DataSource dataSource;
	
	public void setDataSource(DataSource dataSource) {
		this.dataSource = dataSource;
	}
	
	public void add(User user) throws SQLException {
		Connection c = dataSource.getConnection();
		...
	}
}
```



#### 자바 코드 설정 방식

DacFactory 설정 방식을 사용해서 기존의 connectionMaker() 메소드를 dataSource()로 변경하고 SimpleDriverDataSource를 리턴하게 하자!

```
@Bean
public DataSource dataSource {
	//DataSource 구현 클래스 중 테스트 환경에서 간단히 사용할 수 있는 클래스
	//DB 연결에 필요한 필수 정보만 입력받음
	SimpleDriverDataSource dataSource = new SimpleDriverDataSource();
	
	dataSource.setDriverClass(com.mysql.jdbc.Driver.class);
	dataSource.setUrl("jdbc:mysql://localhost/sh");
	dataSource.setUsername("sh");
	dataSource.setPassword("12345");
	
	return dataSource;
}
```

```
@Bean
public UserDao userDao() {
	UserDao userDao = new UserDao();
	userDao.setDataSource(dataSource());
	return userDao;
}
```

SimpleDriverDataSource의 오브젝트를 DI로 주입해서 사용할 수 있는 준비 완료한 것



#### XML 설정 방식

```
<bean id="dataSource" class="org.springframework.jdbc.datasource.SimpleDriverDataSource" />
```



### 1.8.4 프로퍼티 값 주입

```
### 코드를 통한 DB 연결정보 주입

dataSource.setDriverClass(com.mysql.jdbc.Driver.class);
dataSource.setUrl("jdbc:mysql://localhost/sh");
dataSource.setUsername("sh");
dataSource.setPassword("12345");
```

```
### XML을 이용한 DB 연결정보 설정

<property name="driverClass" value="com.mysql.jdbc.Driver" />
<property name="url" value="jdbc:mysql://localhost/sh" />
<property name="username" value="sh" />
<property name="password" value="12345" />
```



#### value 값의 자동 변환

어떻게 "com.mysql.jdbc.Driver"라는 스트링 값이 Class 타입의 파라미터를 갖는 수정자 메소드에서 사용될 수 있는 것일까?

- 스프링이 프로퍼티의 값을, 수정자 메소드의 파라미터 타입을 참고로 해서 적절한 형태로 변환해주기 때문이다. setDriverClass() 메소드의 파라미터 타입이 Class 임을 확인하고 "com.mysql.jdbc.Driver" 라는 텍스트 값을 com.mysql.jdbc.Drvier.class 오브젝트로 자동 변경해주는 것이다. 

- 내부적으로 일어나는 작업은 아래와 같다.

  ```
  Class driverClass = Class.forName("com.mysql.jdbc.Driver");
  dataSource.setDriverClass(driverClass);
  ```

- 즉, 스프링은 value에 지정한 텍스트 값을 적절한 자바 타입으로 변환해준다. (기본타입, 오브젝트 타입, 배열타입 전부 가능)



최종적인 applicationContext!

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/shcma/beans/spring-beans-3.0.xsd">

	<bean id="dataSource" class="org.springframework.jdbc.datasource.SimpleDriverDataSource">
  	<property name="driverClass" value="com.mysql.jdbc.Driver" />
		<property name="url" value="jdbc:mysql://localhost/sh" />
		<property name="username" value="sh" />
		<property name="password" value="12345" />
	</bean>
  
	<bean id="userDao" class="com.sh.dao.UserDao">
		<property name="dataSource" ref="dataSource" />
	</bean>
</beans>
```

