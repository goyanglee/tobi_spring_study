## 5장 서비스 추상화 
### Guide
1. 비즈니스 로직과 데이터 엑세스 코드는 분리한다.
2. Dao 기술 변화에 서비스 계층의 코드가 영향받지 않도록 di를 활용한다.
3. 단위 작업을 보장해주는 트랜잭션에 대해 이해한다.
4.  스프링이 트랜잭션 동기화기법을 제공한다.
5. 자바에서 사용하는 트랜잭션 API 종류와 방법에 대해 안다.
6. 스프링이 제공하는 **트랜잭션 서비스 추상화를** 이해한다.
7. **테스트 대역**이란 개념에 대해 이해한다.
8. **목 오브젝트**에 대해 이해한다.

스프링은 어떻게 성격이 비슷한 여러 종류의 기술을 추상화하고 일관된 방법으로 사용할 수 있도록 지원할까? 
### 사용자 레벨 관리 기능 
> 레벨 : 여기서는 멤버라벨, 등급 같은 걸 의미.
#### 레벨 enum 필드 추가 
레벨을 문자열로 넣는 것은 좋지 않다. DB 용량을 많이 차지하기 때문이다.
```java
class User {
private static final int BASIC=1;
private static final int SILVER=2;
private static final int GOLD=3;
int level;
public void setLevel(int level){
this.level=level;
}
}
```
그리고나서 level을 체크해주면 된다.
```java
if(user1.getLevel() == User.BASIC) {
user1.setLevel(User.SILVER);
}
```
Level은 int이기 때문에 다른 정보를 넣는다고 해서 컴파일러가 체크해주지는 못한다.(string으로 해도 마찬가지 아닌가?) 잘못된 값, 범위를 벗어나는 값 등의 오류를 범해도 코드상에는 전혀 문제가 없기때문에 문제다. 그래서 숫자타입을 직접 사용하는 것 보다 enum을 사용하는 것이 편하다.
```java
public enum Level{
BASIC(1), SILVER(2), GOLD(3);
private final int value;
public static Level valueOf(int value) {
	switch(value){
	case 1:return BASIC;
	case 2:return SILVER;
	case 3:return GOLD;
	default: throw new AssertionError(“”);
	}
}

public int intValue() {
	return value;
	}

Level(int value) {
	this.value=value;
}
}
```
이렇게 하면 겉으로는 Level 타입의 오브젝트이기 때문에 안전하다.
#### user 필드 추가 
#### userDaoTest 수정 
추가되는 필드들을 검증하는 메소드를 따로 빼놓고 재사용한다.

### 사용자 수정 기능 추가
#### 검증법 
1. 같은 객체를 두고 변경된 것을 비교한다.
2. return 값을 확인한다.
3. 레벨이 정해진 경우와 비어진 경우를 확인한다.
4. 파라미터로 넘긴 오브젝트의 필드를 확인한다.
5. DB에서 직접 get해와서 확인한다. **이게 확실**

### 코드 개선 
1. 코드에 중복된 부분은 없는가?
2. 코드가 무엇을 하는 것인지 이해하기 어려운가?
3. 코드가 자신이 있어야 하는 자리에 있는가?
4. 앞으로 변경이 일어난다면 어떤 것이 있고, 그에 대응하기 쉽게 작성되어있는가?

#### 리팩토링
1. 레벨 별로 많은 if문들, 노골적인 동작 코드
```java
if(user.getLevel() == Level.BASIC) user.setLevel(Level.SILVER);
else if(user.getLevel() == Level.SILVER) user.setLevel(Level.GOLD);
```
이는 예외처리도 없고, 조건을 잘못파악해서 이 메소드를 부르면 무조건 업그레이드가 될 것이다. 레벨이 늘어난다면 if 문도 쭈욱 길어질 것이다. 그러니까, 이렇게 수정해본다.
```java
public enum Level {
	GOLE(3,null), SILVER(2,GOLD), BASIC(1,SILVER);
	private final int value;
	private final Level next;
public Level nextLevel() {return this.next;}
}
```
next라는 다음 단계 레벨 정보를 담도록 필드를 추가하면 다음 단계의 레벨이 무엇인지 if 조건식을 쭈욱 만들어줄 필요가 없다.
**오브젝트에게 데이터를 요구하지 말고 작업을 요청하라는 것이 객체지향 프로그래밍의 가장 기본이 되는 원리이다.** 유저에서 데이터를 가져와서 작업하는 것이 아니라 유저에게 “레벨 업그레이드 작업을 해달라”고 요청하고, 유저는 레벨에게 “다음 레벨이 뭔지 알려달라”고 요청하는 것이다. 

