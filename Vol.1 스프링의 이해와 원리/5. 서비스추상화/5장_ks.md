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
