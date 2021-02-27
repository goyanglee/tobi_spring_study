# 스프링 웹 기술과 스프링 MVC



웹 계층은 기술적으로 변화가 많은 계층이다. 스프링은 이런 상황을 대비해서 웹 계층을 다른 계층과 분리하여 개발하는 아키텍처 모델을 추구한다. 

<br/>

### [학습 목표]

- 스프링의 웹 계층 설계 기본원칙
- 스프링 웹 기술의 선정 기본원칙
- 스프링 웹 기술의 다양한 전략

<br/>

스프링에는 두 가지의 '서블릿 웹 애플리케이션 컨텍스트'가 존재한다. 

​	(1) 루트 애플리케이션 컨텍스트: 비즈니스 계층과 데이터 액세스 계층을 작성

​	(2) 서블릿 애플리케이션 컨텍스트: 스프링 웹 기술 기반으로 동작하는 웹 관련 빈 작성

> #### (!) 분리한 이유
>
> - 스프링 웹 서블릿 컨텍스트를 통쨰로 다른 기술로 대체할 수 있도록 하기 위함

<br/>

### 웹 프레임워크로 사용할 수 있는 것들

#### 스프링 웹 프레임워크

- **스프링 서블릿/스프링 MVC**
- 스프링 포틀릿

#### 스프링 포트폴리오 웹 프레임워크

- SWF (Spring Web Flow)
- Spring JavaScript
- Spring Faces
- Spring Web Service
- Spring BlazeDS Integration

#### 스프링 기반이 아닌 웹 프레임워크

- JSP/Servlet
- Struts1 
- Strust2
- Tapestry3, 4 (테피스트리)
- JSF/Seam

<br/>

스프링이 제공해준 MVC 프레임워크에 프로젝트 특성에 맞게 빠르고 편리한 개발이 가능하도록, 필요한 전략을 추가해서 사용할 줄 알아야 한다. 

### 디스패처 서블릿 : 스프링 웹 기술의 핵심/기반

> MVC : 모델 / 뷰 / 컨트롤러가 서로 협력해서 하나의 웹 요청을 처리하고 응답을 만들어내는 구조. 보통 '프론트 컨트롤러 패턴'과 함꼐 사용된다. 이 패턴은 컨트롤러를 프레젠테이션 계층의 가장 앞단에 둬서 서버로 들어오는 모든 요청을 받아서 처리하는 패턴이다. 프론트 컨트롤러의 역할은, 
>
> ​	1) 클라 요청을 받아서 공통작업 수행 후 적절한 세부 컨트롤러로 작업 위임해준다. 
>
> ​	2) 클라에게 보낼 뷰 선택해서 최종 결과 생성해준다.
>
> ​	3) 예외 발생 시 일관되게 처리해준다. 

#### 스프링 서블릿/MVC의 핵심 프론트 컨트롤러 => 디스패처 서블릿

![이미지1](https://github.com/goyanglee/tobi_spring_study/blob/master/Vol.%202%20%EC%8A%A4%ED%94%84%EB%A7%81%EC%9D%98%20%EA%B8%B0%EC%88%A0%EA%B3%BC%20%EC%84%A0%ED%83%9D/3%EC%9E%A5.%20%EC%8A%A4%ED%94%84%EB%A7%81%20%EC%9B%B9%20%EA%B8%B0%EC%88%A0%EA%B3%BC%20%EC%8A%A4%ED%94%84%EB%A7%81%20MVC/3%EC%9E%A5_sh/%E1%84%80%E1%85%B3%E1%84%85%E1%85%B5%E1%86%B73-1.jpeg)

1) HTTP 요청이 들어오고 / 이 요청이 스프링의 디스패처서블릿에 할당된 요청인지 서블릿컨테이너가 판단해서 / 맞으면 서블릿 컨테이너가 HTTP 요청정보를 디스패처 서블릿에 전달해준다. 

- 디스패처서블릿에 할당된 요청인 지 확인은 web.xml에 작성된 '디스패처 서블릿이 전달받을 URL 패턴'을 보고 판단함
- HTTP 요청은 GET과 POST로 구분되고 쿼리스트링/폼파라미터/쿠키/헤더 같은 정보가 전달됨

