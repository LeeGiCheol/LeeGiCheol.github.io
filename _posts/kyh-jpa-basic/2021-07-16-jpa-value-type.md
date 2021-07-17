---
title : "JPA 값 타입"
categories : 
    - jpa
tag :
    - jpa
toc : true
---

[김영한 : 자바 ORM 표준 JPA 프로그래밍 - 기본편](https://www.inflearn.com/course/ORM-JPA-Basic#curriculum) 강의를 공부한 내용입니다.  

---

#### **JPA 데이터 타입 분류**  
- **엔티티 타입**
    - @Entity로 정의하는 객체
    - 데이터가 변해도 식별자로 지속해서 추적 가능

- **값 타입**  
    - int, Integer, String 처럼 단순히 값으로 사용하는 자바 기본 타입이나 객체  
    - 식별자가 없고 값만 있으므로 변경 시 추적 불가  

---

##### **값 타입 분류**  
- **기본 값 타입**
    - 자바 기본 타입 (Primitive Type)  
    - 래퍼 클래스 (Integer, Long)  
    - String  
- **임베디드 타입**
- **컬렉션 값 타입**

---

##### **임베디드 타입**  
- 새로운 값 타입을 직접 정의할 수 있다.  
- 주로 기본 값 타입을 모아서 만들어서 복합 값 타입이라고도 한다.  
- int, String과 같은 값 타입이다.  

예를들어 Member라는 엔티티에 이러한 컬럼이 있다고 하자.  

```java
@Entity
public class Member {
    
    private Long id;
    private String name;
    
    private LocalDateTime startDate;
    private LocalDateTime endDate;
    
    private String city;
    private String street;
    private String zipcode;    

}
```

이러한 경우 공통적인 특징을 가진 컬럼들이 꽤 보인다.  
startDate, endDate는 날짜/시간 관련,  
city, street, zipcode는 주소 관련 이렇게 말이다.  

이런 경우 공통되는 컬럼을 하나의 클래스로 묶어서 사용할 수 있는데, 이를 임베디드 타입이라 한다.  

```java
public class Period {

    private LocalDateTime startDate;
    private LocalDateTime endDate;

}

public class Address {
    
    private String city;
    private String street;
    private String zipcode;
    
}


@Entity
public class Member {

    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    @Column(name = "USERNAME")
    private String name;

    private Period period;

    private Address address;

}
```

안타깝게도 Period와 Address는 기본 값 타입이 아니기 때문에 컴파일 에러가 발생한다.  
이를 해결하기 위해 두 가지 Annotation을 사용해야한다.  

- @Embeddable : 값 타입을 정의하는 곳에 표시  
- @Embedded : 값 타입을 사용하는 곳에 표시  
    - @Embeddable이 있다면 필수로 사용할 필요는 없지만 명확하게 값 타입이라는 것을 알리기 위해 사용하도록 한다.    

```java
@Entity
public class Member {

    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    @Column(name = "USERNAME")
    private String name;

    @Embedded
    private Period period;

    @Embedded
    private Address address;

}


@Embeddable
public class Period {

    private LocalDateTime startDate;
    private LocalDateTime endDate;

}

@Embeddable
public class Address {

    private String city;
    private String street;
    private String zipcode;

}
```

이제 이상없이 잘 동작하는 것을 알 수 있다.  
이떄 기본 생성자는 필수적으로 작성되어야 한다.  

---

##### **임베디드 타입 장점**  
- 재사용성  
- 높은 응집도  
- 임베디드 타입을 포함한 모든 값 타입은,  
  값 타입을 소유한 엔티티의 생명주기를 의존한다.  

---

##### **임베디드 타입과 테이블 매핑**  
임베디드 타입은 그저 엔티티의 값일 뿐이다.  
당연히 임베디드 타입을 사용하더라도 테이블의 매핑정보는 달라지지 않는다.  
또한 객체와 테이블을 아주 세밀하게 매핑하는 것이 가능하다.  

---

##### **@AttributeOverride**  
한 엔티티에서 같은 임베디드 타입을 두개 사용하려면 컬럼명이 중복되기 때문에 에러가 발생한다.  
이때 @AttributeOverride를 사용해서 해결할 수 있다.  

```java
@Entity
public class Member extends BaseEntity {

    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    @Column(name = "USERNAME")
    private String name;

    @Embedded
    private Period period;

    @Embedded
    private Address homeAddress;

    @Embedded
    @AttributeOverrides({
            @AttributeOverride(name = "city", column = @Column(name = "WORK_CITY")),
            @AttributeOverride(name = "street", column = @Column(name = "WORK_STREET")),
            @AttributeOverride(name = "zipcode", column = @Column(name = "WORK_ZIPCODE")),
    })
    private Address workAddress;

}
```

---

##### **임베디드 타입의 문제점**  
```java
public static void main(String[] args) {
    Address address = new Address("city", "street", "10000");

    Member member = new Member();
    member.setName("member");
    member.setHomeAddress(address);
    em.persist(member);

    Member member2 = new Member();
    member2.setName("member2");
    member2.setHomeAddress(address);
    em.persist(member2);

    member.getHomeAddress().setCity("newCity");
}
```

위의 코드의 결과로 기대하는 것은 member의 city만을 newCity로 변경하는 것이다.  
하지만 member2의 city 역시 newCity로 함께 변경된다.  

이러한 같은 경우는 꽤 많이 발생 할 수 있다.  

해결법은 아래와 같다.  

```java
public static void main(String[] args) {
    firstSolution();
    secondSolution();
}


public void firstSolution() {
    Address address = new Address("city", "street", "10000");

    Member member = new Member();
    member.setName("member");
    member.setHomeAddress(address);
    em.persist(member);

    Address member2Address = new Address(address.getCity(), address.getStreet(), address.getZipcode());

    Member member2 = new Member();
    member2.setName("member2");
    member2.setHomeAddress(member2Address);
    em.persist(member2);

    member.getHomeAddress().setCity("newCity");
}


public void secondSolution() {
    Address address = new Address("city", "street", "10000");

    Member member = new Member();
    member.setName("member");
    member.setHomeAddress(address);
    em.persist(member);

    Member member2 = new Member();
    member2.setName("member2");
    member2.setHomeAddress(memberAddress);
    em.persist(member2);

    Address member2Address = new Address("newCity", address.getStreet(), address.getZipcode());

    member2.setHomeAddress(member2Address);
}
```

다양한 방법이 있겠지만, 위 코드처럼 member2의 Address 객체를 새로 만드는 것이다.  
문제점은 사이드이펙트가 발생할 가능성이 너무 많은 것인데,  

이런 객체는 아예 불변객체로 생성하는 것이 좋은 방법이다.  
getter만 만들고, setter를 만들지 않는다.  

---

##### **값 타입의 비교**  
```java
public static void main(String[] args) {
    int a = 10;
    int b = 10;
    System.out.println("a == b : " + (a == b));

    Address address1 = new Address("city", "street", "10000");
    Address address2 = new Address("city", "street", "10000");
    System.out.println("address1 == address2 : " + (address1 == address2));
    System.out.println("address1.equals(address2) : " + address1.equals(address2));
}

//
a == b : true
address1 == address2 : false
address1.equals(address2) : false
```

값 타입은 == 비교가 가능하지만, Address와 같은 참조형 타입은 == 비교가 불가능하다.  
equals를 통해 비교를 해야하는데, Object 클래스의 equals 메서드의 기본은 == 비교이다.  

![error](/assets/images/kyh-jpa-basic/28-object-equals-method.png)    

그렇기 때문에 객체 클래스에서 equals 메서드를 재정의 해줘야한다. (hashCode 메서드도 같이 재정의 하도록 한다.)  

![error](/assets/images/kyh-jpa-basic/29-equals-hashcode.png)    

인텔리제이 혹은 이클립스와 같은 IDE를 사용하면 equals와 hashCode를 쉽게 생성해줄 수 있다.  

![error](/assets/images/kyh-jpa-basic/31-equals-hashcode.png)    

인텔리제이를 사용한다면 Use getters during code generation 옵션을 꼭 체크하도록 하자.  
equals, hashCode 메서드에서 필드에 직접 접근이 아닌  
getter를 통해 접근하도록 하는 옵션인데,  
프록시 객체의 경우 필드에 접근했을 때  
정상적으로 작동하지 않기 때문에 getter를 사용하는 것이 안전하다.  


```java
@Embeddable
public class Address {
    
    ...


    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Address address = (Address) o;
        return Objects.equals(getCity(), address.getCity()) && Objects.equals(getStreet(), address.getStreet()) && Objects.equals(getZipcode(), address.getZipcode());
    }

    @Override
    public int hashCode() {
        return Objects.hash(getCity(), getStreet(), getZipcode());
    }

}
```

메서드를 재정의 해준 후 다시 아래 코드를 실행하면 결과가 변경될 것이다.  

```java
public static void main(String[] args) {
    int a = 10;
    int b = 10;
    System.out.println("a == b : " + (a == b));

    Address address1 = new Address("city", "street", "10000");
    Address address2 = new Address("city", "street", "10000");
    System.out.println("address1 == address2 : " + (address1 == address2));
    System.out.println("address1.equals(address2) : " + address1.equals(address2));
}

//
a == b : true
address1 == address2 : false
address1.equals(address2) : true   <- false에서 true로 정상 동작한다.
```

---

##### **값 타입 컬렉션**  
값 타입을 컬렉션으로 사용하게 될 경우 DB에 별도 테이블이 추가되어야 한다.  

![error](/assets/images/kyh-jpa-basic/30-collection.png)    

@ElementCollection, @CollectionTable Annotation을 통해 사용할 수 있다.  

```java

@Entity
public class Member {

    ...

    @ElementCollection
    @CollectionTable(name = "FAVORITE_FOOD",
                        joinColumns = @JoinColumn(name = "MEMBER_ID"))
    @Column(name = "FOOD_NAME")
    private Set<String> favoriteFoods = new HashSet<>();

    @ElementCollection
    @CollectionTable(name = "ADDRESS_HISTORY",
                        joinColumns = @JoinColumn(name = "MEMBER_ID"))
    private List<Address> addressHistory = new ArrayList<>();

}
```

- @ElementCollection : JPA에게 컬렉션 객체라는 것을 알린다.  
- @CollectionTable : 새로 생성되야 할 테이블의 정보를 작성한다.  
    - name : 테이블 명
    - joinColumes : 조인 컬럼
- @Colume : favoriteFoods는 한개의 컬럼만 사용되기 때문에 
            변수명인 favoriteFoods가 생성되는 컬럼명이다.
            컬럼명을 수정하기위해 사용된다.  

```java
public static void main(String[] args) {
    Member member = new Member();
    member.setName("member");
    member.setHomeAddress(new Address("city", "street", "10000"));
    member.getFavoriteFoods().add("치킨");
    member.getFavoriteFoods().add("족발");
    member.getFavoriteFoods().add("피자");

    member.getAddressHistory().add(new Address("city1", "street1", "20000"));
    member.getAddressHistory().add(new Address("city2", "street2", "30000"));

    em.persist(member);
}
```

사용 할 때는 위 코드 처럼 사용할 수 있다.  

변경 할 때는 이전에 homeAddress를 변경할 때 처럼 새로운 객체를 만들어 줘야한다.  

```java
public static void main(String[] args) {
    Member member = new Member();
    member.setName("member");
    member.setHomeAddress(new Address("city", "street", "10000"));
    member.getFavoriteFoods().add("치킨");
    member.getFavoriteFoods().add("족발");
    member.getFavoriteFoods().add("피자");

    member.getAddressHistory().add(new Address("city1", "street1", "20000"));
    member.getAddressHistory().add(new Address("city2", "street2", "30000"));

    em.persist(member);

    Member findMember = em.find(Member.class, member.getId());

    Address homeAddress = findMember.getHomeAddress();
    findMember.setHomeAddress(new Address("newCity", homeAddress.getStreet(), homeAddress.getZipcode()));

    findMember.getFavoriteFoods().remove("치킨");
    findMember.getFavoriteFoods().add("한식");
    
    findMember.getAddressHistory().remove(new Address("city1", "street1", "20000"));
    findMember.getAddressHistory().add(new Address("newCollectionCity", "street1", "20000"));
}
```

---

##### **값 타입 컬렉션의 문제점**  
addressHistory().remove() 를 할때 객체 자체를 넣어준다.  
객체를 넣어주면, 리스트에 포함된 객체와 비교 후 동일한 객체를 삭제해주는데,  
이때 equals, hashCode 메서드가 정상적으로 재정의되어 있지 않다면,  
원하는 객체를 찾지 못해 삭제되지 않는다.  

꼭 재정의를 하도록 하자.  

다른 문제로는 안타깝게도 remove를 했을 때 ADDRESS 테이블에 정확히 데이터를 매핑할 컬럼이 없다.  

다시말해 WHERE절에 입력할 수 있는 값은 MEMBER_ID뿐이다.  

그렇기 때문에 MEMBER_ID에 해당하는 모든 데이터를 Delete하고, Insert하는 과정이 생긴다.  

또한 값 타입 컬렉션은 null을 허용하지 않으며, 중복 저장이 불가능하다.  

사실 업데이트가 될 일이 없는 단순한 경우엔 값 타입 컬렉션이 유용하게 사용되지만,  
아닐 경우라면 값 타입 컬렉션 보다는 일대다 관계를 고려한다.  

```java

@Entity
@Table(name = "ADDRESS")
public class AddressEntity {

    public AddressEntity() {}
    
    public AddressEntity(Long id, Address address) {
        this.id = id;
        this.address = address;
    }

    @Id @GeneratedValue
    private Long id;

    private Address address;

    public AddressEntity(String city, String street, String zipcode) {
        this.address = new Address(city, street, zipcode);
    }
}


@Entity
public class Member {

    ...

    @OneToMany(cascade = CascadeType.ALL)
    @JoinColumn(name = "MEMBER_ID")
    private List<AddressEntity> addressHistory = new ArrayList<>();

}


public class Main {
    public static void main(String[] args) {
        ...

        Member member = new Member();
        member.setName("member");
        member.setHomeAddress(new Address("city", "street", "10000"));
        member.getFavoriteFoods().add("치킨");
        member.getFavoriteFoods().add("족발");
        member.getFavoriteFoods().add("피자");

        member.getAddressHistory().add(new AddressEntity("city1", "street1", "20000"));
        member.getAddressHistory().add(new AddressEntity("city2", "street2", "30000"));

    }
}
```

Address 객체를 AddressEntity와 같은 Entity 객체로 한번 래핑하고 일대다 관계를 만들어 사용하는 것이다.  

---