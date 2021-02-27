## 스프링의 웹 프레젠테이션 계층 기술
### 스프링에서 사용되는 웹 프레임워크 종류 
#### 스프링 웹 프레임워크
##### 스프링 서블릿/스프링 MVC 
DispatchserServlet을 핵심 엔진으로 사용하고, 어노테이션 설정과 핸들러 메소드를 지원하는 스프링 MVC가 가장 대표적으로 사용되는 스프링 서블릿 기반 기술.
##### 스프링 포틀릿
스프링이 제공하는 포틀릿 MVC 프레임워크 
#### 스프링 포트폴리오 웹 프레임워크 
##### 스프링 web flox 
SWF로 불리기도 한다. 스프링 MVC와 양 축을 이루는 프레임워크.
##### Spring Javascript
자바스크립트 툴킷 Dojo를 추상화한 것. Ajax를 구축할 수 있다.
##### Spring Faces
JSF를 스프링 MVC와 스프링 SWF의 뷰로 사용할 수 있게 해주는 프레임워크 
##### Spring Web Service
SOAP 기반의 웹 서비스 개발을 가능하게 해주는 프레임워크 
##### Spring BlazeDS Integration
어도비 플렉스의 BlazeDS와 통합

#### 스프링을 기반으로 두지 않는 프레임워크 
##### JSP/Servlet
모델1방식의 JSP 나 서블릿을 웹 프레젠테이션 계층으로 사용할 수 있다.
##### Struts1
MVC 프레임워크 
##### Struts2
##### Tapestry 3,4
##### JSF/Seam

### 스프링 MVC와 DispatcherServlet 전략 
DispatcherServlet은 다양한 전략을 DI로 구성해서 확장하도록 만들어진 스프링 MVC/서블릿의 엔진과 같다.

#### DispatcherServlet 과 MVC 아키텍처 
프레젠테이션 계층의 구성요소를 담은 모델(M) + 화면 출력 로직을 담은 뷰(V) + 제어로직을 담은 컨트롤러(C) = MVC
보통 프론트 컨트롤러 패턴과 함께 사용된다. 프론트 컨트롤러란 중앙집중형 컨트롤러를 프레젠테이션 계층의 제일 앞에 둬서 서버로 들어오는 모든 요청을 먼저 받아서 처리하게 만드는 패턴이다.
1. DispatcherServlet이 HTTP 요청 접수 
2. DispatcherSerlvet에서 컨트롤러로 HTTP 요청 위임 
3. 컨트롤러의 모델 생성과 정보 등록 
4. 컨트롤러의 결과 리턴 : 모델과 뷰 . 보통 뷰의 논리적 이름을 리턴하면 뷰 리졸버가 이를 뷰 오브젝트로 생성해준다.(ModelAndView) 대표적으로 JSP 가 있다. 
5. DispatcherServlet의 뷰 호출과 6번 모델 참조 
6. HTTP 응답 돌려주기 

#### DispatcherServlet 의 DI 가능한 전략 
##### HandlerMapping : 어떤 컨트롤러, 핸들러를 사용할 것인지 결정 
##### HandlerAdapter : 선택한 컨트롤러와 핸들러를 DispatcherServlet이 호출할 때 사용하는 어댑터. @RequestMapping 과 @Controller 어노테이션을 통해 정의되는 컨트롤러의 경우는 DefaultAnnotationHandlerMapping 에 의해 핸들러가 결정되ㄴ다.
##### HandlerExceptionResolver
##### ViewResolver
##### LocalResolver
##### ThemeResolver
##### RequestToViewNameTranslator 