<br/>

2) 디스패처서블릿이 URL이나 파라미터정보 HTTP 명령 등을 참고해서 어떤 컨트롤러(핸들러)에게 작업을 위임할 지 결정한다. 이것을 '핸들러 매핑전략'이라고 한다. 그리고 난 후 결정한 컨트롤러오브젝의 메소드를 호출해서 실제 웹 요청을 처리하는 작업을 위임한다. 

- 디스패처서블릿과 컨트롤러 사이의 호출방법
  - 디스패처서블릿은 컨트롤러에 담은 정보를 모른다. 대신 컨트롤러 종류에 따라 적절한 어댑터를 사용한다. 각 어댑터는 담당 컨트롤러에 맞는 호출방식을 사용해서 컨트롤러에 작업 요청을 보내고 결과를 돌려받아서 디스패처서블릿에게 돌려준다. 
  - 디스패처서블릿이 핸들러 어댑터에 웹 요청을 전달할 때는 모든 웹 요청 정보가 담긴 HttpServletRequest 타입의 오브젝트를 전달해준다. 이것을 어댑터가 적절히 변환해서 컨트롤러의 메소드가 받을 수 있는 파라미터로 전달해주는 것이다. 

<br/>

3) 컨트롤러의 모델 생성과 정보 등록. 컨트롤러가 디스패처서블릿에게 돌려줘야할 두 가지정보는 모델과 뷰이다. 

4) 컨트롤러 결과 리턴: 컨트롤러가 뷰오브젝트를 직접 리턴할 수도있지만 대부분 뷰 논리 이름을 리턴해주면 뷰리졸버가 이것을 사용해서 뷰 오브젝트를 생성해준다. 이 때 뷰를 사용하는 전략과 리턴방법도 다양하다. 

5) 디스패처서블릿이 컨트롤러에게 모델과 뷰를 받으면, 뷰 오브젝트에게 모델을 전달해주고, 클라에게 돌려줄 최종 결과물을 생성해달라고 요청한다. 뷰 작업을 통한 최종결과물은 HttpServletResponse 오브젝트에 담긴다.

6) 디스패처서블릿은 등록된 후처리기가 있으면 후속작업 진행 후에, HttpServleResponse에 담긴 최종 결과를 서블릿컨테이너에게 돌려준다. 서블릿 컨테이너는 이 결과정보를 HTTP 응답으로 만들어서 클라에게 전송하고 작업이 종료된다.

<br/>

#### 디스패처서블릿에는 DI로 확장할 수 있는 전략이있다.

- HandlerMapping (URL과 요청정보 기준으로 어떤 컨트롤러를 사용할 지 결정하는 로직 담당)
- HandlerAdapter (선택한 컨트롤러를 디스패처서블릿이 호출할 때 사용함)
  - 컨트롤러 타입에 적합한 어댑터를 사용해서 호출한다. 디폴트로 등록되어 있는 핸들러어댑터는 'HttpRequestHandlerAdapter', 'SimpleControllerHandlerAdapter', 'AnnotationMethodHandlerAdapter' 3가지이다. 
- HandlerExceptionResolver (예외 발생 시 처리하는 로직 담당)
  - 디스패처서블릿은 발생한 예외에 적합한 핸들러익셉션리졸버를 찾아서 예외처리를 위임한다. 디폴트 전략은 'AnnotationMethodHandlerExceptionResolver', 'ResponseStatusExceptionResolver', 'DefaultHandlerExceptionResolver' 세 가지가 등록되어있다. 
- ViewResolver (컨트롤러가 리턴한 뷰 이름 참고해서 적절한 뷰 오브젝트 찾아주는 로직 담당)
- LocaleResolver (지역정보 결정)
  - 디폴트는 'AcceptHeaderLocaleResolver'. HTTP 헤더정보를 보고 지역정보 설정해준다. 
