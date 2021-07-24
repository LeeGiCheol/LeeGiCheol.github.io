---
title : "JPA 연관관계 무한루프"
categories : 
    - jpa
tag :
    - jpa
toc : true
---

[김영한 : 실전! 스프링 부트와 JPA 활용2 - API 개발과 성능 최적화](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8-JPA-API%EA%B0%9C%EB%B0%9C-%EC%84%B1%EB%8A%A5%EC%B5%9C%EC%A0%81%ED%99%94) 강의를 공부한 내용입니다.  

[예제 Github](https://github.com/LeeGiCheol/jpa-problem-demo)

---

#### **JPA RestAPI 양방향 연관관계 주의사항**  
양방향 연관관계를 가진 엔티티를 조회할때 주의사항이 있다.  

먼저 Users와 Orders 엔티티가 있다.  

```java
@Entity
@Getter @Setter
public class Users {

    @Id @GeneratedValue
    private Long id;

    @Column(name = "USER_NAME")
    private String name;


    @OneToMany(mappedBy = "users")
    private List<Orders> orders = new ArrayList<>();

}

@Entity
@Getter @Setter
public class Orders {

    @Id @GeneratedValue
    private Long id;

    @Column(name = "ORDER_NAME")
    private String name;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "USER_ID")
    private Users users;

    public void setUsers(Users users) {
        this.users = users;
        users.getOrders().add(this);
    }

}
```

Orders 엔티티는 USER_ID 컬럼을 FK로 가지며,  
연관관계 편의 메서드를 만들어두었다.  

```java
@RestController
@RequiredArgsConstructor
public class OrdersController {

    private final OrdersService ordersService;
    private final UsersService usersService;

    @PostMapping("/saveOrders")
    public ResponseEntity<?> saveItem(@RequestBody SaveOrdersRequestDto request) {
        Users users = usersService.findById(request.getUserId());
        ordersService.saveOrders(users, request);

        return ResponseEntity.ok(request.getName());
    }

    @PostMapping("/findAllOrders")
    public ResponseEntity<?> findAll() {
        List<Orders> orders = ordersService.findAll();
        return ResponseEntity.ok(orders);
    }
    
}
```

```java
@Service
@RequiredArgsConstructor
public class OrdersService {

    private final OrdersRepository ordersRepository;
    private final ModelMapper modelMapper;

    public void saveOrders(Users users, SaveOrdersRequestDto request) {
        Orders item = modelMapper.map(request, Orders.class);
        item.setUsers(users);
        ordersRepository.save(item);
    }


    public List<Orders> findAll() {
        return ordersRepository.findAll();
    }

}
```

```java
@Data
public class SaveOrdersRequestDto {

    private Long userId;
    private String name;

}
```

주문등록을 잘 마치고, 주문조회를 하려고 하면 상당한 에러가 발생한다.  

```java
2021-07-23 23:57:20.033 ERROR 7062 --- [nio-8080-exec-7] o.a.c.c.C.[.[.[/].[dispatcherServlet]    : Servlet.service() for servlet [dispatcherServlet] in context with path [] threw exception [Request processing failed; nested exception is org.springframework.http.converter.HttpMessageNotWritableException: Could not write JSON: Infinite recursion (StackOverflowError); nested exception is com.fasterxml.jackson.databind.JsonMappingException: Infinite recursion (StackOverflowError) (through reference chain: com.example.jparest.test.domain.Users$HibernateProxy$7ThLi873["item"]->org.hibernate.collection.internal.PersistentBag[0]->com.example.jparest.test.domain.Item["users"]->com.example.jparest.test.domain.Users$HibernateProxy$7ThLi873["item"]->org.hibernate.collection.internal.PersistentBag[0]->com.example.jparest.test.domain.Item["users"]->com.example.jparest.test.domain.Users$HibernateProxy$7ThLi873["item"]............
```

![error](/assets/images/kyh-jpa-basic/37-stack-over-flow.png)    

사진은 포스트맨 응답 결과이다.  
ResponseEntity에 ok로 응답을 보냈으니, status는 200이지만,  
무한루프가 발생해 어마어마한 응답값을 볼 수 있다.  

발생하는 이유는 간단하다.  
@ResponseBody가 붙은 HandlerAdapter 즉 컨트롤러 메서드는  
응답값으로 Json을 반환한다.  
이때 Jackson 라이브러리가 사용되는데,  
Jackson 라이브러리는 orders를 반환하기 위해 파싱을 시도한다.  

그런데 Orders 엔티티에 가보면, Users 엔티티가 있고,  
또 Users 엔티티에 가보면, List\<Orders\> 엔티티가 있다.  

@ToString 때문에 Lombok을 주의해서 사용해야 된다는 것 처럼  
여기서 무한 루프가 발생한다.  

---

##### **해결방법. 1**  
양방향 연관관계의 엔티티 두개 중 한 곳에  
@JsonIgnore 애노테이션을 붙인다.  

```java
@Entity
@Getter @Setter
public class Users {

    @Id @GeneratedValue
    private Long id;

    @Column(name = "USER_NAME")
    private String name;


    @JsonIgnore
    @OneToMany(mappedBy = "users")
    private List<Orders> orders = new ArrayList<>();

}
```

자 이제 무한루프는 해결되었다.  
그렇지만 에러는 아직도 발생할 것이다.  

```java
com.fasterxml.jackson.databind.exc.InvalidDefinitionException: No serializer found for class org.hibernate.proxy.pojo.bytebuddy.ByteBuddyInterceptor and no properties discovered to create BeanSerializer (to avoid exception, disable SerializationFeature.FAIL_ON_EMPTY_BEANS) (through reference chain: java.util.ArrayList[0]->com.example.jparest.test.domain.Orders["users"]->com.example.jparest.test.domain.Users$HibernateProxy$nEHUKqGy["hibernateLazyInitializer"])
```

에러명은 InvalidDefinitionException  
여전히 jackson의 파싱이 문제이다.  

내용을 대략적으로 해석하자면 ByteBuddyInterceptor 라는 타입을 알 수 없다고 한다.  
이게 무슨 내용이냐면, JPA는 LazyLoading 설정이 되어있다면  
해당 엔티티를 조회할 때 DB를 통해 값을 조회하지 않고,  
프록시 객체를 만들어 둔 후, 그 엔티티를 사용할 때 DB에 조회를 한다.  

이러한 프록시 객체를 만들어주는 라이브러리는  
바이트코드를 조작할 수 있는 ByteBuddy를 사용한다.  

안타깝게도 프록시를 어떤식으로 만들어주는지는  
LazyLoading이 설정된 엔티티, xToMany, FetchType의   
바이트코드를 까봐도 해당 내용을 찾을 수 없었다.  

대략 Users는 아래와 같은 모양새로 만들어질 것이다.  

```java
@Entity
@Getter @Setter
public class Orders {

    ...

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "USER_ID")
    private Users users = new ByteBuddyInterceptor();

    ...

}
```

근데 이 상황에서 Jackson이 알수없는 ByteBuddyInterceptor가 있으니 발생한 에러이다.  

이 문제를 해결하기 위해선 Hibernate5Module 의존성을 추가해줘야한다.  

```xml
<!-- https://mvnrepository.com/artifact/com.fasterxml.jackson.datatype/jackson-datatype-hibernate5 -->
<dependency>
    <groupId>com.fasterxml.jackson.datatype</groupId>
    <artifactId>jackson-datatype-hibernate5</artifactId>
    <version>2.12.4</version>
</dependency>
```

```groovy
// https://mvnrepository.com/artifact/com.fasterxml.jackson.datatype/jackson-datatype-hibernate5
implementation 'com.fasterxml.jackson.datatype:jackson-datatype-hibernate5:2.12.4'
```

버전은 여기서 확인할 수 있다.  

[Maven Repository](https://mvnrepository.com/artifact/com.fasterxml.jackson.datatype/jackson-datatype-hibernate5)

의존성을 추가했다면 빈 설정을 해줘야한다.  

```java
@Bean
public Hibernate5Module hibernate5Module() {
    return new Hibernate5Module();
}
```

이제서야 정상적으로 동작할 것이다.  

---

##### **해결방법. 2**
위 방법은 아직 문제가 많다.  
Lazy Loading이 설정된 필드는 조회 하지 못해 null을 반환한다.  

빈 설정 할때 Lazy Loading이 설정된 필드도 모두 조회할 수 있도록 추가해준다.   

```java
@Bean
public Hibernate5Module hibernate5Module() {
    Hibernate5Module hibernate5Module = new Hibernate5Module();
    hibernate5Module.configure(Hibernate5Module.Feature.FORCE_LAZY_LOADING, true);

    return hibernate5Module;
}
```

이제 모든 값을 다 가져올 수 있게 되었다.  

대신 발생하는 문제는 쿼리가 어마어마하게 발생한다.  
마치 Eager Loading을 사용하는 것처럼 말이다.  
사실상 성능 최적화는 포기해야된다.  

---

##### **해결방법. 3**
이런 이유 때문에 Lazy Loading을 포기하고,  
Eager Loading을 사용하는 건 매우 위험하다.  

빈 설정에서 강제 레이지 로딩까지 조회하는 옵션을 제거하고  
강제로 초기화 시키는 방법이 있다.  

```java
@PostMapping("/findAllOrders")
public ResponseEntity<?> findAll() {
    List<Orders> orders = ordersService.findAll();

    for (Orders order : orders) {
        order.getUsers().getName();
    }
    
    return ResponseEntity.ok(orders);
}
```

이렇게 구현하면 Lazy Loading을 유지한채로  
필요한 객체만 추가로 조회할 수 있다.  

이 방법도 사실 쿼리가 많이 발생하고, 코드가 지저분해져서 별로이다.  

---

##### **해결방법. 4**  
사실 API 개발에서 엔티티를 직접 Request로 받고,  
Response로 응답하는 것은 좋지 못한 설계이다.  
Validation을 하기도 어려울 뿐더러  
컬럼의 이름이 변하면 API 규격이 변경되기 때문이다.  

장황하게 글을 작성했지만 꼭 DTO를 만들어서 설계하도록하자.  
위의 모든 문제는 DTO를 만드는 것으로 가장 쉽고 간결하게 해결된다.  

---

#### **다음으로**  
아직 문제가 완벽히 해결되지 않았다.  
N+1 문제가 발생할 수 있는데, 이를 해결하는 것은 다음 글에서 공부해보자.  
[JPA N+1 문제 해결하기 >>](https://leegicheol.github.io/jpa/jpa-n+1-problem/)

---