## 스프링 웹 애플리케이션 환경 구성
### 간단한 스프링 웹 프로젝트 생성
#### 웹 애플리케이션 컨텍스트 구성방법
##### 루트 웹 애플리케이션 컨텍스트
/WEB-INF/applicationContext.xml 에 리스너를 이용해 등록
```xml
<listener>
	<display-name>ContextLoader</display-name>
	<listner-class>org.springframework.web.context.ContextLoaderListener</listenr-class>
</listener>
```
##### 서블릿 웹 애플리케이션 컨텍스트
```xml
<servlet>
	<servlet-name>sevletname</servlet-name>
	<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
	<load-on-startup>1</load-on-startup>
	//url 패턴도 지정 가능
</servlet>
```
#### 스프링 웹 프로젝트 검증 
* 핸들러 어댑터 : SimpleControllerHandlerAdapter
* 핸들러 매핑 : BeanNameUrlHandlerMapping
* 뷰 리졸버 : InternalResourceViewResolver

### 스프링 웹 학습 테스트
#### 서블릿 테스트용 목 오브젝트 
```java
MockHttpServletRequest req = new mockHttpServletRequest("GET", "/hello");
req.addParameter("name","Spring");
SimpleGetServlet servlet = new SimpleGetServlet();
servlet.service(req, res);
```
* MockHttpServletRequest : 서블릿에 전달할 HttpServletRequest 타입의 요청정보
* MockHttpServletResponse
* MockHttpSession
* MockServletConfig, MockServletContext

#### DispatcherServlet 을 확장해서 테스트
##### ConfigurableDispatcherServlet
```java
ConfigurableDispatcherServlet servlet = new ConfigurableDispatcherServlet();
servlet.setRelativeLocations(getClass(), "spring-servlet.xml");
```
##### AbstactDispatcherServletTest 
참고 리스트12-17

## 컨트롤러
### 컨트롤러의 종류와 핸들러 어댑터 
#### Servlet과 SimpleServletHandlerAdapter
표준 서블릿. 서블릿을 컨트롤러로 사용했을 때 서블릿 클래스 코드를 그대로 유지하면서 스프링 빈으로 등록된다. 이러면 스프링 애플리케이션에 맞게 포팅할 수 있다. 
```java
//리스트 12-21
setClasses(SimpleServletHandlerAdpter.class, HelloServlet.class); //핸들러 어댑터와 컨트롤러를 빈으로 등록해준다.
```
```xml
<bean class=“org.springframework.web.servlet.handler.SimpleServletHandlerAdapter” />
```
이러면 DispatcherServlet이 자동으로 감지해 디폴트 핸들러 어댑터를 대신해서 사용한다.
서블릿 타입의 컨트롤러는 모델과 뷰를 리턴하지 않는다.

#### HttpRequestHandler와 HttpRequestHandlerAdapter
Http 프로토콜을 기반으로 한 전용 서비스를 만들려고 할 때 사용한다.

#### Controller와 SimpleConterollerHandlerAdapter
어노테이션과 관례를 이용한 컨트롤러가 나오기 전까지 스프링 MVC 컨트롤러라고 하면 이를 의미했다.
* synchronizeOnSession : Http 세션에 대한 동기화 여부 결정 
* supportedMethods : 컨트롤러가 허용하는 Http 메소드를 지정할 수 있다.
* useExpiresHeader, useCacheControllHeader, useCacheControlNoStore, cacheSeconds : Http헤더를 이용해서 브라우저의 캐시 설정 정보를 보내줄 것인지를 결정한다.

```java
public class HelloController extends SimpeController {
	public HelloController() {
		this.setRequiredParams(new Stringp[{“name”}];
		this.setViewName(“/WEB-INF/view/hello.jsp”);
	}
}
```

```java
public abstract class SimpleController implements Controller {
	private String[] requiredParams;
	private String viewName;
	// setRequiredParams
	//setViewName
	final public ModelAndView handlerRequest(~) {
		~
	}
	public abstract void control(Map~);
}
```

#### AnnotationMethodHandlerAdapter
클래스나 메소드에 붙은 애노테이션의 정보와 메소드 이름, 파라미터, 리턴 타입에 대한 규칙을 분석해서 컨트롤러를 선별하고 호출 방식을 결정한다.
DefaultAnnotationHandlerMapping 과 함께 사용해야한다.

