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
