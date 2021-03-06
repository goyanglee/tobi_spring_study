
# Guide
1. 템플릿/콜백에 대해 이해한다.

# 스프링에 적용된 템플릿 기법
## 템플릿이란
변경이 거의 일어나지 않으며 일정한 패턴으로 유지되는 특성을 가진 부분을 자유롭게 변경되는 성질을 가진 부분으로부터 독립시켜서 효과적으로 활용할 수 있도록 하는 방법이다.

## 예외 처리 기능을 추가하기 - try/catch/ finally 구문
try : 예외가 발생할 가능성이 있는 코드
catch : 예외가 발생했을 때 작업할 코드
finally : try에서 예외가 발생하든 아니든 무조건 실행하게 되는 코드
-> 예외 발생 지점이 어디냐에 따라 finally에서 실행할 코드가 다르고, finally에서도 예외가 발생할 수 있기 때문에 내부에 또다른 try/catch 구문을 사용하는게 좋다.

### 예외 기능의 문제점
1. try/catch 의 중첩
2. 같은 코드의 중복

### 해결법
1. 변하는 코드를 재사용할 수 있도록 분리시킨다.
2. 템플릿 메소드 패턴을 이용한다. 변하지 않는 부분은 슈퍼클래스에 두고 변하는 부분은 추상 메소드로 정의해둬서, 서브클래스가 이를 오버라이드해서 새롭게 정의해 쓰도록 한다.
> 템플릿 메소드 패턴 : 상속을 통해 기능을 확장해서 사용하는 패턴. 단, 로직마다 상속을 통해 새로운 클래스를 만들어야하는 단점이 있다.
3. 전략 패턴 적용 - 구체적인 전략 클래스를 사용하도록 고정되어 있으면 전략을 바꿔쓸 수 있다는 전략 패턴의 장점을 무시하게 된다.
> 전략 패턴 : 확장에 해당하는 변하는 부분을 별도의 클래스로 만들어 추상화된 인터페이스를 통해 위임하는 방식이다.
4. DI : 클라이언트가 행위를 정하게 됨.
> public void jdbcContextWithStatementStrategy(StatementStrategy stmt) throws SQLException {
Connection c = null;
PreparedStatement ps = null;

try{
c = dataSource.getConnection();
ps = stmt.makePreparedStatement(c);
ps.executeUpdate();
} catch~
}

