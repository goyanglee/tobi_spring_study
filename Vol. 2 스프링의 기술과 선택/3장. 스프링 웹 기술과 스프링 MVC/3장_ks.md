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
