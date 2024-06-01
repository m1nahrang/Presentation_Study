## 핵심개념
#### Dependency Injection
- 의존성 주입
- 객체가 필요로 하는 다른 객체를 직접 생성하는 것이 아니라, 외부에서 생성된 객체를 전달하여 객체 간의 의존성을 관리하는 프로세스
- 객체 간의 결합도를 낮추고 유연하고 재사용 가능한 코드를 작성할 수 있다.

#### Dependency Resolution
- 의존성 해결
- 특정 빈 객체가 필요로 하는 다른 빈 객체를 가져와서 해당 빈 객체의 인스턴스 변수나 세터 메서드를 통해 주입하여 해당 빈 객체가 정상적으로 동작할 수 있도록 한다.

#### Singleton Bean
- Singleton은 스프링 컨테이너에서 사용하는 빈의 생성 방식 중 하나로, 애플리케이션 전역에서 한 번만 인스턴스화되는 빈을 의미한다. 즉, 여러 개의 빈이 같은 인스턴스를 공유하여 사용한다.
- 해당 빈이 처음으로 요청될 때 인스턴스화되고, 이후에는 컨테이너에 의해 생성된 동일한 인스턴스가 계속해서 반환된다.

## Dependency Injection
### 정의
DI란 빈을 생성할 때 의존성이 있는 객체를 스프링이 자동으로 주입해주는 테크닉을 의미한다. IoC의 기본 원리에 걸맞게, 개발자가 코드 내에서 어떤 시점에 의존 객체를 주입할지 명시하지 않는다.
즉, 의존 객체를 통째로 전달받기 때문에 객체는 의존하는 다른 객체를 생성하거나 관리하지 않으므로 대상 객체는 의존 객체에 대해 전혀 알 필요가 없다.
빈의 메타데이터(생성, 의존성 등)는 스프링에서 지원하는 XML, Java Configuration, Annotation 등의 방식으로 설정할 수 있으며 이것들은 자동으로 `BeanDefinition`으로 추상화되어 Spring IoC 컨테이너에서 사용한다.

### 자동 주입 설정
DI 명시를 위해 생성자 기반 또는 Setter기반 설정을 사용할 수 있다.

#### 생성자 기반 DI
```java
@Component
public class MyComponent {
  private final MyDependency dependency;

  @Autowired
  public MyComponent(MyDependency dependency) {
    this.dependency = dependency;
  }
}
```

#### Setter 기반 DI
```java
@Component
public class MyComponent {
  private MyDependency dependency;

  public MyComponent() {
    // 기본 생성자
  }

  @Autowired
  public void setDependency(MyDependency dependency) {
    this.dependency = dependency;
  }
}
```
Setter 기반 DI는 빈 객체의 기본 생성자를 먼저 호출한 뒤에 해당 빈의 세터 메서드를 호출해서 의존성을 주입하는 방식이다.
`MyComponent` 클래스는 `setDependency()`라는 세터 메서드를 가지고 있고, 이 메서드를 통해 `MyDependency` 객체를 주입받는다.   
**생성자 기반과 Setter 기반으로 주입 방식을 굳이 나눠둔 이유는 무엇일까?**    
생성자 기반 DI는 필수적인 의존성을 명확하게 나타낼 수 있고, 런타임에 누락된 의존성을 빨리 감지할 수 있도록 도와준다.
또한 생성자를 호출할 때 검증 로직을 추가할 수 있어서 런타임 오류를 줄일 수 있는 장점이 있다. 반면에 Setter 기반 DI는 필수적이지 않은 선택적인 의존성을 주입할 때 유용하다.   
또한 추후에 재구성이나 재주입이 가능하고, 빈의 의존성이 변경될 가능성이 높은 상황에서 유연성을 제공한다.   
Setter 기반 DI를 사용하면 해당 값이 존재하는지에 대한 확신이 없기 때문에 사용할 때마다 `null`체크를 통해 혹시 모를 오류에 대비해야 하므로 스프링 팀에서는 일반적으로 생성자 기반 DI를 권장한다.

