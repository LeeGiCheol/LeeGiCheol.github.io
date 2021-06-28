---
title : "JPA 상속관계 매핑"
categories : 
    - jpa
tag :
    - jpa
toc : true
---

[김영한 : 자바 ORM 표준 JPA 프로그래밍 - 기본편](https://www.inflearn.com/course/ORM-JPA-Basic#curriculum) 강의를 공부한 내용입니다.  

---

#### **상속관계 매핑**  
관계형 데이터베이스에 상속관계는 존재하지 않는다.  
그렇기 때문에 객체의 상속과 제일 유사한 SuperType, SubType 관계라는 모델링 기법을 사용한다.  

---

#### **전략**  
DB는 상속관계를 매핑하는 방법이 크게 3가지가 있다.  
JPA는 어떤 방법을 사용해도 모두 매핑을 할 수 있다.  

```java
@Entity
public class Item {

    @Id @GeneratedValue
    private Long id;

    private String name;

    private int price;

}


@Entity
public class Book extends Item {

    private String author;

    private String isbn;

}


@Entity
public class Album extends Item {

    private String artist;

}


@Entity
public class Movie extends Item {

    private String director;

    private String actor;
    
}

//

Hibernate: 
    
    create table Item (
       DTYPE varchar(31) not null,
        id bigint not null,
        name varchar(255),
        price integer not null,
        actor varchar(255),
        director varchar(255),
        author varchar(255),
        isbn varchar(255),
        artist varchar(255),
        primary key (id)
    )
```

기본 전략으로는 Item 테이블에 모든 컬럼을 몰아버리는 단일 테이블 전략이 사용된다.  
@Inheritance Annotation으로 전략을 변경할 수 있다.  

---

##### **조인 전략**  
![error](/assets/images/kyh-jpa-basic/18-inheritance-mapping.png)   
데이터가 INSERT 될때 두개의 테이블에 INSERT된다.  
조회 시 각각의 테이블과 조인을 통해 조회한다.  

```java
@Entity
@Inheritance(strategy = InheritanceType.JOINED)
public class Item {

    @Id @GeneratedValue
    private Long id;

    private String name;

    private int price;

}

//

Hibernate: 
    
    create table Album (
       artist varchar(255),
        id bigint not null,
        primary key (id)
    )
Hibernate: 
    
    create table Book (
       author varchar(255),
        isbn varchar(255),
        id bigint not null,
        primary key (id)
    )
Hibernate: 

    create table Item (
       id bigint not null,
        name varchar(255),
        price integer not null,
        primary key (id)
    )
Hibernate: 
    
    create table Movie (
       actor varchar(255),
        director varchar(255),
        id bigint not null,
        primary key (id)
    )
```

```java
Movie movie = new Movie();
movie.setDirector("LEE");
movie.setActor("GICHEOL");
movie.setName("바람과 함께 사라지다.");
movie.setPrice(10000);

em.persist(movie);

//

Hibernate: 
    /* insert me.gicheol.Movie
        */ insert 
        into
            Item
            (name, price, id) 
        values
            (?, ?, ?)
Hibernate: 
    /* insert me.gicheol.Movie
        */ insert 
        into
            Movie
            (actor, director, id) 
        values
            (?, ?, ?)
```

위에서 말했듯 movie를 persist하면 item에도 INSERT가 발생한다.  

SELECT도 마찬가지이다.  
movie를 조회하면 알아서 join을 통해 조회를 한다.  

```java
Movie movie = new Movie();
movie.setDirector("LEE");
movie.setActor("GICHEOL");
movie.setName("바람과 함께 사라지다.");
movie.setPrice(10000);

em.persist(movie);

em.flush();
em.clear();

Movie findMovie = em.find(Movie.class, movie.getId());
System.out.println("findMovie = " + findMovie);

//

Hibernate: 
    select
        movie0_.id as id1_2_0_,
        movie0_1_.name as name2_2_0_,
        movie0_1_.price as price3_2_0_,
        movie0_.actor as actor1_6_0_,
        movie0_.director as director2_6_0_ 
    from
        Movie movie0_ 
    inner join
        Item movie0_1_ 
            on movie0_.id=movie0_1_.id 
    where
        movie0_.id=?
```

여기서 @DiscriminatorColumn Annotation을 사용하면 부모 테이블에서 자식 테이블을 구분할 수 있는 컬럼이 주어진다.  

```java
@Entity
@Inheritance(strategy = InheritanceType.JOINED)
@DiscriminatorColumn
public class Item {

    @Id @GeneratedValue
    private Long id;

    private String name;

    private int price;
}

//

Hibernate: 
    
    create table Item (
       DTYPE varchar(31) not null,
        id bigint not null,
        name varchar(255),
        price integer not null,
        primary key (id)
    )
```

![error](/assets/images/kyh-jpa-basic/21-inheritance-mapping.png)   


이때 생기는 컬럼의 값은 기본적으로 테이블명이 컬럼의 값으로 들어가게 된다.  
만약 수정하고 싶다면 자식 테이블에서 @DiscriminatorValue를 사용하면 된다.  

```java
@Entity
@DiscriminatorValue("m")
public class Movie extends Item {

    private String director;

    private String actor;

}
```

![error](/assets/images/kyh-jpa-basic/22-inheritance-mapping.png)   

---

##### **단일 테이블 전략**
![error](/assets/images/kyh-jpa-basic/19-inheritance-mapping.png)   
상속 관계를 위와 같이 한개에 테이블에 모두 모으는 방법이다.  


```java
@Entity
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
public class Item {

    @Id @GeneratedValue
    private Long id;

    private String name;

    private int price;

}

//

Hibernate: 
    
    create table Item (
       DTYPE varchar(31) not null,
        id bigint not null,
        name varchar(255),
        price integer not null,
        actor varchar(255),
        director varchar(255),
        author varchar(255),
        isbn varchar(255),
        artist varchar(255),
        primary key (id)
    )
```

![error](/assets/images/kyh-jpa-basic/23-inheritance-mapping.png)   

한 테이블에 모든 컬럼이 들어가기 때문에  
각 테이블을 구분할 수 있는 컬럼이 무조건 필요하다.  
그렇기 때문에 @DiscriminatorColumn을 사용하지 않더라도 DTYPE 컬럼이 자동으로 생기며 이 컬럼을 통해 테이블을 구분한다.  

쿼리가 한번만 발생하기 때문에 성능적으로는 이점이 많다.  
그러나 중간중간 null값이 생기는 단점이 있다.  


---

##### **구현 클래스마다 테이블 전략**  
![error](/assets/images/kyh-jpa-basic/20-inheritance-mapping.png)   
NAME과 PRICE 컬럼을 테이블에서 각자 가지고 있는 방법이다.  

```java
@Entity
@Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)
public class Item {

    @Id @GeneratedValue
    private Long id;

    private String name;

    private int price;

}

//

Hibernate: 
    
    create table Album (
       id bigint not null,
        name varchar(255),
        price integer not null,
        artist varchar(255),
        primary key (id)
    )
Hibernate: 
    
    create table Book (
       id bigint not null,
        name varchar(255),
        price integer not null,
        author varchar(255),
        isbn varchar(255),
        primary key (id)
    )
Hibernate: 
    
    create table Movie (
       id bigint not null,
        name varchar(255),
        price integer not null,
        actor varchar(255),
        director varchar(255),
        primary key (id)
    )
```
---

이 방법은 Item 테이블이 생성되지 않는다.  
또한 각각 테이블로 나누었기 때문에 구분할 컬럼이 필요없다.  
@DiscriminatorColumn을 사용하더라도 아무일도 생기지 않는다.  

이러한 방식은 INSERT할때는 간편하지만 SELECT가 문제이다.  

```java
Movie movie = new Movie();
movie.setDirector("LEE");
movie.setActor("GICHEOL");
movie.setName("바람과 함께 사라지다.");
movie.setPrice(10000);

em.persist(movie);

em.flush();
em.clear();

Item findItem = em.find(Item.class, movie.getId());
System.out.println("findItem = " + findItem);

Hibernate: 
    /* insert me.gicheol.Movie
        */ insert 
        into
            Movie
            (name, price, actor, director, id) 
        values
            (?, ?, ?, ?, ?)
Hibernate: 
    select
        item0_.id as id1_2_0_,
        item0_.name as name2_2_0_,
        item0_.price as price3_2_0_,
        item0_.actor as actor1_6_0_,
        item0_.director as director2_6_0_,
        item0_.author as author1_1_0_,
        item0_.isbn as isbn2_1_0_,
        item0_.artist as artist1_0_0_,
        item0_.clazz_ as clazz_0_ 
    from
        ( select
            id,
            name,
            price,
            actor,
            director,
            null as author,
            null as isbn,
            null as artist,
            1 as clazz_ 
        from
            Movie 
        union
        all select
            id,
            name,
            price,
            null as actor,
            null as director,
            author,
            isbn,
            null as artist,
            2 as clazz_ 
        from
            Book 
        union
        all select
            id,
            name,
            price,
            null as actor,
            null as director,
            null as author,
            null as isbn,
            artist,
            3 as clazz_ 
        from
            Album 
    ) item0_ 
where
    item0_.id=?
```

ITEM 테이블을 조회하기 위해서는 구분할 방법이 없기 때문에,  
모든 자식 테이블을 UNION ALL을 통해 조회한다.  
굉장히 비효율적이다.  

---

#### **각 전략의 장단점**  

---

##### **조인 전략**  
**가장 이상적인 전략이다.**  

- 장점
    - 테이블이 정규화 되어있다.
    - 외래키 참조 무결성 제약조건을 활용 가능하다.
    - 저장공간 효율화
- 단점
    - 조회시 조인을 많이 사용하므로 쿼리가 복잡하며 성능 저하가 있다.  
    - 데이터 저장 시 INSERT 쿼리가 2번 호출된다.  

---

##### **단일 테이블 전략**  
**정말 단순한 상속관계라면 단일 테이블 전략을 사용해도 좋다.**  

- 장점
    - 조인이 필요 없으므로 조회 쿼리가 단순하며 성능이 빠르다.  
- 단점
    - 자식 엔티티가 매핑한 컬럼은 모두 null을 허용해야한다.  
    - 단일 테이블에 모든 것을 저장하기 때문에 테이블이 커질 수 있고, 그러므로 조회 성능이 오히려 떨어질 수 있다.  

---

##### **구현 클래스마다 테이블 전략**  
**이 전략은 아예 사용하지 않는 것이 좋다**  

- 장점
    - 서브타입을 명확하게 구분해서 처리할 때 효과적이다.  
    - not null 제약조건을 사용할 수 있다.  
- 단점
    - 여러 자식 테이블을 조회할때 성능이 매우 떨어진다.  
    - 시스템이 변경될때 수정해야 할 부분이 많다. (확장성이 떨어진다.)  

---

#### **@MappedSuperClass**  
공통 매핑 정보가 필요할 때 사용하는 Annotation이다.  
공통 매핑을 모은 클래스에 사용을 하고, 해당 매핑정보를 받아 쓸 엔티티에서 상속받아서 사용한다.  

```java
@MappedSuperclass
public class BaseEntity {

    @Column(name = "INSERT_ID")
    private String createdBy;
    private LocalDateTime createdAt;
    private String lastUpdatedBy;
    private LocalDateTime lastUpdatedAt;

}


@Entity
public class Member extends BaseEntity {

    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    @Column(name = "USERNAME")
    private String name;

}
```

@MappedSuperclass는 상속관계 매핑이 아니다.  
엔티티처럼 @Column과 같은 Annotation을 사용할 수는 있지만,  
엔티티는 아니다. 그러므로 BaseEntity라는 테이블이 생성되는 것은 아니다.  
자식 테이블에게 매핑 정보만 제공한다.  

직접 생성해서 사용하는 것이 아니기 때문에 추상 클래스로 사용하는 것이 좋다.  

---