# 4장 예외
## 예외의 종류
### Error
시스템에 뭔가 비정상적인 상황이 발생한 경우. 주로 JVM에서 발생시킨다. 대응방법이 없고, 시스템 레벨의 개발이 아니라면 무시해도 된다.

### Exception
1. 체크 예외 : 예외 처리를 강제한다. Exception 의 서브클래스이면서 RuntimeException 을 상속하지 않는 Exception
2. 언체크 예외 : 명시적인 예외 처리를 강제하지 않는다. 피할 수 있는 에러이며 주로 개발자의 부주의로 발생할 수 있는 예외들이다. Exception 의 서브클래스이면서 RuntimeException을 상속하는 Exception
ex) NullPointerException, IllegalArgumentException,..

RuntimeException도 Exception을 상속하지만 RuntimeException부터 상속하는 모든 Exception을 언체크예외라고 한다.

## 예외의 처리
### 예외 복구
사용자에게 알려서 다른 작업의 흐름을 유도하는 방법. 또는 일정시간 재시도를 하기도 한다. 복구 가능성이 있는 경우 사용한다.

### 예외 회피 
호출한 메소드로 책임을 throw한다. 무분별하게 던질 부작용이 있다.

### 예외 전환 
적절한 예외를 전환해서 던진다. 의미를 분명하게 하는 구체적인 Exception 을 던져서 받는 쪽에서 예외를 짐작할 수 있기 한다.
1. 중첩 예외 
처음 발생한 예외를 확인할 수 있다.

~~~ java
    catch (Exception e) {
        throw {Exception class}.initCause(e)
    }
 ~~~

2. 포장(wrapping)
주로 체크예외를 언체크 예외로 바꾸는 경우 사용한다. -> 이해가 안되서 다시 확인 필요


