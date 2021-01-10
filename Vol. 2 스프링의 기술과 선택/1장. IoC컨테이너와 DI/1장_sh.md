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