- ThemeResolver (테마정보 결정해주는 전략)
- RequestToViewNameTranslator (컨트롤러에서 뷰이름이나 뷰 오브젝트를 제공해주지 않았을 경우 URL과 같은 요청정보를 참고해서 자동으로 뷰 이름을 생성해줌)

<br/>

> #### 디스패처서블릿은 각 전략의 디폴트 설정을 Dispatcher.properties라는 전략 설정파일에서 가져와서 초기화한다. 위 전략을 통해 디스패처서블릿의 동작방식을 확장 가능하다. 
>
> 디스패처서블릿은 서블릿컨테이너가 생성하고 관리하는 오브젝트이고 스프링의 컨텍스트에서 관리하는 빈 오브젝트가 아니다. 따라서 디스패처서블릿에 직접 DI설정은 할 수 없다. 대신 내부에 서블릿 웹 어플리케이션 컨텍스트를 갖고 있고 여기에 개발자가 전략을 추가하거나 설정을 수정한 빈 오브젝트를 담을 수 있다. 추가해준 빈이있으면 이것을 디폴트전략 대신 가져와서 사용한다. 
> 
<br/>

서비스계층이나 데이터액세스계층은 독립적으로 만들어져서 테스트환경에서 쉽게 실행이 가능하지만, 

웹 계층의 경우에는, 서버에 배치하고 클라이언트를 통해 접근할 수 있는 환경을 구성해야 정확한 테스트가 가능하다.

<br/>

## 웹 계층을 테스트하기 위한 환경을 준비해보기

### STS을 사용해서 스프링 웹프로젝트를 생성하는 두 가지 방법

1. Spring MVC Project 선택하기

   -스프링 MVC 템플릿 생성됨

   -빌드방식은 메이븐 사용

   생소한 설정방식과 템플릿 때문에 초보자는 어려울 수 있음

2. Dynamic Web Project 선택하기

   -기본 웹 프로젝트 생성

   -필요한 라이브러리 직접 추가

   -필요한 설정파일 직접 작성 : web.xml과 스프링 설정파일을 직접 작성해줘야 한다

   -번거롭지만 초보자의 경우 도움 많이 됨

<br/>

### Dynamic Web Project를 선택해서 웹 프로젝트 만들어보기

> #### 전제조건
>
> - 필요한 런타임 라이브러리 jar 파일은 WebContent/WEB-INF/lib에 추가
> - 웹 어플리케이션의 컨텍스트를 구성하는 방법 중. 루트컨텍스트와 서블릿컨텍스트를 사용하는 방법 선택
>   - (!) 서블릿컨텍스트가 루트컨텍스트를 부모로 가지고 / 자식컨텍스트는 부모컨텍스트를 참조할 수는 있지만 그 반대는 불가능하다. 

<br/>

#### [1] 루트 컨텍스트와 서블릿 컨텍스트를 web.xml을 사용해서 등록한다.

- 루트컨텍스트 등록

  ```
  <listener>
  	<display-name>ContextLoader</display-name>
  	<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
  </listener>
  ```

  - web.xml에 리스너를 등록하면 루트컨텍스트가 생성됨

  - 디폴트 설정파일은 "/WEB-INF/applicationContext.xml" 

    - 웹 환경과 직접관련이 없는 모든 빈은 여기에 등록된다 (서비스계층 + 데이터액세스계층)
    - (!) STS 사용 시 [File] > [New] > [new Spring Bean Definition File]로 생성가능
    - 보편적 경로: WebContent/WEB-INF/applicationContext.xml
      - WebContent 폴더 = 웹루트

  - 루트 컨텍스트가 제대로 등록됐는 지 테스트하려면?

    - 컨트롤러 하나 생성해서 applicationContext.xml에 빈으로 등록해주고

      ```
      <bean id="helloString" class="com.sh.HelloSpring" />
      ```

    - "JSP 모델 1" 방식을 사용해서 테스트할 수 있음 (hellospring.jsp)

      ```
      ...
      <%@page import="org.springframework.context.ApplicationContext" %>
      <$@page import="org.springframework.web.context.support.WebApplicationContextUtils"%>
      <$@page import="com.sh.HelloSpring"%>
      
      <html>
      ...
      <body>
      <%
      	ApplicationContext context = WebApplicationContextUtils.getWebApplicationContext(request.getSession.getServletContext()); 
      	HelloSpring helloSpring = context.getBean(HelloSpring.class); 
      	out.println(helloSpring.sayHello("Test!"));
      %>
      </body>
      </html>
      ```

      - 스프링 MVC 외의 웹 기술에서 스프링의 애플리케이션 컨텍스트를 사용하려면 'WebApplicationContextUtils'를 이용하면 된다. 

    - STS의 [Run] > [Run As]  하면 STS에 포함된 SpringSource tcServer를 통해 간단한 배치와 서버 구동까지 할 수 있다. 웹 브라우저에 /hellospring.jsp 입력해서 확인

  <br/>

