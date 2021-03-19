## @RequestMapping 핸들러 매핑 
@MVC의 가장 큰 특징은 핸들러 매핑과 핸들러 어댑터의 대상이 오브젝트가 아니라 메소드라는 점이다. URL 당 하나의 컨트롤러 오브젝트가 매핑되고 하나의 메소드만 DispatcherServlet의 호출을 받을 수 있다.
DefaultAnnotationHandlerMapping이 빈으로 등록되어 있어야 한다.
### 클래스/메소드 결합 매핑정보
DefaultAnnotationHandlerMapping은 @RequestMapping 어노테이션을 활용하고, 타입 레벨, 메소드 레벨도 붙일 수 있다. 

#### RequestMapping 어노테이션 
##### String[] value() : URL 패턴 
```java
@RequestMapping(“/hello”)
@RequestMapping(“/admin/.*”)
@RequestMapping(“/admin/*”)
@RequestMapping(“/admin/**”)
@RequestMapping(“/admin/{userId}”)
@RequestMapping(“/admin”,”/hi”)
```
##### RequestMethod[] method() : HTTP 요청 메소드 
```java
@RequestMapping(“/admin/.*”, method=RequestMethod.GET)
```
##### String[] params() : 요청 파라미터 
```java
@RequestMapping(“/admin/.*”, params=“type=admin”)
@RequestMapping(“/admin/.*”, params=“!type”) //특정 파라미터가 존재하지 않아야 한다는 조건 지정 

```
같은 매핑정보가 있을 때 좀 더 상세한 매핑조건이 우선된다.
##### String[] headers() : HTTP GPEJ 
```java
@RequestMapping(“/admin/.*”, headers=“content-type=text/*”)
```
#### 타입 레벨 매핑과 메소드 레벨 매핑의 결합 
```java
@RequestMapping(“/user”)
public class UserController {
	@RequestMapping(“/add”) public String add(...){} ///user/add 로 결합된다.
}
```
#### 메소드 레벨 단독 매핑 
```java
public class UserController {
	@RequestMapping(“/add”) public String add(...){} 
}
```
#### 타입 레벨 단독 매핑 
```java
@RequestMapping(“/user”)
public class UserController {}
```
```java
@RequestMapping(“/user/*”)
public class UserController {
	@RequestMapping public String add(...){} ///user/add 로 매핑된다.
}
```
### 타입 상속과 매핑 
@RequestMapping 정보는 상속된다. 단, 서브클래스에서 @RequestMapping을 재정의하면 슈퍼클래스의 정보는 무시된다. 
#### 매핑정보 상속의 종류 
##### 상위 타입과 메소드의 @RequestMapping 상속 
인터페이스에 작성한 @Requestmapping은 하위 클래스에서 그대로 상속받는다.
##### 상위 타입의 @RequestMapping 과 하위 타입 메소드의 @RequestMapping 결합 
인터페이스에 작성된 url 과 결합된다.
```java
@RequestMapping(“/user”) public class Super{}

public class Sub extends Super {
@Requestmapping(“/list”) public String list() {}
}

//일 때 /user/list로 매핑된다.
```
##### 상위 타입 메소드의 @RequestMapping 과 하위 타입의 @RequestMapping 결합 
```java
public class Super {
@RequestMapping(“list”) public String list() {}
}
@RequestMapping(“/user”) public class Sub extends Super{}
```
의 경우에는 오버라이드한 메소드만 /user/list 로 매핑된다.
##### 하위 타입과 메소드의 @RequestMapping 재정의 
서브클래스에서 재정의하면 슈퍼클래스의 정의 내용은 무시된다. 
##### 서브클래스 메소드의 URL 패턴없는 @RequestMapping 재정의 
재정의할 때 url 조건이 없다면 상위 메소드의 url 조건이 그대로 상속된다. 
#### 제네릭스와 매핑정보 상속을 이용한 컨트롤러 작성 
상위 타입에는 타입 파라미터와 메소드 레벨의 공통 매핑 정보를 지정해놓고, 이를 상속받는 개별 컨트롤러에는 구체적인 타입과 클래스 레벨의 기준 매핑 정보를 지정해주는 기법을 사용할 수 있다. 
```java
public abstract class GenericController<T,K,S> {
	S service;
	@RequestMapping(“/add”) public void add(T entity){}
	@RequestMapping(“/view”) public T view(K id){}
	...
}
```
개별 컨트롤러는 GenericController를 상속해서 만들면 된다.
@ModelAttribute가 붙은 파라미터 타입의 오브젝트에 모두 담아서 전달해주는 것을 커맨드라고 부른다. 웹페이지의 폼 내용을 받을 때에도 @ModelAttribute를 사용한다. 
@ModelAttribute도 생략 가능하다. 이는 String, int 등은 @RequestParam으로 보고 복잡한 오브젝트는 ModelAttribute가 생략되었다고 간주한다. 그러나 명시적으로 작성해주는 것을 지향한다.

