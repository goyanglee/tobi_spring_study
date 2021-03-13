# @MVC (= 어노테이션 기반의 MVC)

#### [학습목표]

- 어노테이션 기반 스프링 MVC의 상세한 기능과 활용 전략

<br/>

## @RequestMapping

@MVC에서는 모든 것이 메소드레벨로 세분화되었다. 핸들러매핑과 핸들러어댑터의 대상이 오브젝트가 아닌 "메소드"라는 점이 가장 큰 특징이다.

> (!) @MVC의 핸들러매핑전략
>
> - DefaultAnnotationHandlerMapping이 디폴트 (명시하지 않아도 됨) 
>
>   => @RequestMapping을 매핑정보로 활용하는 전략
>
> - 다른 핸들러매핑 빈을 등록했을 땐 디폴트도 함께 명시해줘야 함

<br/>

#### 엘리먼트들

- String[] value() 

  - URL 패턴 지정
  - 와일드카드(*) 지정 가능
  - { } 사용해서 패스변수 하나 이상 지정가능
  - 하나 이상의 URL 패턴 지정 가능
  - 디폴트 접미어 패턴 적용됨
    - "/sh" : 확장자가 붙은 URL("/sh.do", "/sh.html"), '/'로 끝나는 URL("/hello/") 모두 매핑됨

- RequestMethod[] method()

  - HTTP 메소드 지정
  - GET / HEAD / POST / PUT / DELETE / OPTIONS / TRACE
  - 같은 URL도 메소드에 따라 다르게 매핑 가능
  - 여러개 지정 가능

  > (!) HTML 폼에서는 GET과 POST만 지원한다. PUT이나 DELETE 사용하려면 자바스크립트를 이용하거나 스프링이 지원하는 커스텀 태그 < form:form >을 이용해서 히든필드를 통해 HTTP 메소드를 전달하는 방법을 사용해야 한다. 

  <br/>

- String[] params() 

  - 요청 파라미터 지정

  - 세번쨰 매핑방식이다. 요청파라미터와 그 값을 비교해서 매핑해줌

    - 같은 URL에서 HTTP 요청 파라미터에 따라 별도의 작업을 해주고자 할 때 사용

      > 예)
      >
      > 1) @RequestMapping(value="/user/edit", params="type=admin")
      >
      > /user/edit?type=admin 요청에 매핑됨
      >
      > 2) @RequestMapping(value="/user/edit", params="type=member")
      >
      > /user/edit?type=member 요청에 매핑됨

      <br/>

      > 만약에 3) @RequestMapping(value="/user/edit") 이라는 매핑을 추가했을 때 /user/edit?type=admin 매핑은 어디로 갈지?
      >
      > => 3)이 아닌 1)로 간다. 좀 더 많은 조건을 충족시키는 메소드를 탄다. 

    - params에 지정한 파라미터는 URL에 포함된 것 말고도 폼 파라미터도 대상으로 한다. 

    - 특정 파라미터가 존재하지 않아야한다는 조건도 지정 가능

      - @RequestMapping(value="/user/edit" param="!type")

    - 여러개 지정 가능 

- String[] headers()

  - HTTP 헤더정보 지정
  - params와 비슷하게 '헤더이름 = 값' 형식 사용

<br/>

#### 메소드 매핑은 클래스 매핑을 상속받는다

클래스와 인터페이스처럼 '타입' 레벨에 붙는 @RequestMapping은 메소드들의 공통조건을 지정할 때 사용한다. 

> (*) 공통조건이 없어도 클래스 등 타입레벨에는 "조건없는 @RequestMapping"을 붙여줘야한다. 생략하면 클래스 자체가 매핑 대상이 되지 않기 때문에. 단, @Controller 어노테이션을 클래스에 붙여서 빈 자동스캔방식으로 등록되게 했다면 클래스레벨의 @RequestMapping은 생략해도 된다. 

<br/>

#### 핸들러 매핑과 핸들러 어댑터는 독립적으로 조합될 수 있다. 

어노테이션 방식의 핸들러매핑에서도 오브젝트까지만 매핑을 하고 최종 실행할 메소드는 핸들러 어댑터가 선정한다.

따라서 @RequestMapping을 타입 레벨에 단독으로 사용해서 다른 타입 컨트롤러에 대한 매핑을 위해 사용할 수도 있다. 

```
@RequestMapping("/sh")
public class SHContorller implements Controller {
...
}
```

<br/>

#### URL 패턴을 메소드명으로 사용할 수 있다.

클래스레벨의 URL패턴을 "/*"로 끝나도록 설정한 경우에는 메소드 이름을 메소드레벨의 URL 패턴으로 사용할 수 있다. 