```java
//sudo 코드 
@Test
public void upgradeLevels() {
public static final int SILVER=50;
private boolean canUpgradeLevel(User user) {
	switch(levle){
		case BASIC:return (user.getLogin() >= SILVER);
	}
}
}
```

### 트랜잭션 서비스
여러 오브젝트를 처리할 때 그 중 하나가 실패하면 다른 오브젝트들도 영향을 받을까? 하나의 트랜잭션 안에서 동작하지 않았다면 영향을 받지 않을 것이다. 하지만 영향을 받아야 하는 시나리오가 있을 수 있다. 모든 유저의 레벨을 업그레이드 시킨다고 한다면 누구는 되고, 누구는 안되는 시나리오는 발생하면 안된다. 여러 작업 중 하나라도 실패하면 취소(트랜잭션 롤백)해야 한다.

#### JDBC 트랜잭션의 경계 설정
트랜잭션이 한번 시작되면 commit() 또는 rollback() 메소드가 호출될 때까지의 작업이 하나의 트랜잭션으로 묶인다. 이 작업을 **트랜잭션 경계 설정**이라고 한다. 트랜잭션의 경계는 하나의 커넥션이 만들어지고 닫히는 범위 안에 존재한다. 하나의 DB 커넥션 안에서 만들어지는 트랜잭션을 **로컬 트랜잭션**이라고 한다.

