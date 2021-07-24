---
title : "JPA N+1 문제 해결하기"
categories : 
    - jpa
tag :
    - jpa
toc : true
---

[김영한 : 실전! 스프링 부트와 JPA 활용2 - API 개발과 성능 최적화](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8-JPA-API%EA%B0%9C%EB%B0%9C-%EC%84%B1%EB%8A%A5%EC%B5%9C%EC%A0%81%ED%99%94) 강의를 공부한 내용입니다.  

[예제 Github](https://github.com/LeeGiCheol/jpa-problem-demo)

---

#### **JPA N+1 문제 해결하기**  
이전에 양방향 연관관계 조회 시 무한루프가 발생하는 문제를 해결해보았다.  
DTO를 사용하는 방법으로 간단하게 해결이 됬지만,  
N+1 문제가 일어나기 딱 좋은 상태이다.  

이전에 살펴봤던 Users, Orders 엔티티를 예시로 들겠다.  

```java
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
```

양방향 연관관계로 묶인 두 엔티티가 있다.  
Orders를 조회할 때 문제가 발생할 수 있는데,  
N+1 문제란 처음 조회한 엔티티에 연관관계가 Lazy Loading으로 설정되어 있을 경우  
연관관계가 있는 엔티티를 추가적으로 조회하기 위해 발생하는 쿼리를 의미한다.  

만약 첫 번째로 조회한 엔티티의 개수가 5개라면  
엔티티와 연관관계가 있는 엔티티가 추가적으로 5번 쿼리를 하게된다.  
5+1이 되어 총 6개의 쿼리가 발생하는 것이다.  
(지금은 연관관계가 한개이지만, 두개라면 5+5+1으로 총 11번 쿼리한다.)  

그나마 영속성 컨텍스트에서 관리되는 엔티티라면 쿼리가 나가지 않으니  
최악의 경우만 N+1이지만, 굉장히 위험한 문제이다.  
한번만 쿼리해도 되는 것을 6번이나 쿼리해야 되니 성능 문제의 주범으로 보인다.  

---

##### **N+1 예제**  
먼저 예제를 보기전에 회원은 5명이 존재하며,  
5명의 회원이 모두 한개씩 주문을 했다는 가정하에  
전체 주문을 조회하는 예제이다.  

```java
@RestController
@RequiredArgsConstructor
public class OrdersController {

    private final OrdersService ordersService;

    @PostMapping("/findAllOrders")
    public ResponseEntity<?> findAll() {
        List<Orders> orders = ordersService.findAll();
 
        List<FindOrdersResponseDto> orderDto = orders.stream()
                .map(FindOrdersResponseDto::new)
                .collect(Collectors.toList());

        return ResponseEntity.ok(orderDto);
    }
}
```

```java
@Service
@RequiredArgsConstructor
public class OrdersService {

    private final OrdersRepository ordersRepository;
  
    public List<Orders> findAll() {
        return ordersRepository.findAll();
    }

}
```

```java
@Data
public class FindOrdersResponseDto {
    public FindOrdersResponseDto(Orders order) {
        this.orderId = order.getId();
        this.name = order.getName();
        this.users = order.getUsers();
    }

    private Long orderId;
    private String name;
    private Users users;
    
}
```

DTO를 리턴해서 무한루프가 해결됬지만 N+1 문제가 말썽이다.  
우선 쿼리부터 확인해보자..  

```sql
hibernate.SQL                        : 
    select
        orders0_.id as id1_0_,
        orders0_.order_name as order_na2_0_,
        orders0_.user_id as user_id3_0_ 
    from
        orders orders0_


hibernate.SQL                        : 
    select
        users0_.id as id1_1_0_,
        users0_.user_name as user_nam2_1_0_ 
    from
        users users0_ 
    where
        users0_.id=?

hibernate.SQL                        : 
    select
        users0_.id as id1_1_0_,
        users0_.user_name as user_nam2_1_0_ 
    from
        users users0_ 
    where
        users0_.id=?

hibernate.SQL                        : 
    select
        users0_.id as id1_1_0_,
        users0_.user_name as user_nam2_1_0_ 
    from
        users users0_ 
    where
        users0_.id=?

hibernate.SQL                        : 
    select
        users0_.id as id1_1_0_,
        users0_.user_name as user_nam2_1_0_ 
    from
        users users0_ 
    where
        users0_.id=?

hibernate.SQL                        : 
    select
        users0_.id as id1_1_0_,
        users0_.user_name as user_nam2_1_0_ 
    from
        users users0_ 
    where
        users0_.id=?
```

미쳤다...  

맨 처음 Orders를 조회했을 때  
5개의 주문이 있으니 5개의 값이 나왔을 것이다.  
그런데 각각 Orders는 Users를 가지는데,  
Lazy Loading이기 때문에  
Orders 조회 이후에 필요에 따라 Users를 조회한다.  

근데 지금은 전체 조회이므로  
WHERE절에 각각 Orders에 매핑된 Users의 ID를 조건으로 조회를 한다.  

총 6번의 조회 쿼리가 발생한다.  

이는 Eager Loading으로 변경한다 한들 해결되지 않는다.  
오히려 더 이상한 쿼리가 발생하게된다.  

---

##### **해결방법. 1**  
해결방법은 Fetch Join을 사용하는 것이다.  

```java
@RestController
@RequiredArgsConstructor
public class OrdersController {

    private final OrdersService ordersService;

    @PostMapping("/findAllOrders/fetchJoin")
    public ResponseEntity<?> findAllFetchJoin() {
        List<Orders> orders = ordersService.findAllFetchJoin();

        List<FindOrdersResponseDto> orderDto = orders.stream()
                .map(FindOrdersResponseDto::new)
                .collect(Collectors.toList());

        return ResponseEntity.ok(orderDto);
    }
}
```

```java
@Service
@RequiredArgsConstructor
public class OrdersService {

    private final OrdersRepository ordersRepository;

    public List<Orders> findAllFetchJoin() {
        return ordersRepository.findAllFetchJoin();
    }

}
```

```java
@Repository
@RequiredArgsConstructor
public class OrdersRepository {

    private final EntityManager em;

    public List<Orders> findAllFetchJoin() {
        return em.createQuery(
                    "SELECT o FROM Orders o " +
                    "JOIN FETCH o.users u", Orders.class)
                .getResultList();
    }

}
```

```java
@Data
public class FindOrdersResponseDto {
    
    public FindOrdersResponseDto(Orders order) {
        this.orderId = order.getId();
        this.name = order.getName();
        this.users = new FindUsersResponseDto(order.getUsers());
    }

    private Long orderId;
    private String name;
    private FindUsersResponseDto users;

}

@Data
public class FindUsersResponseDto {
 
    public FindUsersResponseDto(Users users) {
        this.id = users.getId();
        this.name = users.getName();
    }

    private Long id;
    private String name;

}
```

이게 끝이다.  

```sql
hibernate.SQL                        : 
    select
        orders0_.id as id1_0_0_,
        users1_.id as id1_1_1_,
        orders0_.order_name as order_na2_0_0_,
        orders0_.user_id as user_id3_0_0_,
        users1_.user_name as user_nam2_1_1_ 
    from
        orders orders0_ 
    inner join
        users users1_ 
            on orders0_.user_id=users1_.id
```

지연로딩이지만 Orders와 Users를 한번에 조회한다.  
이처럼 한번에 끝낼 수 있는데, N+1 문제가 발생해서  
성능저하가 생기지 않도록 조심해야 한다.  


Fetch Join으로 해결했지만 Fetch Join의 한계도 분명하다.  
이전에 작성한 글이 있으니, 기억이 안날 경우 다시 복습하도록 한다.  

[Fetch Join 참고](https://leegicheol.github.io/jpa/jpa-jpql-use-2/#%ED%8E%98%EC%B9%98-%EC%A1%B0%EC%9D%B8-fetch-join)  

---

##### **해결방법. 2**  
Fetch Join의 단점은 모든 컬럼을 다 조회해야된다는 점이다.  
만약 테이블에 굉장히 많은 컬럼이 있고, 실질적으로 사용되는 컬럼 수는 적다면  
모든 컬럼을 조회하는 것이 부담스러울 수 있다.  

이때 New 키워드를 사용해서 JPQL이 DTO를 리턴하게 할 수 있다.  

아래는 orderName, userId, userName(Users) 세 개의 컬럼을 리턴 받는 예제이다.  

```java
@RestController
@RequiredArgsConstructor
public class OrdersController {

    private final OrdersService ordersService;

    @PostMapping("/findAllOrders/new")
    public ResponseEntity<?> findAllNew() {
        List<FindOrdersNewResponseDto> orders = ordersService.findAllNew();

        return ResponseEntity.ok(orders);
    }

}
```

```java
@Service
@RequiredArgsConstructor
public class OrdersService {

    private final OrdersRepository ordersRepository;

    public List<FindOrdersNewResponseDto> findAllNew() {
        return ordersRepository.findAllNew();
    }

}
```

```java
@Repository
@RequiredArgsConstructor
public class OrdersRepository {

    private final EntityManager em;

    public List<FindOrdersNewResponseDto> findAllNew() {
        return em.createQuery(
                    "SELECT new com.example.jparest.test.dto.FindOrdersNewResponseDto(o.name, u.id, u.name) " +
                    "JOIN o.users u", Orders.class)
                .getResultList();
    }

}
```

```java
@Data
public class FindOrdersNewResponseDto {
    
    public FindOrdersNewResponseDto(String orderName, Long userId, String userName) {
        this.orderName = orderName;
        this.users = new FindUsersNewResponseDto(userId, userName);
    }

    private String orderName;
    private FindUsersNewResponseDto users;

}

@Data
public class FindUsersNewResponseDto {

    public FindUsersNewResponseDto(Long id, String name) {
        this.id = id;
        this.name = name;
    }

    private Long id;
    private String name;

}
```

JPQL이 리턴할 DTO 클래스는 필요한 컬럼만 필드로 설정하고,  
생성자를 통해 주입할 예정이다.  
OrdersRepository를 보면 쿼리가 조금 지저분하다.  
SELECT 후에 new 키워드를 사용하고, DTO 클래스의 package명을 전부 써줘야한다.  
그리고 생성자를 넣어주면 된다.  

```sql
hibernate.SQL                        : 
    select
        orders0_.order_name as col_0_0_,
        users1_.id as col_1_0_,
        users1_.user_name as col_2_0_ 
    from
        orders orders0_ 
    inner join
        users users1_ 
            on orders0_.user_id=users1_.id
```

실행해보면 쿼리도 한번 나가고,  
SELECT문에서 내가 필요한 컬럼만 받아오는 것을 알 수 있다.  

대신 단점으로는 Repository에서 Entity가 아닌 DTO를 기준으로 조회해온다는 점,  
JPQL이 복잡해진다는 점,  
다른 API에서 재사용하기 힘들다는 점이 있다.  


Fetch Join과 비교했을 때 무엇이 더 좋다 할 수 없는  
트레이드오프가 있다.  

상황에 맞게 사용하도록 한다.  

---

##### **DTO**  
위 예제에서 DTO를 사용했는데 

```java
@Data
public class FindOrdersNewResponseDto {
    
    public FindOrdersNewResponseDto(String orderName, Long userId, String userName) {
        this.orderName = orderName;
        this.users = new FindUsersNewResponseDto(userId, userName);
    }

    private String orderName;
    private FindUsersNewResponseDto users;

}

@Data
public class FindUsersNewResponseDto {

    public FindUsersNewResponseDto(Long id, String name) {
        this.id = id;
        this.name = name;
    }

    private Long id;
    private String name;

}
```

이와같이 FindOrdersNewResponseDto 에서  
Users를 사용하지 않고 FindUsersNewResponseDto를 사용했다.  

DTO로 래핑을 하더라도 안에 사용되는 필드가 엔티티면 안된다.    
그렇기때문에 엔티티를 래핑할 클래스를 따로 만든 것이다.  

---