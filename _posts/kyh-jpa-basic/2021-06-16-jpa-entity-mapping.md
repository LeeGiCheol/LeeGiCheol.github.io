---
title : "JPA Entity Mapping"
categories : 
    - jpa
tag :
    - jpa
toc : true
---

[김영한 : 자바 ORM 표준 JPA 프로그래밍 - 기본편](https://www.inflearn.com/course/ORM-JPA-Basic#curriculum) 강의를 공부한 내용입니다.  

---

#### **객체와 테이블 매핑**  

##### **@Entity**  
@Entity 가 붙은 클래스는 JPA가 관리하며 엔티티라 한다.  

특징은 아래와 같다.  

- 기본 생성자가 필수이다.  
- final 클래스, enum, interface, inner 클래스 사용이 불가하다.
- DB에 저장할 필드는 final을 사용하지 못한다.  

속성 : name
- JPA에서 사용할 엔티티 이름을 지정한다.  
- 기본값은 클래스 이름이다.  

---

##### **@Table**  
@Table은 엔티티와 매핑할 테이블을 지정한다.  

속성 : name
- 매핑할 테이블 이름을 지정한다.  
- 기본값은 엔티티 이름이다.  

---

##### **데이터베이스 스키마 자동 생성**  
애플리케이션 실행 시점에 DDL을 자동으로 실행시켜준다.  
이때 데이터베이스 방언에 맞게 적절한 DDL을 실행시킨다.  

사용방법은 persistence.xml에 아래 옵션을 추가해주면 된다.  

```xml
<property name="hibernate.hbm2ddl.auto" value="create"/>
```

[value 참고](https://leegicheol.github.io/jpa/jpa-start/#jpa-%EC%84%A4%EC%A0%95%ED%95%98%EA%B8%B0)  

---

#### **DDL 생성 기능**  
DDL 생성 기능은 DDL을 자동 생성할 때만 사용된다.  

```java
@Column(unique = true, length = 10)
```

---

#### **필드와 컬럼 매핑**  

##### **@Column**  
컬럼 매핑을 한다.  
속성은 아래와 같다.  
- name : 필드와 매핑할 테이블의 컬럼 이름을 설정한다.  
- insertable, updatable : 등록/변경 가능 여부 false로 설정 시 반영되지 않는다.  
- nullable : null값의 허용 여부를 설정한다. false의 경우 not null 제약조건이 걸린다.   
- unique : unique 제약조건을 건다.  
- length : 문자 길이 제약조건. String 타입에만 사용한다.  
- columnDefinition : DB 컬럼 정보를 직접 줄 수 있다.  
- precision, scale : BigInteger / BigDecimal과 같은 아주 큰 수 에서 정밀한 값을 요구할 때 사용한다.   
    - precision은 소수점을 포함한 전체 자릿수. scale은 소수의 자리수  

---

##### **@Enumerated**  
Enum 타입 매핑을 한다.  
속성은 EnumType.ORDINAL과 EnumType.STRING 두 가지가 있다.  
ORDINAL은 enum의 순서이기 때문에 변동될 수 있는 가능성이 매우 높다.  
그렇기때문에 꼭 EnumType.STRING으로 사용하도록 한다.  

---

##### **@Temporal**  
Date, Calendar를 매핑할때 사용된다.  
LocalDate, LocalDateTime을 사용 할 경우 생략 가능하다.  

---

##### **@Lob**  
긴 문자에 매핑한다.  
문자열은 CLOB, 나머지는 BLOB으로 매핑된다.  

---

##### **@Transient**  
해당 Annotation이 붙은 필드는 매핑을 하지 않아 데이터베이스에 저장되지 않는다.  

---

#### **기본 키 매핑**  

##### **@Id**  
엔티티의 PK로 사용될 컬럼에 사용하며, 필수로 사용해야한다.  

##### **GeneratedValue**  

##### **GenerationType.IDENTITY**  
기본 키 생성을 데이터베이스에 위임. (Ex. MySQL Auto Increment)

Auto Increment는 Id값이 NULL로 들어오면 값을 생성한다.  
즉 INSERT 쿼리를 실행한 이후에 값을 알 수 있다.  
그런데 영속성 컨텍스트에서 관리되기 위해선 무조건 PK 값이 필요하다.  
이를 해결하기 위해 IDENTITY 옵션을 사용한 경우  
JPA는 transaction commit 시점이 아닌 persist 시점에 즉시 INSERT 쿼리를 실행한다.  

##### **GenerationType.SEQUENCE**  
데이터베이스 시퀀스 오브젝트 사용. (Ex. ORACLE Sequence)  
영속성 컨텍스트에 관리되기 위해 persist 시점에 DB로부터 Id의 값을 받고 객체에 값을 저장한다.  
실제 INSERT 쿼리는 commit 시점에 실행된다.  

이때 매 persist 시점에 시퀀스를 조회해야 하기 때문에 성능문제가 생길 수 있다.   
이는 미리 몇 개의 값을 불러올지 정하는 옵션인 allocationSize로 해결할 수 있다.  

아래는 allocationSize가 50인 경우이다.  

```java
tx.begin();

Member member1 = new Member();
member1.setUsername("A");

Member member2 = new Member();
member2.setUsername("B");

Member member3 = new Member();
member3.setUsername("C");

System.out.println("===============");

em.persist(member1);
em.persist(member2);
em.persist(member3);

System.out.println("===============");

tx.commit();


// 결과
===============
Hibernate: 
    call next value for MEMBER_SEQ
Hibernate: 
    call next value for MEMBER_SEQ
===============
Hibernate: 
    /* insert me.gicheol.Member
        */ insert 
        into
            Member
            (name, id) 
        values
            (?, ?)
Hibernate: 
    /* insert me.gicheol.Member
        */ insert 
        into
            Member
            (name, id) 
        values
            (?, ?)
Hibernate: 
    /* insert me.gicheol.Member
        */ insert 
        into
            Member
            (name, id) 
        values
            (?, ?)
```

3번의 persist가 발생했지만 시퀀스 조회는 2번 뿐이다.  
첫 조회에 1을, 두 번째 조회에 50을 미리 조회해온다.    
이 값을 메모리에 올려둔 후 Id가 51이 되기 전까지 시퀀스 조회를 하지 않는다.  

allocationSize는 높을수록 좋긴 하지만,  
미리 조회하고 메모리에 저장하는 것이기 때문에  
채워지지 않은 시퀀스는 서버를 내리는 순간 사용하지 못하게 되므로 적당한 값을 사용해야 한다.  


@SequenceGenerator 필요하다.  

```java
@Entity
@SequenceGenerator(
    name = "MEMBER_SEQ_GENERATOR",
    sequenceName = "MEMBER_SEQ",
    initialValue = 1, allocationSize = 1
)
public class Member {

    @Id @GeneratedValue(strategy = GenerationType.SEQUENCE,
                        generator = "MEMBER_SEQ_GENERATOR")
    private Long id;

}
```
##### **GenerationType.TABLE**  
키 생성용 테이블 사용. Sequence를 따라하는 것이기 때문에 속도가 느린편이다. 모든 DB에서 사용  

@TableGenerator 필요하다.  
```java
@Entity
        name = "MEMBER_SEQ_GENERATOR",
        table = "MY_SEQUENCES",
        pkColumnValue = "MEMBER_SEQ", allocationSize = 1
)
public class Member {

    @Id @GeneratedValue(strategy = GenerationType.TABLE,
                        generator = "MEMBER_SEQ_GENERATOR")
    private Long id;

}
```
##### **GenerationType.AUTO**  
방언에 따라 자동 지정  


---