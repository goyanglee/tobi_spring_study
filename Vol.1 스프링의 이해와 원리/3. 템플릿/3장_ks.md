
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

