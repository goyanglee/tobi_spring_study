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

1. 파라미터 타입의 오브젝트를 만든다. 
2. 준비된 모델 오브젝트의 프로퍼티에 웹 파라미터를 바인딩해준다. 
3. 모델의 값을 검증한다. 
**바인딩**
1. xml 설정 
2. HTTP 를 통해 들어온 데이터를 모델 오브젝트등으로 변환 
### 바인딩 API 1. PropertyEditor 
스프링이 기본적으로 제공하는 바인딩용 타입 변환 API 이다. 
#### 디폴트 프로퍼티 에디터 
#### 커스텀 프로퍼티 에디터 
기본 구현이 되어 있는 PropertyEditorSupport 클래스를 상속하는 것이 낫다. 
##### @InitBinder 
@InitBinder가 붙은 initBinder() 는 메소드 파라미터를 바인딩하기 전에 자동으로 호출되어 WebDataNinder에 커스텀 프로퍼티 에디터를 추가할 수 있는 기회를 제공한다. 
**WebDataBinder에 커스텀 프로퍼티 등록하는 방법**
1. 특정 타입에 무조건 적용되도록 registerCustomerEditor() 를 사용 
2. 특정 이름의 프로퍼티에만 적용되는 프로퍼티 에디터 등록 
```java
@InitBinder
public void initBinder(WebDataBinder dataBinder) {
	dataBinder.regeterCustomEditor(int.class, “age”, new MinMaxPropertyEditor(0,200));
}
```
#### WebNindingInitializer 
```java
public class MyWebIndingInitialzier implements WebBindingInitializer {~}
```
#### 프로토타입 빈 프로퍼티 에디터 
프로퍼티 에디터는 싱글톤 빈으로 등록될 수 없다. 타입이 변경될 때 잠깐동안 데이터를 갖고 있기 때문에 멀티 쓰레드에서 공유해서 사용하면 안되기 때문이다. 프로토타입은 매번 생성하기 때문에 적격이다. 
**단순타입이 아닌 경우 바인딩하는 방법**
1. 별도의 codeid 필드로 바인딩 
```java 
@RequestParam(“/add”)
public void add(@ModelAttribute User user) {
	user.setUserType(this.codeService.getCode(user.getUserTypeId()));
}
```
가장 간단하지만 매번 COde의 정보를 DB에서 가져오기 때문에 성능상 문제가 생길 수 있다. 
2. 모조 오브젝트 프로퍼티 에디터 
모조 프로퍼티 에디터를 만든다. id 만 가진 모조 오브젝트(fake object) 를 가진 프로퍼티 에디터이다. 폼에서 수정하거나 등록한 사용자 정보를 단순히 저장하는 목적이라면 유용하고 다른 용도로 활용하지 않아야 한다. 
3. 프로토타입 도메인 오브젝트 프로퍼티 에디터 
디비에서 매번 데이터를 가져와야하기 때문에 성능상 문제가 생길 수 있다. 
### Converter 와 Formatter
#### Converter 
소스타입에서 타깃 타입으로 단방향 변환을 지원한다. 
```java
public class LevelToStringConveter implements Converter<Level, String> {
	public String convert(Level level) {
		return String.valueOf(level.intValue());
	}
}
```
#### ConversionService 
GenericConversionService 는 일반적으로 빈으로 등록하고 필요한 컨트롤러에서 주입받아서 WebDataBinder에 설정한다. 
1. @InitBinder를 통한 수동 등록 
일부 컨트롤러에만 직접 구성한 ConversionService를 적용하거나 다른 변환 서비스를 선택하고 싶을 때 설정한다. 				
• GenericConversionService를 상속해서 addConverter()를 사용해 추가하는 방법
• ConversionServiceFactoryBean을 이용해서 genericConvertionService를 가져온다. 
2. configurableWebBindingInitializer를 이용해 일괄 등록 
모든 컨트롤러에 한 번에 적용한다. 
#### Formatter 와 FormattingConversionService 
FormattingConversionServiceFactoryBean 을 사용했을 때 자동으로 등록되는 포맷. 
1. @NumberFormat : 다양한 타입의 숫자 변환을 지원 
2. @DateTimeFormat : Joda Time을 이용하는 어노테이션 기반 포맷터 
#### 바인딩 우선 순위 
1. 사용자 정의 타입 바인딩 일괄 적용 : Conveter
2. 조건부 변환 : ConditionalGenericConverter 
3. 어노테이션으로 HTTP 바인딩 : AnnotationFormatterFactory 와 Formatter 
4. 특정 필드만 변환 : PropertyEditor 
커스텀 프로퍼티 에디터 > 컨버전 서비스의 컨버터 > 내장 디폴트 프로퍼티 에디터 
### WebDataBinder 설정 
#### allowedFields
바인딩 허용 필드 목록 
#### disallowedFields 
바인딩 금지 필드 목록 
#### requiredFields 
파라미터가 없으면 예외 
#### fieldMarkerPrefix 
체크박스의 경우 감안해서 검증하는 방법. 
#### fieldDefaultPrefix 
디폴트 값은 ! 이다. 디폴트 값 설정 
```xml
<input type=“hidden” name=“!type” value=“member” />
```
type이라는 파라미터가 존재하지 않는다면 member값에 담긴 내용을 바인딩한다. 
### Validator 와 BindingResult, Errors 
#### Validator 
오브젝트 검증기 API . 오류가 발견되면 Errors에 오류 정보를 등록하고 최종적으로 BindingResult에 담겨 컨트롤러에 반환한다. 
보통 미리 정해진 단순 조건을 이용해 검증하는데 사용한다. 
1. 컨트롤러 메소드 내의 코드로 Validator 적용 
2. @Valid 를 이용해 자동 검증 
3. 서비스 계층 오브젝트에서의 검증 : 자주 사용되지는 않는다. 
4. 서비스 계층을 활용 : Validator 를 빈으로 등록해서 서비스 계층의 기능을 사용해 검증 
#### JSR-303 
LocalValidatorFactoryBean 을 이용해 JSR-303 검증 기능을 사용한다. @Min 이나 @NotNull 등이 그 예이다. 
```java
@Target({ElementType.Method, ElementType.FIELD)}
@Retention(RetentionPolicy.RUNTIME)
@Constraint(validatedBy=MemberNoValidator.class)
public @interface MemberNo {}
```
#### BindingResult, MessageCodeResolver 
BindingResult에는 모델의 바인딩 작업 중에 나타난 에러 정보를 담아 폼으로 다시 띄울때 사용한다. 
#### MessageSource 
**구현방법**
##### 코드 
Errors에 등록된 에러 코드를 DefaultMessageCodeResolver를 이용해서 필드와 이름의 조합으로 만들어낸 메시지 키 후보값이 있다. 
##### 메시지 파라미터 배열 
messages.properties에 설정 
##### 디폴트 메시지 
##### 지역정보 
LocaleResolver에 의해 결정된 현재의 지역정보를 이용한다. Locale.KOREAN 지역에서는 messages_ko.properties를 찾게 된다. 
### 모델의 라이프 사이클 
#### HTTP 요청으로부터 컨트롤러 메소드까지
!(http-controller)[https://github.com/goyanglee/tobi_spring_study/blob/master/Vol.%202%20스프링의%20기술과%20선택/4장.%20스프링%20%40MVC/4장_ks/79DBCF0D-03D8-4507-9694-A56A462EB624.jpeg]
1. @ModelAttribute 로 파라미터 가져오기 
2. @SessionAttribute 세션 확인 
3. WebDataBinder 에 등록된 변환 기능으로 타입 변환 
4. WebDataBinder에 등록된 검증기로 @Valid 가 지정되어 있는 것들을 검증 
5. ModelAndView 의 모델 맵. 임시 모델 맵에 저장되어 DispatcherServlet으로 전달 
6. 파라미터가 전달되어 컨트롤러 메소드 실행 
#### 컨트롤러 메소드로부터 뷰까지 
!(controller-view)[https://github.com/goyanglee/tobi_spring_study/blob/master/Vol.%202%20스프링의%20기술과%20선택/4장.%20스프링%20%40MVC/4장_ks/22FA04D4-1C39-426E-A417-4C6E627AE483.jpeg]
1. DispatcherServlet이 ModelAndView 을 받는다.
2. WebDataBinder에 등록되어있는 MessageCodeResolver가 메시지 코드 후보 목록을 만든다. 
3. 빈으로 등록된 MessageSource와 LocaleResolver : 최종 출력할 에러 메시지를 결정한다. 
4. @SessionAttribute로 지정한 이름과 일치하는 게 있다면 HTTP 세션에 저장된다. 
5. 뷰의 EL 과 스프링 태그 또는 매크로 : 뷰의 표현식 언어(EL)로 참조되어 콘텐츠에 포함되어 출력된다. 