- 서블릿 컨텍스트 등록

  ```
  <servlet>
  	<servlet-name>spring</servlet-name>
  	<servlet-class>org.springframework.web.servlet.DispatcherServlce</servlet-class>
  	<load-on-startup>1</load-on-startup>
  </servlet>
  <servlet-mapping>
  	<servlet-name>spring</servlet-name>
  	<url-pattern>/app/*</url-pattern>
  <servlet-mapping>
  ```

  - web.xml에 DispatcherServlet 서블릿을 추가해서 서블릿 컨텍스트를 등록한다. 

    - DispatcherServlet이 담당할 HTTP 요청의 URL 패턴을 지정해준다. 
    - 이렇게 등록되면, DispatcherServlet은 초기화 시에 서블릿 수준의 웹 애플리케이션 컨텍스트를 생성해준다. 
    - 서블릿 컨텍스트가 사용할 설정파일명은 "-servlet.xml"
    - 웹루트/WEB-INF 아래에 -servlet.xml 만들어두면 DispatcherServlet의 내부에 생성되는 서블릿 컨텍스트가 이 파일을 사용해서 컨텍스트를 초기화해준다. 

  - 디스패처서블릿과 그 내부의 서블릿 컨텍스트가 제대로 동작하는 지 확인할 수 있는 MVC 코드를 작성한다. (스프링 MVC를 사용하려면 디스패처서블릿 전략을 선택하고 그에 맞는 코드를 작성해야 함)

    - 디폴트전략 사용해보기 (컨트롤러: 디폴트전략에서 지원하는 컨트롤러 타입 사용. 뷰: 디폴트 전략에서 지원하는 JSP 사용)

      - 디폴트 핸들러어댑터: SimpleControllerHandlerAdapter

      ```
      public interface Controller {
      	ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception; 
      }
      ```

      ```
      public class HelloController implements Controller {
      	@Autowired HelloSpring helloSpring; 
      	
      	public ModelAndView handleRequest(HttpServletRquest req, HttpServletResponse res) throws Exception {
      		String name = req.getParameter("name"); 
      		String message = this.helloSpring.sayHello(name); 
      		
      		Map<String, Object> model = new HashMap<String, Object>(); 
      		model.put("message", message); 
      		
      		return new MedelAndView("/WEB-INF/view/hello.jsp", model); 
      	}
      }
      ```

      ​		- HelloController의 handleRequest()는 Controller 타입 핸들러를 담당하는 SimpleControllerhandlerAdapter를 통해 디스패처서블릿으로부터 호출된다. 

      - 서블릿 컨텍스트가 사용할 설정파일 "spring-servlet.xml" 안에 HelloController 빈을 등록한다.

        ```
        <context:annotation-config />
        <bean name="/hello" class="com.sh.HelloController" />
        ```

        - 실제로는 /app/hello에 매핑됨

      - 디폴트 뷰 리졸버 InternalResourceViewResolver를 사용한다. 컨트롤러가 리턴한 뷰 이름을 참고해서 JSP와 서블릿을 템플릿으로 사용하는 InternalResourceView를 돌려준다. 

        ```
        <html>
        <head>...</head>
        <body>
        	${message}
        </body>
        </html>
        ```

      - http://localhost:8080/shproj/app/hello?name=seok 요청해서 확인