## @Controller 
### 메소드 파라미터의 종류 
#### HttpServletRequest, HttpServletResponse
#### HttpSession 
#### WebRequest, NativeWebRequest
#### Locale
#### InputStream, Reader
#### OutputStream, Writer
#### Pathvariable 
#### RequestParam 
#### @RequestHeader
#### Map, Model, ModelMap 
#### @ModelAttribute 
#### Errors, BindingResult 
검증 작업이 추가적으로 진행된다. 요청 파라미터의 타입 변환을 시도한다. 하지만 @ModelAttribute를 사용할 때에 발생하는 실패는 예상한 것으로 간주하고 400 error가 아니라 하나의 결과로서 반환한다. 예를 들어서 사용자 로그인 폼 같은 경우에는 사용자가 실수로 잘못된 값을 넣을 수도 있다. 실패를 예상한다는 것은 이를 의미한다. 
BindingResult나 Error 타입을 사용하고, @ModelAttribute 뒤에 파라미터로 존재한다. 
#### SessionStatus 
모델 오브젝트는 세션에 저장하고 다시 활용할 수 있게 하는 기능 
#### @RequestBody 
HTTP 요청의 본문 부분을 XMl 이나 Json 등으로 컨버팅해서 사용한다. 
1. HttpMessageConverter 
2. StringHttpMessageConverter
3. MarshallingHttpMessageConverter 
4. MappingJacksonHttpMessageConverter 
#### @Value 
주로 시스템 프로퍼티나 다른 빈의 프로퍼티 값, 좀 더 복잡한 SpEL 을 이용해 클래스의 상수를 읽어오거나 호출한 값, 조건식 등을 넣을 수 있다. 
```java
@Value(“#{systemProperties[‘os.name’]}”) String osName
```
#### @Valid 
JSR-303의 빈 검증기를 통해 모델 오브젝트를 검증하는 지시자 
### 리턴타입의 종류 
컨트롤러가 DispatcherServlet에 돌려줘야 하는 정보는 모델과 뷰이고, 드물게 HttpServletResponse를 줄 때도 있다. 
#### 자동 추가 모델 오브젝트와 자동생성 뷰 이름 
다음 네 가지는 메소드 리턴 타입에 상관없이 조건만 맞으면 모델에 자동으로 추가된다. 
1. @ModelAttribute 모델 오브젝트 또는 커맨드 오브젝트 
2. Map, Model, ModelMap 파라미터
3. @ModelAttribute 
단, @RequestMapping을 함께 붙이지 말아야 한다.
```java
@ModelAttribute(“codes”)
public List<Code> codes() { return codeService.getAllCodes();}
```
4. BindingResult 
#### ModelAndView 
자주 사용되지는 않는다. ModelAndView 에 addObject() 를 통해 오브젝트를 추가할 수 있다. 
#### String 
메소드의 리턴 타입이 스트링이면 이 리턴 값은 뷰 이름으로 사용된다. 
#### Void
메소드의 리턴 타입이 Void 라면 RequestToViewNameResolver에 의해 자동생성되는 뷰 이름을 사용한다. 
#### 모델 오브젝트 
뷰 이름을 자동 생성으로 사용하고, 하나의 모델 오브젝트를 갖는 경우라면 모델 오브젝트를 직접 리턴해줘도 된다.
```java 
@RequestMapping(“/view”)
public User view(@RequestParam int id) {
	return userService.getUser(id);
}
```
#### Map/ModelModelMap 
Map 타입의 리턴 값은 그 자체로 모델 맵으로 인식해서 엔트리 하나하나를 개별적인 모델로 다시 등록해버린다. 
#### View 
#### @ResponseBody 
### @SessionAttributes 와 SessionStatus 
Http 요청에 의해 동작하는 서블릿은 기본적으로 상태를 유지하지 않지만 유지해줘야하는 정보가 있을 때가 있다. 
#### 도메인 중심 프로그래밍 모델과 상태 유지를 위한 세션 도입의 필요성 
예를 들어서 사용자 정보 수정을 할 때에는 수정 전에 사용자 정보를 보여줘야 한다. 여러번 호출해서 가져오는 것은 비효율 적이다. 또, 일부만 수정했을 경우 일부의 값만 요청되어져서 데이터가 잘못 변경될 수도 있다.
##### 히든 필드 
```xml
<input type=“hidden” />
```
문제점
1. 데이터 보안에 심각한 문제를 일으킨다. HTML 소스를 열어보면 모두 알 수 있다. 
2. 사용자 필드가 추가되거나 수정되면 그때마다 히든필드의 수정이 필요하다. 
##### DB 재조회 
문제점
1. 성능 이슈 
2. 수정했을 경우 어떤 값이 수정했는지 하나하나 확인해야한다.
##### 계층 사이의 강한 결합 
계층끼리 강한 결합을 맺는다. 즉, 다른 계층에서 어떤 값이 들어올 지 정확하게 알고 대응한다.
문제점 
1. 수정이 일어날 경우 연관되어 있는 모든 계층에 전부 수정이 일어나야 한다. 
#### @SessionAttributes
이를 해결하기 위해 세션을 사용한다. 
1. @SessionAttributes에 지정한 이름과 동일한 모델 정보가 있다면 이를 세션에 저장해준다. 
2. @ModelAttribute가 지정된 파라미터가 있을 때 세션에서 가져와서 전달해준다. 
#### SessionStatus 
사용을 완료한 데이터는 개발자가 직접 해제해주어야한다. 
## 모델 바인딩과 검증 
