# 1장 IoC 컨테이너와 DI



스프링 애플리케이션에서는 오브젝트 관리를 컨테이너가 해주고 코드에서 직접 오브젝트를 생성하지 않는다. 코드대신 컨테이너가 오브젝트에 대한 제어권을 갖고 있어서 스프링 컨테이너를 IoC(제어의 역전) 컨테이너라고 부른다. 그리고 이 컨테이너를 스프링에서는 '빈 팩토리' 또는 '애플리케이션 컨텍스트'라고 부르기도 한다. 

<br/>

BeanFactory, ApplicationContext 인터페이스로 정의함 (ApplicationContext는 BeanFactory를 상속한다)

```
public interface ApplicationContext extends ListableBeanFactory, HierarchicalBeanFactory, MessageSource, ApplicationEventPublisher, ResourcePatternResolver { ... }
```

<br/>

스프링 애플리케이션은 최소한 하나 이상의 IoC 컨테이너 즉 애플리케이션 컨텍스트 오브젝트를 갖고 있으며 그 이상을 갖고 있는 경우도 있음
<br/>

#### 컨테이너 동작에 필요한 요소 => POJO 클래스, 설정 메타정보

- POJO 클래스는 독립기능을 수행하게, 확장성고려해서 만들어야 함
- 설정 메타정보는 BeanDifiniton 인터페이스로 정의된 것
  - 빈 아이디, 이름, 별칭
  - 클래스, 클래스이름
  - DI에 사용할 프로퍼티 이름과 값, 참조하는 빈 이름
  - DI에 사용할 생성자 파리미터 이름과 값, 참조할 빈 이름
  - 지연된 로딩 여부, 우선 빈 여부, 자동와이어링 여부, 부모 빈 정보, 빈팩토리 이름 등 
  <br/>
  

### IoC 컨테이너 종류

스프링에는 여러가지 ApplicationContext 구현 클래스가 존재하지만, 이 클래스 오브젝트들을 직접 코드 상에서 생성하는 경우는 거의 없고 대부분 설정을 통해 ApplicationContext가 자동으로 만들어지게 한다. 

<br/>

#### StaticApplicationContext

- 빈 메타정보를 코드를 통해 등록할 때 사용한다. 테스트에서만 사용하는 것이 좋다. 

<br/>

#### GenericApplicationContext

- 실전에서 사용가능하다. 컨테이너 주요 기능을 DI를 통해 확장할 수 있다. 
- 외부 빈 설정 메타정보를 Reader를 통해 읽어서 메타정보로 변환해서 사용한다. 

<br/>

#### GenericXmlApplicationContext

- XmlBeanDefinitionReader를 내장한다. GenericApplicationContext를 사용하는 경우에, XmlBeanDefinitionReader를 직접 생성안하고 한 번에 코드 구현이 가능하다. 

<br/>

#### WebApplicationContext

- 가장 많이 사용된다. 웹 환경에서 필요한 기능이 추가된 컨텍스트이다. 

<br/>
  