```
@RequestMapping("/sh/*")
public class SHController {
	@RequestMapping public String go(...){} //-> /sh/go에 매핑
	@RequestMapping public String stay(...){} //-> /sh/stay에 매핑 
}
```

<br/>

### @Controller의 메소드 파라미터

- HttpServletRequest, HttpServletResponse

- HttpSession

  - HttpServletRequest에서 가져올 수도 있음. 세션만 필요한 경우에는 직접 선언해서 사용하는 것이 좋음
  - 서버에 따라 멀티스레드 안정성이 보장되지 않을 수 있음. 서버 상관없이 안전성을 위해 핸들러어댑터의 "synchronizeOnSession"을 "true"로 설정해줘야한다. 

- WebRequest, NativeWebRequest

  - WebRequest : HttpServletRequest의 요청정보 대부분을 그대로 갖고 있음 (스프링 서블릿/MVC의 컨트롤러에서 필수적인 것은 아니다)
  - NativeWebRequest : WebRequest 내부에 감춰진 HttpServletRequest와 같은 환경종속적인 오브젝트를 가져올 수 있는 메소드가 포함되어있음

- Locale

  - DispatcherServlet의 지역정보 리졸버가 결정한 Locale 오브젝트를 받을 수 있음

- InputStream, Reader

- OutputStream, Writer

- @PathVariable

- @RequestParam

  - 파라미터 이름을 지정하지않고 Map<String, String> 타입으로 선언하면 모든 요청 파라미터를 담은 맵으로 받을 수 있다. 

    ```
    public String add(@RequestParam Map<String, String> params) { ... }
    ```

  - 파라미터가 선언되어있으면 무조건 값이 넘어와야한다. 안넘어오면 400에러 발생됨. 선택적으로 제공하게 하려면 "required" 옵션 사용

    ```
    public void view(@RequestParam(value="id", required=false, defaultValue="-1") int id) { ... }
    ```

  - 메소드 파라미터의 이름과 요청파라미터 이름이 일치한다면 @RequestParam의 이름을 생략할 수도 있다. 하지만, 파라미터가 많아지면 명시적으로 이름을 사용하는 것을 권장.

    ```
    public String view(@RequestParam int id) { ... }
    ```

- @CookieValue

  - HTTP 요청과 함께 전달된 쿠키 값을 메소드 파라미터에 넣어줄 수 있다. 쿠키이름을 넣어줌. 이 때도 쿠키이름과 메소드 파라미터 이름이 동일하면 쿠키이름은 생략가능

    ```
    public String check(@CookieValue("auth") String auth) { ... }
    ```

  - 선언되었으면 쿠키값이 반드시 존재해야 한다. 선택적으로 제공하려면 "reqired" 옵션사용

    ```
    public String check(@CookieValue(value="auth", required=false, defaultValue="NONE") String auth) { ... }
    ```

    - 쿠키값 없을때는 "NONE"으로 설정

- @RequestHeader

  ```
  public void header(@RequestHeader("Host") String host, @RequestHeader("Keep-Alive") long keepAlive)
  ```

  - 헤더도 선언했으면 반드시 존재해야한다. 

- Map, Model, ModelMap

  - Java.util.Map / org.springframework.ui.Model / org.springframework.ui.ModelMap
  - Model과 ModelMap은 모두 addAttribute() 메소드를 제공한다. 

- @ModelAttribute

  - 모델 맵에 담겨서 뷰에 전달되는 모델 오브젝트의 하나로 별도 설정없이도 자동으로 뷰에 전달된다. 

    > (*) 
    >
    > 컨트롤러가 사용하는 모델 중에는 "클라이언트로부터 받는 HTTP 요청정보를 이욯해서 생성되는 것"이 있다. 웹 페이지 폼처럼 컨트롤러가 받아서 내부 로직에서 사용하고 필요에 따라 다시 화면에 출력하기도하는 케이스. 
    >
    > 이렇게 클라이언트로부터 컨트롤러가 받는 요청정보 중 하나 이상의 값을 가진 오브젝트 형태로 만들 수 있는 구조적인 정보를 @ModelAttribute라고 부른다. 즉, 컨트롤러가 전달받은 오브젝트 형태의 정보를 의미한다. 

  - @RequestParam을 사용하는 것과 @ModelAttribute를 사용하는 것?

    - 정보의 종류가 다른것은 아님
    - 전자는, 요청파라미터를 메소드파라미터에서 1:1로 받는 것
    - 후자는, 요청 파라미터를 도메인오브젝트나 DTO 프로퍼티에 바인딩해서 한 번에 받는 것

  - Errors, BindingRequest

   