## Dependency Resolution
#### 의존성 해결
컨테이너는 다음과 같은 방식으로 Bean 의존성을 해결한다.

- ApplicationContext는 모든 빈에 대한 configuration metadata를 이용해 빈들을 생성하고 초기화한다.
- 빈이 생성될 때 종속적인 빈이 제공된다.
- 각 프로퍼티나 생성자 인수는 스프링이 자동으로 형변환을 수행한다.

그렇다면 스프링은 복잡한 의존성을 가진 빈들을 어떻게 효율적으로 해결할까? 일반적으로 Spring IoC 컨테이너는 애플리케이션 구동 시점에서 모든 빈의 BeanDefinition을 먼저 읽어들인 뒤 각 빈의 의존성 정보를 분석하여 의존성 그래프를 구성하고, 이를 바탕으로 생성 순서를 정한다.
즉, BeanDefinition을 이용해 메타 정보를 이해하는 것과 실제 빈을 만드는 작업은 동시에 일어나지 않는다는 말이다.
의존성 그래프를 구성하는 방식은 Topological Sort 알고리즘을 사용하는 것이 일반적이라고 한다.
이 알고리즘은 노드 간의 의존 관계가 있는 방향 그래프에서, 모든 노드를 방향성에 거스르지 않도록 나열하는 알고리즘이다.
Spring IoC 컨테이너에서도 이와 유사한 알고리즘이 사용되며 빈들의 의존성 그래프를 구성한 다음, 생성 순서를 정하게 된다.
이렇게 생성된 빈은 **해당 빈이 필요한(호출되는) 런타임에서** 생성된 순서대로 초기화된다.

스프링이 Bean 의존성을 해결하는 방법을 떠올리며 실제로 스프링이 구동될 때 어떤 식으로 작동하는지 짚고 넘어가도록 하자.
1. 빌드 타임
   1. 애플리케이션을 빌드하고 빌드 결과물을 생성하는 시점
   2. 스프링은 XML 파일이나 어노테이션 등을 이용해 설정 정보를 읽어들여 BeanDefinition을 생성한다.
2. 런타임 전반
   1. ApplicationContext가 초기화되고 빈이 생성되는 시점
   2. 스프링은 컨테이너를 생성하고 싱글톤 빈 인스턴스들을 등록, 생성, 초기화하는 작업을 수행한다.
   3. 이때 컨테이너에 등록되는 빈들은 애플리케이션 전체에서 공유된다.
3. 런타임 후반
   1. 런타임 중 애플리케이션이 빈을 요청하고 의존성을 해결하는 시점
   2. 컨테이너에 등록된 빈들을 사용하게 되는 시점
   3. 빈을 사용하는 코드에서는 컨테이너에서 해당 빈을 가져와서 사용하며, 빈들간의 의존성도 런타임 시점에서 해결된다. 즉, 빈이 필요하여 가져오는 시점에 의존성 주입을 진행한다. -> 이 시점에서 의존 객체가 준비되어 있지 않으면 런타임 에러가 발생할 수 있다.
   4. 처음 호출되어 정상적으로 의존성 주입을 마치면 이후 호출에서는 의존성 해결이 된 상태로 남아있다.

#### Lazy-initialized Beans
위의 2. 런타임 전반을 보면 스프링은 ApplicationContext 초기화와 함께 빈을 일단 생성한단 것을 알 수 있다. 물론 의존성 주입은 실제로 빈이 필요하여 호출되는 시점에 진행하지만 일단 만들어 둔다.
Lazy-initialized. 즉 지연 초기화를 사용하면 런타임 전반에 빈이 생성되지 않고 빈을 필요로 하여 호출하는 시점에서 인스턴스가 생성되며 의존성 주입 또한 함께 일어난다.   
ex. Annotation 기반
```java
@Component
@Lazy
public class MyComponent {
  //...
}
```

