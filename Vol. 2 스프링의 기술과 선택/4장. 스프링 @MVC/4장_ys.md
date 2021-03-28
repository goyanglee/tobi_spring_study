# 4장. 스프링 mvc

스프링 MVC 프레임워크의 장점은 유연한 확장이 가능하도록 설계된 MVC 엔진이 DispatcherServlet이다. DispatcherServlet이 사용하는 7가지 전략을 바탕으로 MVC프레임워크 기능과 동작방식을 재구성하거나 확장할 수 있다.

# 4.1 @RequestMapping 핸들러 매핑

@MVC의 가장 큰 특징은 핸들러 매핑과 어댑터의 대상이 오브젝트가 아니라, 메소드라는 점이다.

@MVC에서는 모든것이 메소드 레벨로 세분화됐다.

메소드 레벨에서 컨트롤러를 작성할 수 있게 된 기술적인 배경에는 어노테이션을 이용한 프로그래밍 모델이 있다. 어노테이션 타입(클래스,인터페이스) 레벨뿐 아니라 메소드 레벨에도 적용이 가능하기 때문이다.

@MVC가 등장하기 전의 컨트롤러는 타입을 비교해서 컨트롤러를 선택하고 타입에 정의된 메소드를 통해 실행하는 방법을 사용했다면,

@MVC의 컨트롤러를 기존의 인터페이스와 같은 타입을 이용해서 하던 일을 어노테이션으로 대체해버렸다. 어노테이션은 부여되는 대상의 타입이나 코드에서는 영향을 주지 않는 메타정보이므로 훨씬 유연한 방식으로 컨트롤러를 구성할 수 있게 해준다.

@MVC의 핸들러 매핑을 위해서는 DefaultAnnotationHandlerMapping이 필요하다. DefaultAnnotationHandlerMapping은 디폴트 핸들러 매핑 전략이므로 다른 핸들러 매핑 빈을 명시적으로 등록하지 않았다면 기본적으로 사용할 수 있다. 다른 핸들러 매핑 빈을 등록했을 경우에는 디폴트 핸들러 매핑 전략이 전용되지 않으므로 DefaultAnnotationHandlerMapping도 함께 빈으로 등록해줘야 한다.

## 4.1.1 클래스/메서드 결합 매핑정보

DefaultAnnotationHandlerMapping의 핵심은 매핑정보로 @RequestMapping 어노테이션을 활용한다는 점이다.

- String[] value():URL패턴 : @RequestMapping("/user/{userId}")
- RequestMethod[] method():HTTP 요청 메소드 : @RequestMapping(value="/user/add", method=RequestMethod.GET)
- String[] params() : 요청 파라미터
- String[] headers():HTTP헤더

### 타입 레벨 매핑과 메소드 레벨 매핑의 결합

타입 레벨에 붙는 @RequestMapping은 타입 내의 모든 매핑용 메서도의 공통 조건을 지정할 때 사용한다.

### 메소드 레벨 단독 매핑

### 타입 레벨 단독 매핑

## 4.1.2 타입 상속과 매핑

- 상위 타입과 메소드 @RequestMapping 상속
- 상위 타입 @RequestMapping과 하위타입 메서드 @RequestMapping의 결합
- 상위 타입 메소드 @RequestMapping과 하위타입 @RequestMapping 결합
- 하위 타입과 메소드 @RequestMapping 재정의
- 서브클래스 메소드 URL 패턴 없는 @RequestMapping 재정의

# 4.2 @Controller

핸들러 어댑터는 DispatcherServlet으로부터 HttpServletRequest와 HttpServletResponce를 받아 이를 컨트롤러가 사용하는 파라미터 타입으로 변환해서 제공해준다. 그리고 컨트롤러로부터 받은 결과를 ModelAndView 타입의 오브젝트에 담아서 DispatcherServlet에 돌려준다.

그래서 Controller는 DispatcherServlet과 가장 자연스럽게 연동되는 컨트롤러 타입이다.

## 4.2.1 메소드 파리미터의 종류

HttpServletRequest,HttpServletResponse

httpSession

WebRequest, NativeWebRequest