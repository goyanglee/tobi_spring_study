

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

## 빈 설정 메타정보 작성
### 빈 설정 메타 정보
| 이름  |  내용  |  디폴트 |
|——|———|————|
|beanClassName |빈 오브젝트 클래스 명 | 없음. 필수값|
|parentName|부모 BeanDefinition의 이름|없음|
|factoryBeanName| 팩토리를 사용해 빈을 생성하는 경우 팩토리 빈의 이름 | 없음|
|factoryMethodName|다른 빈 또는 클래스의 메소드를 통해 생성하는 경우 메소드의 명 | 없음 |
|scope|빈 오브젝트의 생명 주기.크게 싱글톤, 비싱글톤으로 구분|싱글톤|
|lazyInit|빈 오브젝트의 생성 지연 여부.|false|
|dependsOn|먼저 만들어져야 하는 빈 지정|없음|
|autowireCandidate|autowired할 대상으로 지정 여부|true|
|primary|autowired시 여러 di 후보가 생길 때 우선순위 지정 여부|false|
|abstract|추상 빈 생성 여부|false|
|autowireMode| autowired 전략 | 없음|
|dependencyCheck|프로퍼티나 레퍼런스가 모두 설정되어있는지 체크|체크하지 않음|
|initMethod|di 후 초기화 할 메소드 명|없음|
|destroyMethod|빈 생명주기가 끝나고 제거하기 전 메소드 명|없음|
|propertyValued|프로퍼티 이름과 설정 값 또는 레퍼런스|없음|
|constructorArgumentValued|생성자 이름과 설정 값 또는 레퍼런스|없음|
|annotationMetadata|빈 클래스에 있는 어노테이션과 그 애트리뷰트 값|없음|


### 빈 등록 방법
#### xml<bean> 태그 
<bean id=“” class=“”>
<property name=“”>
<bean class=“”/>
</property>
</bean> 의 형식으로 정의한다.

#### xml 네임스페이스와 전용 태그
애플리케이션 컨텍스트는 스스로에게 빈을 di 시켜서 유연하게 동작하기도 한다. 그런 경우에는 개발자가 만드는 빈과 차이점이 없으므로 <aop:pointcut id=“” expression=“”/>와 같이 네임스페이스를 사용해 선언해서 구분한다.

#### 자동등록 
빈 스캐닝 : 특정 어노테이션이 붙은 클래스를 자동으로 찾아서 등록 
1. <context:component-scan> 태그를 사용
2. AnnotationConfigApplicationContext 사용. 빈 스캐너 내장 
스테레오타입 어노테이션 : 디폴트 필터에 적용되는 어노테이션 
|어노테이션|적용 대상|
|@Repository|dao 또는 레파지토리 클래스에 사용. aop의 적용 대상을 선정하기 위해서도 사용|
|@Service| 서비스 계층의 클래스에 사용|
|@Controller|MVC 컨틀로러에 사용되며 컨트롤러 빈으로 선정|

#### 자바코드로 빈 등록 
@Configuration 어노테이션이 달린 클래스를 이용해서 빈 설정 메타정보를 담아서 스프링 컨테이너가 인식할 수 있는 빈 메타정보 겸 빈 오브젝트 팩토리가 된다. 
이 때, 빈 생성 메소드를 다른 곳에서 호출하면 새로운 객체가 생성될까? 결과는 아니다. 빈 생성 메소드를 호출하면 이미 빈으로서 생성된 객체를 반환하게 된다. 왜냐하면 디폴트 스코프가 싱글톤이기 때문이다. 
@Configuration 스스로도 빈으로 생성이 된다. 

#### 자바코드로 빈을 등록할 경우 장점 
1. 컴파일러를 통해 타입 검증이 가능하다.
2. IDE 의 자동완성 같은 기능을 최대한 이용할 수 있다.
3. 이해하기 쉽다.
4. 빈 설정이나 초기화를 손쉽게 적용할 수 있다.

#### 일반 빈 클래스의 @Bean 메소드 
위에서 빈으로 생성 후 생성한 메소드를 계속 호출하면 같은 객체가 나온다고 했다. 단, 이 경우는 @Configuration 클래스 안의 @Bean 생성에만 적용이다. @Configuration 이 없이 @Bean만으로 빈을 생성해버리면 매번 다른 객체가 반환된다. 
@configuation 없이 같은 객체가 반환되도록 하려면 그 클래스를 싱글톤으로 만들어서 같은 객체만 반환하도록 코드를 작성하면 된다. (리스트 10-38)

#### 빈 등록 메타정보 구성 전략 
1. Xml 단독 사용 : 모든 빈을 xml에서 확인할 수 있지만 개수가 많아지면 관리가 번거로워질 수 있다.
2. Xml 과 빈 스캐닝 혼용
3. 빈 스캐닝 단독 사용 : 전용 태그를 사용할 수 없다. 

### 빈 의존관계 설정 방법
#### xml: <property>, <constructor-arg>
1. 프로퍼티와 생성자로 di를 지정할 수 있다. 
Ex) <property name=“name” value=“”>
value는 타입에 상관없이 작성할 수 있다. 
2. 생성자 사용
Ex) <constructor-arg index=“0” value=“”/>

#### xml 자동 와이어링
1. 이름을 사용한 자동 와이어링 - byName
Ex) <bean id=“hello class=“” autowire=“byName”>
2. 타입을 사용한 자동 와이어링 - byType
Ex) autowire=“byType” 이나 default-autowire=byType” 
단, 타입이 같은 빈이 두개 이상 존재하면 적용되지 못한다. 또, 이름으로 비교하는 것에 비해 타입으로 비교하는 방식은 성능이 느리다. 모든 빈의 프로퍼티에 일괄 적용되기 때문에 개수가 많아지면 모든 빈의 타입과 비교하는 작업이 일어나야 해서 시간이 소요된다.
생성자에 자동와이어링 하는 경우도 있다. Ex) autowire=“constructor”
xml은 가독성이 안좋다. 의존관계를 바로 파악하기 어려울 수 있다. 
하나의 빈에 대해 한 가지 자동와이어링 방식밖에 지정할 수 없다.