ex. Java 기반
```java
@Configuration
public class AppConfig {
  @Bean
  @Lazy
  public MyComponent myComponent() {
    return new MyComponent();
  }
}
```
이미 빈을 호출할 때 의존성 주입을 함으로써 성능적인 이점을 가져가고 있는 스프링인데 왜 지연 초기화 같은 것을 사용하는 것일까?

Lazy-initialized를 통해 얻을 수 있는 장점은 두 가지로 요약할 수 있다. 첫째로, 애플리케이션 시작 시간을 단축시킬 수 있다.
모든 빈을 미리 생성하는 것이 아니라 필요한 빈만 생성하므로 애플리케이션 구동 속도가 빨라진다. 
둘째로, 자원을 효율적으로 관리할 수 있다. 자원이 많이 필요한 빈이나 DB 커넥션과 같은 경우, 생성자를 통해 필요한 시점에 빈을 생성하면 자원을 효율적으로 활용할 수 있다.
필요하지 않은 경우에는 빈을 생성하지 않기 때문에 자원 낭비를 줄일 수 있다.

당연히 단점도 존재하는데 우선 빈을 사용하기 직전에 생성되므로, 애플리케이션의 성능이 느려질 수 있다. 특히, 매번 빈을 사용할 때마다 생성하는 경우에는 오버헤드가 크다.
또한 빈이 필요할 때마다 생성하므로, 빈이 생성되는 시점이 예측하기 어려울 수 있다.

이렇듯 Lazy-initialized는 자원을 효율적으로 관리하고 애플리케이션 시작 시간을 단축시킬 수 있는 장점이 있지만, 성능 저하와 예측 어려움이라는 단점도 고려해야 한다.
자원이 많이 필요한 빈, 예를 들면 DB 커넥션과 같은 경우에는 자원 절약 효과를 높이기 위해 lazy-initialized를 사용하는 것을 추천하며 애플리케이션 구동 시간을 줄이기 위해 모든 빈을 한 번에 생성하지 않고 필요한 빈만 생성하는 경우에도 사용할 수 있을 것이다.

lazy-initialized를 사용하는 이유는 대부분의 애플리케이션에서 필요하지 않은 빈들까지 초기화하는 비용을 줄이기 위해서이지만 **대부분의 빈을 미리 생성하는 것이 좋다.**
해당 빈이 사용되는 시점에서 생성되므로 런타임 오류가 발생할 가능성이 커지고 런타임에서 해당 의존성이 해결되기 전까지 다른 빈의 생성 및 초기화 작업도 지연될 가능성이 있기 때문이다.

## 맺음말
스프링의 핵심 개념인 DI(Dependency Injection)에 대해 다뤄보았다.

스프링에서는 IoC 컨테이너가 개발자 대신 객체의 생명주기와 의존성 관리를 담당하며 DI는 객체 간의 의존성을 명시적으로 정의하고, IoC 컨테이너가 필요한 의존성을 객체에 주입하는 것을 의미한다. 이를 통해 객체 간의 결합도를 낮추고 유연성과 재사용성을 높일 수 있다.

두 가지 개념은 서로 보완적이며, 스프링 프레임워크는 IoC 컨테이너를 통해 DI를 구현하여 객체의 생명주기와 의존성 관리를 편리하게 지원한다.

Ref
[의존 관계 주입 핵심 정리]([https://velog.io/@think2wice/Spring-%EC%9D%98%EC%A1%B4%EC%84%B1-%EC%A3%BC%EC%9E%85DI%EC%97%90-%EB%8C%80%ED%95%98%EC%97%AC](https://backendcode.tistory.com/249))   
[Spring docs](https://docs.spring.io/spring-framework/reference/core/beans/dependencies/factory-collaborators.html)   