## 전략패턴 최적화
![리스트3-13 도식화](https://github.com/goyanglee/tobi_spring_study/blob/master/Vol.1%20스프링의%20이해와%20원리/3.%20템플릿/3장_ks/B32D6818-434F-4814-8F0A-497916532186.jpeg)
<br/>
그림과 같은 방식으로 설계를 하면 메소드를 공유하게 되면서 try/catch/finally 구문을 반복해서 사용할 필요가 없게된다.  
<br/>
* 단점 : 
1. DAO 메소드마다 새로운 구현체를 만들어야 한다.(기존 로직마다 상속을 사용하게 했던 템플릿 메소드 패턴과 다를게 없어진다.) 
2. 사용할 파라미터가 있는 경우 이를 저장할 변수와 생성자를 계속 만들어주어야 한다. 
* 해결책:
1. 로컬 클래스.
<br/>
중첩 클래스를 사용한다. 클래스 안에 클래스를 선언하는 방법. 단점 1,2번을 보완할 수 있다. 
<br/>
> 중첩 클래스 : 스태틱 클래스(독립적으로 오브젝트가 만들어질 수 있다.), 내부클래스(자신이 정의된 클래스 안에서만 만들어진다. final 로 선언된 자신이 정의된 클래스 안의 변수를 사용할 수 있다.) 
<br/>
2. 익명 내부 클래스. 
<br/>
메소드를 사용할 시점에만 유효한 익명 내부 클래스에서 로직을 서술한다. 

## 클래스 독립시키기

기존 방식에서 JDBC 작업 흐름을 가지고 있는 부분은 다른 DAO 에서도 사용 가능하다. 그렇다면 바깥으로 독립시키는 것은 어떨까?

### 클래스 분리 
JDBC가 의존하고 있는 DataSource 타입의 빈을 DI 받을 수 있게 setter를 만들어준다.

### 분리된 클래스 사용 
기존 DAO는 분리된 JdbcContext 클래스를 setter로 DI 받는다.
이렇게 되면 스프링의 DI는 인터페이스를 사이에 두고 의존 클래스를 바꿔서 사용하는 것이 목적인데 이를 위반한다. 하지만 기능이 구현이 바뀔 가능성이 없는 독립적이다. 그래서 이 구조가 되었다.
![현재 의존관계](https://github.com/goyanglee/tobi_spring_study/blob/master/Vol.1%20스프링의%20이해와%20원리/3.%20템플릿/3장_ks/53E519DF-B6C1-4DA8-9948-19E102C7D884.jpeg)

### 분리된 클래스 주입 방식
 1. 스프링 빈으로 
인터페이스를 사용하지 않고 주입하는 방식은 엄밀히 말하면 온전한 DI라고 볼 수는 없지만 외부에 제어권을 위임했다는 넓은 의미로 보면 부합한다. DAO 와 JdbcContext는 강한 응집도를 가지고 있다. 다른 구현으로 대체할 필요가 없을 정도이기 때문에 이를 허용할 수 있다.
스프링의 빈을 사용해야하는 이유는 무엇일까?
	1. 스프링 컨테이너의 싱글톤 레지스트리에서 관리되는 싱글톤 빈이된다. 그래서 여러 오브젝트에서 공유해서 사용할 수 있게 된다.
	2. DataSource를 의존하고 있다. 그래서 DataSource도 빈으로 등록해놓으면 스프링이 생성하고 관리해준다.
그렇다면 장점은 무엇일까?
	1. 클래스들의 의존 관계를 설정파일을 통해 명확하게 알 수 있다.
단점은?
	1. 엄밀히 말해 DI에 부합하지 않은 의존관계가 노출된다. 


2. 수동으로 
DAO 마다 직접 주입한다. JdbcContext가 의존하고 있는 DataSource도 DAO 가 주입받아 JdbcContext에 직접 주입해준다.
이렇게 해도 기껏해야 DAO 개수만큼 만들어지기 때문에 부담은 거의 없다. 그래서 아래와 같은 구조가 된다.
![수동주입](https://github.com/goyanglee/tobi_spring_study/blob/master/Vol.1%20스프링의%20이해와%20원리/3.%20템플릿/3장_ks/D7A4F0AC-29FB-48BE-A163-D2A5FB1DD538.jpeg)
이 방식의 장점은?
인터페이스가 굳이 필요없을 정도로 긴밀한 관계를 갖는 것들은 어색하게 따로 빈으로 만들지 않고 직접 만들어 사용하면서도 수동 주입을 할 수 있다. 따라서 관계가 외부에 드러나지 않아 전략을 감출 수 있다.
단점은?
	1. 여러 오브젝트가 사용하더라도 싱글톤으로 만들 수 없다.
	2. 수동주입을 위한 별도의 코드(생성자, setter)가 필요하다. 


## 템플릿/콜백 패턴

템플릿 : 바뀌지 않는 부분. 컨텍스트 
콜백 : 자주 바뀌는 부분. functional object 
![템플릿콜백 그림](https://github.com/goyanglee/tobi_spring_study/blob/master/Vol.1%20스프링의%20이해와%20원리/3.%20템플릿/3장_ks/065EA090-73D5-455F-97A6-3B59D60D6035.jpeg)

> 전략패턴(참조 1.3.4) : 전략에 따라 구현 클래스를 바꿔 사용하는 패턴 
![전략패턴 그림](https://github.com/goyanglee/tobi_spring_study/blob/master/Vol.1%20스프링의%20이해와%20원리/3.%20템플릿/3장_ks/0360D33B-F438-47B6-A946-DC120EA9031F.jpeg)

-> 스프링에서는 템플릿/콜백 패턴이라고 한다.

### 특징

1. 일반적으로 단일 메소드 인터페이스를 사용한다.
-> 보통 특정 기능을 위해 한 번 호출되기 때문이다.
2. 콜백 인터페이스의 메소드에는 보통 파라미터가 있다 
-> 컨텍스트 정보를 파라미터로 전달받음
3. 메소드 레벨의 DI 이다.
4. 의존하는 오브젝트를 setter로 전달받아두고 사용하는 방식과는 달리, 메소드 단위로 사용할 오브젝트를 새롭게 전달받는다.
5. 자신을 생성한 클라이언트 메소드 내의 정보를 직접 참조할 수 있다. 강하게 결합되어있다.
6. 전략 패턴 + DI + 익명 내부클래스 사용

### 단점

1. 매번 익명 내부 클래스 코드를 작성해야한다.

![그림3-7](https://github.com/goyanglee/tobi_spring_study/blob/master/Vol.1%20스프링의%20이해와%20원리/3.%20템플릿/3장_ks/5F2242A7-D96E-430E-8C76-A044C8537182.jpeg)
![그림3-9](https://github.com/goyanglee/tobi_spring_study/blob/master/Vol.1%20스프링의%20이해와%20원리/3.%20템플릿/3장_ks/2D64FE22-0045-4BDC-BDB3-82DF3076D35B.jpeg)

### 템플릿/콜백 패턴을 적용시킬 수 있는 것

1. 반복되는 try/catch/finally 구조의 코드
2. 제네릭스를 이용할 수 있는 것 
: 리턴타입과 파라미터를 제네릭스 T를 활용한다. 인풋할 때 타입을 지정하면 리턴도 맞는 타입으로 변경되어 나온다.

## 스프링의 템플릿/콜백
JdbcTemplate를 사용해서 그동안 만들 것들을 대체할 수 있다.

![그림 3-58](https://github.com/goyanglee/tobi_spring_study/blob/master/Vol.1%20스프링의%20이해와%20원리/3.%20템플릿/3장_ks/EA28474A-A06C-4606-BE2B-F0654C8F6584.jpeg)
![그림 3-58 2](https://github.com/goyanglee/tobi_spring_study/blob/master/Vol.1%20스프링의%20이해와%20원리/3.%20템플릿/3장_ks/0E000316-6B90-47CD-8CE8-53191CE613D2.jpeg)

### 개선점
1. userMapper를 DI 로 바꿀 수 있다. 
2. SQL을 외부에서 가져올 수 있다.