### 핸들러 매핑 
#### BeanNameUrlHandlerMapping
url을 http 요청의 url과 비교해서 반환 
#### ControllerBeanNameHandlerMapping
빈의 아이디나 빈 이름을 이용해 매핑 
#### ControllerClassNameHandlerMapping
클래스 이름을 url에 매핑 
#### SimpleUrlHandlerMapping
url과 컨트롤러 매핑 정보를 한곳에 모아서 관리
#### DefaultAnnotationHandlerMapping
@RequestMapping 을 사용해서 클래스나 메소드에 직접 부여하고 매핑.
#### 공통 설정 정보 
* order : 두 개 이상의 핸들러 매핑을 적용했을 때 url 매핑정보가 중복되는 경우를 위해 지정하는 우선순위
* defaultHandler : url을 매핑할 대상을 찾지 못했을 경우 자동으로 선택해준다.
* alwaysUserFullPath : 보통은 url 을 상대경로 기준으로 사용하는데 full path 를 사용해야하는 경우 사용
* detectHandlersInAncestorContexts : 핸들러 매핑 클래스는 루트가 아니라 현재 컨텍스트 안에서 매핑할 컨트롤러를 찾는다. 이 옵션이 true가 되면 루트에서도 찾게 된다.

### 핸들러 인터셉터 
DispatcherServlet이 컨트롤러를 호출하기 전과 후에 요청과 응답을 참조하거나 가공할 수 있는 일종의 필터.
#### HandlerInterceptor
* boolean preHandle : 컨트롤러가 호출되기 전에 실행 
* void postHandler : 컨트롤러를 실행하고 난 후 호출 
* void afterCompletion : 모든 뷰에서 최종 결과를 생성하는 일을 포함한 모든 작업이 완료된 후 실행 

### 핸들러 입터셉터 적용 
```xml
<bean class=“org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping”>
	<property name=“interceptors”>
		<list>
			<ref bean=“simpleInterceptor”/>
			<ref bean==“eventInterceptor”/>
		</list>
	</property>
</bean>
```
서블릿 필터와 핸들러 인터셉터는 하는일이 비슷하다.
서블릿 필터는 web.xml에 별도로 등록해줘야하고 스프링의 빈이 아니다. 하지만 애플리케이션으로 들어오는 모든 요청에 적용된다.
핸들러 인터셉터는 DispatcherServlet의 특정 핸들러 매핑으로 제한된다는 제약이 있지만 스프링의 빈으로 등록된다.

### 컨트롤러 확장 
#### 커스텀 컨트롤러 인터페이스와 핸들러 어댑터 개발 
```java
@Retention(RetentionPolicy.RUNTIME)
@Inherited
public @interface ViewName{
	String value();
}

@Retention(RetentionPolicy.RUNTIME)
@Inherited
public @interface RequiredParams {
	String[] value();
}

public class HelloController implements SimpleController {
	@VuewName(“/WEB-INF/view/hello.jsp”)
	@RequuiredParams({“name”})
	public void control(Map~) {
		model.put(“message”,”Hello”);
	}
}

public class SimpleHandlerAdapter implements HandlerAdapter {
~
public ModelAndView handle(~) {
	Method m=ReflectionUtils.findMethod(handler.getClass(), “control”, Map.class, Map.class);
	ViewName viewName=AnnotationUtils.getAnnotation(m, viewName.class);
	RequiredParams rp = AnnotationUtils.getAnnotation(m, RequiredParams.class);
	~
}
}
```
핸들러 매핑에서 HelloController를 찾게 되면 DispatcherServlet은 현재 등록된 모든 핸들러 어댑터의 supports() 메소드를 호출해서 HelloController 타입을 처리할 수 있는지 물어보고 handler() 메소드를 호출해서 컨트롤러를 실행한다.
