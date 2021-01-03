

# IoC컨테이너와 DI
## IoC컨테이너 : 빈 팩토리와 애플리케이션 컨텍스트 
오브젝트 사이에 런타임 관계를 설정하는 DI관점으로 볼 때 컨테이너를 빈 팩토리 + 여러 컨테이너 기능을 추가한 것 = 애플리케이션 컨텍스트  = 스프링 IoC컨테이너
### POJO 클래스 
### 설정 메타 정보
POJO 클래스를 어떻게 빈으로 만들고 동작하게 할 것인지에 대해 서술한 메타 정보를 XML, properties 등의 파일에 담고 BeanDefinition 인터페이스로 만든 오브젝트로 만든다.
#### 사용하는 메타정보
1. 빈 아이디, 이름, 별칭
2. 클래스 또는 클래스 이름
3. 스코프 : 싱글톤, 프로토타입,..
4. DI에 사용할 프로퍼티값 혹은 참조 
5. 생성자 파라미터 혹은 참조
6. 우선순위, 자동와이어링, 지연된 로딩 등의 대한 정보 

#### 만들어지는 방식 
[!그림 10-2](url)
POJO 클래스와 설정 메타정보를 이용해 IoC 컨테이너가 만들어주는 오브젝트의 조합을 스프링 애플리케이션이라고 한다. 

### IoC 컨테이너의 종류와 사용방법 
#### StaticApplicationContext
학습 외에 잘 사용되진 않는다.
#### GenericApplicationContext
일반적으로 사용되며 특정 포맷의 빈 설정 메타 정보를 읽어서 만든다. 
#### GenericXmlApplicationContext
GenericApplication에 xml설정 파일을 사용할 경우 자동으로 해준다. refresh()를 통해 초기화하는 것까지 끝낼 수 있다.
#### WebApplicationContext
가장 많이 사용된다. 웹 환경에서 사용할 때 필요한 기능이 추가되어있고, WmlWebApplicationContext가 디폴트고 많이 사용된다. 
IoC 컨테이너를 기동하는 방법에는 자바의 main() 메소드처럼 어딘가에서 처음 한 번은 특정한 빈 오브젝트의 메소드를 호출해야 서로 연결된 오브젝트들이 호출되면서 애플리케이션이 동작한다. IoC 컨테이너는 최초로 애플리케이션을 기동할 빈 하나를 제공해주는 역할을 한다. 
[!그림 10-1](url)

서블릿 컨테이너는 클라이언트로부터 들어오는 요청을 받아서 서블릿을 동작시켜준다. 그리고 미리 지정된 메소드를 호출함으로서 애플리케이션의 기능이 시작된다.

### IoC 컨테이너 계층구조 
[!그림 10-4](url)
때에 따라서 애플리케이션 컨텍스트는 계층 구조를 가질 수 있다. 부모는 자식 컨텍스트에 요청하지 않고, 자식 컨텍스트는 부모 컨텍스트에서 빈을 찾을 수 있다. 같은 빈이 있다면 자신의 컨텍스트의 우선순위가 높다. 

```
ApplicationContext parent = new GenericXmlApplicationContext(path);
```
```
GenericApplicationContext child = new GenericApplicationContext(parent);
XmlBeanDefinitionReader reader = new XmlBeanDefinitionReader(child);
reader.loadBeanDefinitions(path);
child.refresh();
```

### IoC 컨테이너의 구성 
#### 웹 모듈 안에 컨테이너를 두는 방법 
프론트 컨트롤러 패턴 : 몇 개의 서블릿이 중앙집중식으로 모든 요청을 다 받아서 처리하는 방식.
1. 서블릿 안에서 IoC컨테이너 생성
2. 웹 애플리케이션 레벨에서 IoC컨테이너 생성 
을 모두 사용해 컨테이너를 만든다. 
#### 엔터프라이즈 애플리케이션 레벨에 컨테이너를 두는 방법 
#### 계층 구조 
각 서블릿 별로 애플리케이션이 생성되고 공통된 빈들은 보통 웹 애플리케이션 레벨의 컨텍스트에 등록한다.
[!그림 10-5] (url)
일반적으로는 한 개의 서블릿 컨텍스트를 갖는다. 
#### 구성 방법
1. 서블릿 컨텍스트와 루트 애플리케이션 컨텍스트 계층 구조 
가장 기본. 스프링 기술 관련 빈들은 서블릿의 컨텍스트에 두고 나머지는 루트 애플리케이션 컨텍스트에 둔다.
2. 루트 애플리케이션 컨텍스트 단일 구조 
스프링 기술을 사용하지 않는 경우 스프링 서블릿을 둘 필요가 없기 때문에 단일 루트만 등록한다.
3. 서블릿 컨텍스트 단일 구조 
스프링과 스프링 외 기술을 사용할 생각이 아니라면 루트를 제외한다.

#### 루트 애플리케이션 컨텍스트 등록하기 
1. 서블릿의 이벤트 리스너 사용. 
웹 앱 시작과 종료시 발생하는 이벤트를 처리하는 ServletContextListener를 구현한 ContextLoaderListener를 등록한다. 
• contextConfigLocation 
디폴트 xml 파일 위치 지정
• contextClass
기본 XmlWebApplicationContext를 변경

#### 서블릿 애플리케이션 컨텍스트 등록하기 
DispatcherServlet 선언 
• <servlet-name> : 컨텍스트를 구분하는 키. 보통 /WEB-INF/서블릿네임스페이스.xml 로 만든다.
• <load-on-startup> : 순서 지정



