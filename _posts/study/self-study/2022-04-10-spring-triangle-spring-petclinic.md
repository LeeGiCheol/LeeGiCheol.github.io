---
title : "Spring Triangle Pet-Clinic으로 공부하기"
categories : 
    - spring
tag :
    - spring
toc : true
---

[인프런 백기선-스프링 프레임워크 입문을 공부한 내용입니다.](https://www.inflearn.com/course/spring/dashboard)

[Spring Pet-Clinic Git](https://github.com/spring-projects/spring-petclinic)

---

#### **Pet Clinic 소개**
Spring에서 만든 굉장히 단순한 CURD 프로젝트이다.  

단순하지만 Owner, Pet, Visit, Vet  
총 4개의 Domain으로 구성되어 있으며,  

기본적인 스프링 문법인 @Controller, @RequestMapping 등 부터 시작해서 AOP, Validation, 다양한 테스트 그리고 Spring Data JPA의 연관관계까지도 잘 표현한 프로젝트이다.  

인프런에 무료로 공개된 백기선님의 스프링 프레임워크 입문 강의를 보면서  

Spring Triangle이라 불리는 스프링의 3대 요소인 IoC, AOP, PSA를 공부하려고 한다.  

---

#### **Inversion Of Control (IoC)란 무엇인가**  
IoC는 의존성에 대한 제어권을 역전시킨다는 의미이다.  

의존성의 제어권을 역전시킨다는것이 무슨 말일까?  

일반적으로 의존성에 대한 제어권은 자기 자신이 가진다.  

```java
public class OwnerController {
    private OwnerRepository repository = new OwnerRepository();
}
```

그러나 자기 자신이 의존성을 가진다면 유연성이 떨어질 수 밖에 없다.  

**만약 OwnerRepository가 새로 생겨 교체가 필요하다면?  
혹은 테스트용 OwnerRepository가 따로 있다면?**  

위 처럼 직접 객체를 만들어 사용하는 것은 충분히 불편한 일이다.  

그렇기 때문에 스프링은 의존성의 제어권을 역전시켜 IoC 컨테이너 (혹은 스프링 컨테이너)에서 가지도록 한다.  

```java
@Controller
public class OwnerController {
    private final OwnerRepository ownerRepository;

    public OwnerController(OwnerRepository ownerRepository) {
        this.ownerRepository = ownerRepository;
    }    
}
```

스프링의 IoC 기능을 활용한다면 위 코드는 내가 직접 OwnerRepository 객체를 만들고 OwnerController의 생성자를 만들지 않지만 null이 아니며,  
누군가 즉 IoC 컨테이너가 의존성을 **주입** 해주는 것을 기대할 수 있다.   

---

#### **IoC 컨테이너**
IoC 컨테이너는 스프링 빈을 만들고 엮어주며, 제공해주는 역할을 한다.  

org.springframework.context 패키지의 **ApplicationContext** 인터페이스가 바로 IoC 컨테이너이다.  

스프링 빈으로 등록이 된다면 IoC 컨테이너가 객체를 만들고 의존성을 관리하게 된다.  

그렇기 때문에 위에서 봤던 코드의 생성자에 OwnerRepository 객체가 null이 아닌 것이다.  

```java
@Controller
public class OwnerController {
    private final OwnerRepository ownerRepository;

    public OwnerController(OwnerRepository ownerRepository) {
        this.ownerRepository = ownerRepository;
    }    
}
```


예전엔 보통 스프링빈을 xml에 설정하는 방식이었지만, 요새는 자바 설정, 스프링 부트를 사용한다면 내장 된 기본설정을 사용한다.  

따라서 ApplicationContext는 이제 특수한 상황이 아닌 이상 쓸 일이 거의 없어졌다.  

---

#### **Spring Bean**  
스프링 빈이란 IoC 컨테이너가 관리하는 객체이다.  

그렇다면 스프링 빈은 어떻게 등록할까?  

xml 설정과, 자바 설정, Component Scan을 하는 방법이 있다.  

1. xml 설정 

```xml
<bean id="ownerRepository" class="org.springframework.samples.petclinic.owner.OwnerRepository" />

<bean id="ownerController" class="org.springframework.samples.petclinic.owner.OwnerController">
    <constructor-arg ref="ownerRepository"/>
</bean>
```

2. 자바 설정
**@Configuration**이 붙은 클래스는 이 클래스가 xml 설정을 대체한다는 의미이다.  
이러한 클래스에서 **@Bean**을 사용하여 메서드를 만든다면, 이 메서드에서 리턴되는 클래스를 빈으로 등록해준다.  

3. Component Scan
Component Scan 또한 xml, 자바설정 모두 가능하다.

- xml 설정

```xml
<context:component-scan base-package="me.gicheol.scan"/> 
```

root-context.xml과 같은 xml 파일에 ComponentScan을 설정해주면  
base package를 기준으로 클래스를 스캔하여 빈을 등록한다.  
이때 base package는 여러개의 패키지를 등록할 수 있다.  

```xml
<context:component-scan base-package="me.gicheol.scan, me.gicheol.component"/> 
```

- 자바 설정

```java
@Configuration
@ComponentScan(basePackages = "me.gicheol.scan")
public class AppConfig { }
```

**@Configuration**은 xml을 대체할 수 있는 설정 클래스임을 알린다.  
**@ComponentScan**을 통해 basePackages를 설정한다.  

ComponentScan은 **BeanFactoryPostProcessor**를 구현한 **ConfigurationClassPostProcessor**에 의해 동작한다.  

BeanFactoryPostProcessor는 다른 빈을 만들기 전에 BeanFactoryPostProcessor의 구현체의 빈을 먼저 생성한다.  

즉, 다른 Bean들을 등록하기 전에 컴포넌트 스캔을해서 Bean으로 등록해준다.


여기서 Scan의 기준은 **@Component** 애노테이션이 붙은 클래스이다.  

@Controller  
@Service  
@Repository  
@Configuration  

위 4개의 애노테이션도 내부적으론 @Component이기 때문에 빈으로 등록되게 된다.  

![error](/assets/images/self-study/2022-04-10/component-scan.png)  

스프링 부트의 경우 메인 메서드가 있는 최상단 클래스를 기준으로 Component Scan이 되는데,  

이 클래스에는  **@SpringBootApplication** 애노테이션이 붙어있다.  
@SpringBootApplication 의 내부적으론 @ComponentScan을 사용하는 것을 알 수 있다.  

![error](/assets/images/self-study/2022-04-10/spring-boot-component-scan.png)  


---

#### **Dependency Injection**  

**@Autowired**를 통해 의존성을 주입할 수 있다.  

이때 @Autowired는 어디에 붙일 수 있을까?  

- **생성자**  

```java
@Controller
public class OwnerController {

    private final OwnerRepository ownerRepository;

    @Auwotired
    public OwnerController(OwnerRepository ownerRepository) {
        this.ownerRepository = ownerRepository;
    }
}
```

생성자 주입 방식은 스프링에서 가장 권장하는 방법이다.  

그렇기 때문에 @Autowired를 생략하고 생성자 주입을 할 수 있는 방법도 제공한다.  

빈으로 등록된 클래스에 생성자가 하나만 있고, 그 생성자의 매개변수 또한 빈으로 등록된 클래스라면 @Autowired를 생략하더라도 생성자의 매개변수를 주입해준다.  


- **필드**  

```java
@Controller
public class OwnerController {

    @Autowired
    private OwnerRepository ownerRepository;

}
```

- **생성자**  

```java
@Controller
public class OwnerController {


    private OwnerRepository ownerRepository;

    @Autowired
    public void setOwnerRepository(OwnerRepository ownerRepository) {
        this.ownerRepository = ownerRepository;
    }

}
```

---

#### **Dependency Injection은 어디에 붙이는게 가장 좋을까?**  
스프링에서는 생성자 주입을 가장 권장한다.  

그 이유를 알아보자.  

**1. 객체의 불변성 확보**
setter 주입을 할 경우 객체 생성 이후 불필요한 수정이 발생될 수 있다.  
이것은 OCP를 위반하게 되는데, 어쩌면 굉장히 위험할 수 있다.  
**그러므로 생성자 주입을 통해 변경의 가능성 자체를 배제하고 객체의 불변성을 보장하도록 한다**.  

**2. 테스트 코드의 용이성**
만약 필드 주입을 사용했다면 순수 자바 코드의 단위 테스트는 불가능하다.  

```java
@Service
public class SampleService {
    
    @Autowired
    private SampleRepository sampleRepository;

    public void addSample(Sample sample) {
        sampleRepository.save(sample);
    }

}
```

```java
public class SampleServiceTest {
    
    @Test
    void addTest() {
        SampleService sampleService = new SampleService();
        sampleService.addSample(new Sample());
    }

}
```

이 코드는 SampleRepository를 주입받지 못해 NPE가 발생한다.  

그렇다면 생성자 주입은 어떨까?  

```java
@Service
public class SampleService {
    
    private final SampleRepository sampleRepository;

    public SampleService(SampleRepository sampleRepository) {
        this.sampleRepository = sampleRepository;
    }

    public void addSample(Sample sample) {
        sampleRepository.save(sample);
    }

}
```

```java
public class SampleServiceTest {
    
    @Test
    void addTest() {
        SampleRepository sampleRepository = new SampleRepository();
        SampleService sampleService = new SampleService(sampleRepository);
        sampleService.addSample(new Sample());
    }

}
```

만약 주입되어야 할 객체가 누락되었다면 컴파일 시점에 미리 오류를 발견할 수 있고,  
테스트를 위한 객체를 생성자에 넣어서 테스트를 할 수도 있다.  

**3. final 키워드 작성 및 Lombok과의 결합** 
생성자 주입을 제외한 나머지 방식은 객체의 생성 이후에 주입이 되기 때문에 필드에 final 키워드를 사용할 수없다.  
final을 사용함으로써 컴파일 시점 누락된 의존성을 확인할 수 있다.  

그리고 Lombok은 생성자를 만들어 줄때 final이 붙은 필드를 매개변수로써 사용할 수 있는 @RequiredArgsConstructor를 제공한다.  

사실 개발하면서 객체의 의존성이 추가되거나 변경 또는 제거 되는 일은 상당히 많다.  
이때마다 생성자를 지우고, 다시 만드는 과정은 상당히 귀찮다.  

이 애노테이션을 사용하면 생성자를 직접 만들지 않기 때문에 간편하고 깔끔한 코딩을 할 수 있다.  

```java
@Service
@RequiredArgsConstructor
public class SampleService {
    
    private final SampleRepository sampleRepository;

    public void addSample(Sample sample) {
        sampleRepository.save(sample);
    }

}
```

**4. 순환참조 에러 방지**

```java
@Service
public class SampleService {
    @Autowired
    private SimpleService simpleService;

    public void addSample(Sample sample) {
        simpleService.add(sample);
    }
}
```

```java
@Service
public class SimpleService {
    @Autowired
    private SampleService sampleService;

    public void add(Sample sample) {
        sampleService.addSample(sample);
    }
}
```

순환참조는 끔찍한 일이다.  
두 메서드는 서로를 바라보며 계속 호출을 하고, CallStack이 계속 쌓이다 결국에는 StackOverflow 에러가 발생한다.  
게다가 이러한 에러는 메서드가 실행되어야만 발생한다.  

반면 생성자 주입을 사용할 경우 애플리케이션 실행 시점에 객체를 생성하면서 아래와 같은 에러가 발생한다.  

![error](/assets/images/self-study/2022-04-10/circular-reference.png)  


**결론**
- 객체의 불변성을 확보한다.  
- 테스트 코드 작성이 용이하다.  
- 필드에 final 키워드를 사용할 수 있고, Lombok과 결합으로 간결한 코드 작성이 가능하다.  
- 순환참조 문제를 애플리케이션 실행 시점에 파악할 수 있다.  

이러한 네 가지 장점으로 생성자 주입을 가장 권장한다.  

---

#### **Aspect Oriented Programming**
AOP는 흩어진 코드를 한 곳으로 모으는 기법이다.  

보통 로깅을 하거나, Transaction을 사용할 때, 성능측정, 인증 등에 AOP를 사용하게 된다.  


```java
class A {
    method a() {
        AAAA
        ...business logic...
        BBBB
    }

    method b() {
        AAAA
        ...another business logic...
        BBBB
    }
}

class B {
    method c() {
        AAAA
        ...something logic...
        BBBB
    }

}
```

만약 이러한 의사코드가 있다고 가정하자.  

AAAA와 BBBB는 공통으로 계속 등장하지만 중간에 무언가의 로직이 있어 분리하기가 참 모호하다.  

이럴 때 AOP를 사용한다면 유용하게 쓸 수 있다.  

AOP는 바이트코드를 조작하는 방법과, 프록시 패턴을 사용하는 방법이 있다.  

스프링에서는 프록시 패턴을 사용하고 있다.  

먼저 흩어진 공통 코드를 한 곳으로 모아준다.  

```java
class A {
    method a() {
        ...business logic...
    }

    method b() {
        ...another business logic...
    }
}

class B {
    method c() {
        ...something logic...
    }
}

class AAAABBBB {
    method aaaa() {
        AAAA
    }

    method bbbb() {
        BBBB
    }
}
```

그리고 프록시 패턴을 적용한다.  

```java
class A {
    method a() {
        ...business logic...
    }

    method b() {
        ...another business logic...
    }
}

class AProxy extends A {
    method a() {
        AAAABBBB.AAAA
        ...business logic...
        AAAABBBB.BBBB
    }

    method b() {
        AAAABBBB.AAAA
        ...another business logic...
        AAAABBBB.BBBB
    }
}

class B {
    method c() {
        ...something logic...
    }
}

class BProxy extends B {
    method c() {
        AAAABBBB.AAAA
        ...something logic...
        AAAABBBB.BBBB
    }
}

class AAAABBBB {
    method aaaa() {
        AAAA
    }

    method bbbb() {
        BBBB
    }
}
```

---

#### **AOP를 적용하는 방법**  
애노테이션을 통해 로깅을 하는 AOP 예제를 만들어보자.  

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface LogExecutionTime {
}
```

먼저 AOP의 대상이 될 애노테이션을 만들어준다.  
메소드에 붙여줄 예정이니 Target은 METHOD 레벨로,  
애노테이션은 사실 주석의 역할이기 때문에 런타임에도 사용하기 위해서 Retention은 RUNTIME 레벨로 설정한다.  


```java
@Aspect
@Component
public class LogAspect {

    Logger logger = LoggerFactory.getLogger(LogAspect.class);

    @Around("@annotation(LogExecutionTime)")
    public Object logExecutionTime(ProceedingJoinPoint joinPoint) throws Throwable {
        StopWatch stopWatch = new StopWatch();
        stopWatch.start();

        Object proceed = joinPoint.proceed();

        stopWatch.stop();
        logger.info(stopWatch.prettyPrint());

        return proceed;
    }

}
```

AOP를 사용하기 위해 @Aspect를 사용한다.  
또한 AOP는 Before, After, AfterReturnning, Around 등 다양한 동작 시점을 설정할 수 있지만 예제는 실행 시점과 종료 시점의 실행 시간을 파악해 로깅하는 예제이기 때문에 Around 시점을 사용한다.  

JoinPoint가 바로 실행되어야 할 기존 메서드를 의미한다.  

JoinPoint를 proceed 즉 실행하기 전 StopWatch를 실행하고 proceed 이후 StopWatch를 종료하고 로깅을 한다.  

```java
@RestController
public class SampleController {

    @LogExecutionTime
    @GetMapping("/hello")
    public String hello() {
        return "hello cheeolee";
    }

}
```

```java
2022-04-11 02:12:16.577 DEBUG 21804 --- [nio-8080-exec-1] o.s.web.servlet.DispatcherServlet        : GET "/hello", parameters={}
2022-04-11 02:12:16.580 DEBUG 21804 --- [nio-8080-exec-1] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped to org.springframework.samples.petclinic.owner.SampleController#hello()
2022-04-11 02:12:16.589  INFO 21804 --- [nio-8080-exec-1] o.s.samples.petclinic.aspect.LogAspect   : StopWatch '': running time = 3783625 ns
---------------------------------------------
ns         %     Task name
---------------------------------------------
003783625  100%  
```

---

#### **Portable Service Abstraction**  
환경의 변화와 관계 없이 일관된 방식의 기술로 접근 환경을 제공하는 추상화 구조이다.  

쉽게 말하자면 **잘 만든 인터페이스**이다.  

확장성이 좋지 않거나, 어떤 기술에 특화되어 있는 코드는  
테스트하기 어렵고, 어떤 기술이 바뀌기 때문에 나의 코드도 변경되어야 한다.  

인터페이스가 잘 설계되어있다면 JDBC를 쓰다가 JPA로 변경하더라도 나의 코드는 변경되지 않는다.  

Log4j를 쓰다가 Logback로 변경하더라도 나의 코드는 변경되지 않는다.  

혹은 Spring MVC를 쓰다가 Spring Reactive로 변경하더라도 나의 코드는 변경되지 않는다.  

Pet-Clinic 예제를 통한 PSA를 알아보자.  

---

#### **Spring Transaction**
Pet-Clinic은 트랜잭션 처리를 @Transactional을 통해 하고있다.  

이 애노테이션을 처리할 Aspect가 존재하며 이 Aspect는 트랜잭션 처리를 기술에 독립적인 PlatformTransactionManager 인터페이스를 사용한다.  

그렇기 때문에 PlatformTransactionManager의 구현체가 바뀌더라도 Aspect의 코드는 변경되지 않는다.  

Pet-Clinic 예제는 JPA를 사용하고 있기 때문에 JpaTransactionManager 구현체를 사용하고, 마찬가지로 Aspect의 코드는 변경되지 않는다.   

(트랜잭션을 처리하는 Aspect 코드는 org.springframework.transaction.interceptor.TransactionAspectSupport 이 클래스 인 것 같으나 AOP 관련 코드를 찾을 수가 없었다.)  

---

#### **Spring Cache**
Pet-Clinic에서는 Cache 기능을 사용한다.  

CacheConfiguration 클래스에서는  
스프링 프레임워크의 애노테이션인  
@EnableCaching을 통해 캐시 기능을 사용한다.  

@Cacheable를 통해 캐시를 저장/조회할 수 있고,  
@CachePut을 통해 캐시를 저장하고,
@CacheEvict를 통해 캐시를 제거할 수 있다.  

이러한 기능도 트랜잭션과 마찬가지이다.  

Cache 애노테이션을 처리하는 Aspect가 있으며 그 Aspect에서는 처리하기 위해 CacheManager 인터페이스가 있어야 한다.  
이 CacheManager의 구현체가 바뀌더라도 Aspect의 코드가 변경되지 않는다.  

---

#### **Spring Web MVC**  
Spring Framework5에서 추가된 WebFlux가 있다.  

Tomcat과 Servlet 기반의 Spring Web MVC와 다르게  

Netty와 Reactive Stream Adapter를 사용한다.  

```java
@Controller
public class OwnerController {
    
    private static final String VIEWS_OWNER_CREATE_OR_UPDATE_FORM = "owners/createOrUpdateOwnerForm";

    
    private final OwnerRepository ownerReposioty;

    public OwnerController(OwnerRepository ownerReposioty) {
        this.ownerReposioty = ownerReposioty;
    }

    @GetMapping("/owners/new")
    public String initCreationForm(Map<String, Object> model) {
        Owner owner = new Owner();
        model.put("owner", owner);
        return VIEWS_OWNER_CREATE_OR_UPDATE_FORM;
    }

}
```

과연 이 코드를 보고 Spring Web MVC인지, Spring Web Flux인지 구분할 수 있을까?  

이 또한 PSA라고 볼 수 있다.  

---

#### **정리**  
스프링 프레임워크의 핵심은 IoC, AOP, PSA이다.  

- IoC는 직접 객체를 생성해 의존성을 만드는 것이 아닌, 스프링을 통해 의존성을 주입받는 형태로 의존성의 제어권이 개발자에서 스프링으로 역전되는 것을 제공한다.  

- AOP는 흩어진 코드를 한 곳으로 모아 비즈니스 로직에서 분리하고 재사용한다.  

- 잘 만들어진 인터페이스를 사용하면 구현체가 바뀌더라도 나의 코드는 변경되지 않는다.  
스프링 프레임워크가 제공하는 대부분의 API는 PSA이다.

---

#### **참고자료**

[Component Scan - [스프링 핵심기술] - @Component와 @ComponentScan - 짱호](https://jjingho.tistory.com/9)    

[생성자 주입을 사용해야 하는 이유 - [Spring] 다양한 의존성 주입 방법과 생성자 주입을 사용해야 하는 이유 - (2/2) - 망나니개발자](https://mangkyu.tistory.com/125)  

[PSA - [Spring] PSA(Portable Service Abstraction)란?](https://dev-coco.tistory.com/83)    


---