![그림5-2](https://github.com/goyanglee/tobi_spring_study/blob/master/Vol.1%20스프링의%20이해와%20원리/5.%20서비스추상화/5장_ks/AF2D9C27-7302-460B-9BA3-42BAFFEC2F90.jpeg)

#### 로직 내의 트랜잭션 경계 설정 
커넥션을 공유하도록 한다. 
##### 한계점 
1. JDBC API 를 사용하는 방식을 사용해야 한다.
2. 비즈니스 로직을 담고있는 메소드에 공유할 커넥션 파라미터가 추가되어야 한다.
3. 그렇게 되면 데이터 엑세스 기술에 독립적일 수가 없다. dao의 구현방식을 변경하려고 하면 커넥션 대신에 세션을 받아야하기 때문에 결국 전부 수정이 일어나야 한다.
4. 테스트코드에도 영향을 준다.

#### 트랜잭션 동기화 사용
트랜잭션 동기화란, 트랜잭션을 시작하기 위해 만든 커넥션 오브젝트를 특별한 저장소에 보관하고 호출되는 dao 의 메소드에서는 이를 가져다 사용하게 한다. 트랜잭션이 종료되면 동기화를 마친다.
**TransactionSynchronizationManager**

![그림5-3](https://github.com/goyanglee/tobi_spring_study/blob/master/Vol.1%20스프링의%20이해와%20원리/5.%20서비스추상화/5장_ks/EC79D7CE-5BCC-4054-9291-0019326AF768.jpeg)

1. 커넥션을 생성하고
2. 동기화 저장소에 보관한 후
```java
TransactionSynchronizationManager.initSynchronization();
Connection c=DataSourceUtils.getConnection(dataSource);
c.setAutoCommit(false);
```
3. 트랜잭션을 시작한다. 
4. 저장소에 트랜잭션이 있는지 확인한 후 
5. 가져와서 sql 을 실행한다.
6. 커넥션을 가져왔을 경우 커넥션을 닫지 않는다. 
7. 트랜잭션 내의 모든 작업들이 정상적으로 완료됐으면 트랜잭션을 완료시킨다. 
```java
DataSourceUtils.releaseConnection(c,dataSource);
TransactionSynchronizationManager.unbindResource(this.dataSource);
```
8. 완료시키고 나면 커넥션을 제거한다.
```java
TransactionSynchronizationManager.clearSynchronization();
```

> 트랜잭션 저장소는 작업 스레드마다 독립적으로 커넥션을 관리하기 때문에 멀티쓰레드 환경에서도 충돌나지 않는다.

#### JdbcTemplate 과 트랜잭션 동기화 
JdbcTemplate은 트랜잭션 동기화 저장소에 등록된 커넥션을 확인한 후 작업을 진행한다.

### 트랜잭션 서비스 추상화 
하나의 트랜잭션 안에 여러 DB 로 작업을 해야하는 경우에는?
**글로벌 트랜잭션** 방식을 사용해야한다. 트랜잭션 매니저를 통해 여러 DB가 참여하는 작업을 하나의 트랜잭션으로 만들 수 있다. 또, JMS와 같은 트랜잭션 기능을 지원하는 서비스도 트랜잭션에 참여시킬 수 있다. 자바에서는 이를 지원하는 API 인, JTA(Java Transaction API)도 제공하고 있다.
애플리케이션은 트랜잭션 매니저를 통해 DB를 관리한다. 리소스 매니저와 트랜잭션 사이는 XA 프로토콜을 통해 연결된다.
> XA 프토토콜 : 분산 트랜잭션에서 데이터베이스와 연결하는 프로토콜

```java
InitialContext ctx.= new InitialContext();
UserTransaction tx = (UserTransaction) ctx.lookup(tx name);
tx.begin();
tx.commin();
tx.rollback();
```

> 하이버네이트는 connection 이 아니라 session을 사용한다.

#### 스프링 트랜잭션 서비스 추상화 
```java
PlatformTransactionManager transacitonManager = new DataSourceTransactionManager(dataSource);
TrnasactionStatus status=transactionManager.getTransaction(new DefaultTransactionDefinition());
try{
	userDato.getAll();
	transactionManager.commit(status);
}catch~
```
스프링이 제공하는 트랜잭션 경계 설정을 위한 추상 인터페이스는 platformTransationManager다. 
1. JDBC를 이용하는 경우 : DataSourceTransactionManager

#### 트랜잭션 기술 설정 분리 
PlatformTransactionManager -> JTATransactionManager

```java
PlatformTransactionManager txManager = new JTATransactionManager();
```

### 서비스 추상화와 단일 책임 원칙
지금까지 구현한 것은,
1. 애플리케이션 계층
2. 서비스 추상화 계층
3. 기술 서비스 계층
이다. 

#### 단일 책임 원칙 
객체지향의 설계의 원칙 중 하나. Single Responsibility Principle. 하나의 모듈은 하나의 책임을 가져야 한다. 하나의 모듈이 바뀌는 이유는 한 가지여야 한다.
##### 장점 
변경이 필요할 때 수정 대상이 명확해진다. 적절하게 책임과 관심이 다른 코드를 분리해서 서로 영향을 주지 않도록 다양한 추상화 기법을 도입하고 로직과 환경을 분리해서 복잡해지는 애플리케이션을 구성하면 수정이 용이해진다.

### 예제. 메일 서비스 추상화 
테스트 코드를 작성할 떄 메일이 보내져버리면 안된다.  그럴 땐 실제 메일이 전송되지 않는 테스트 메일 서버를 연결해서 테스트를 한다. 
그런데, JavaMail의 핵심 api는 인터페이스로 만들어져서 구현을 바꿀 수 없다. 애초에 확장이나 지원이 불가능하도록 만들어져서 그렇다. 단, 스프링은 이를 해결하기 위해 추상화 기능을 제공한다.
```java
javax.activation.jar
javax.mail.jar
context.supplort.jar
interface MailSender
```
* 테스트 대역(test double) : 테스트 대상이 되는 오브젝트의 기능에만 충실하게 수행하면서 사용하는 오브젝트
* 테스트 스텁(test stub) : 대표적인 테스트 대역. 
테스트 <-> 테스트 대상 오브젝트 <-> 목 오브젝트 로 동작하며 테스트 대상 오브젝트로 입출력 하는 것이 직접 테스트 구간, 테스트 대상 오브젝트에서 목 오브젝트로 입출력하는 것이 간접 테스트 구간이다.
