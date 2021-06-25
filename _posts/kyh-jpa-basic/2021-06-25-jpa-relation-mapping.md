---
title : "JPA 다양한 연관관계 매핑"
categories : 
    - jpa
tag :
    - jpa
toc : true
---

[김영한 : 자바 ORM 표준 JPA 프로그래밍 - 기본편](https://www.inflearn.com/course/ORM-JPA-Basic#curriculum) 강의를 공부한 내용입니다.  

---

#### **다중성**  
- 다대일 : @ManyToOne
- 일대다 : @OneToMany
- 일대일 : @OneToOne
- 다대다 : @ManyToMany

---

#### **단방향, 양방향**  
- 테이블
    - 외래키 하나로 양쪽 조인이 가능하다.  
    - 사실 데이터베이스에 방향이라는 개념은 없다.  

- 객체  
    - 참조용 필드가 있는 쪽으로만 참조가 가능하다.  
    - 한쪽만 참조하면 단방향이며, 양쪽이 서로 참조하면 양방향이다.  

---

#### **연관관계의 주인**  
- 테이블은 외래키 하나로 두 테이블이 연관관계를 맺는다.  
- 객체 양방향 관계는 A -> B, B -> A 처럼 참조가 두 군데로, 둘 중 테이블의 외래키를 관리할 곳을 지정해야한다.  
- 연관관계의 주인은 외래키를 관리하는 참조이다.  
- 주인의 반대편은 외래키에 병향을 주지 않고, 단순 조회만 가능하다.  

---

#### **다대일**  
[다대일 관계 정리](https://leegicheol.github.io/jpa/jpa-relation-mapping/)  

---

#### **일대다 단방향**  
![error](/assets/images/kyh-jpa-basic/13-one-to-many.png)   

일대다 단방향은 "일"이 연관관계의 주인이다.  
외래키는 "다" 쪽에 있으며, 객체와 테이블의 차이 때문에 반대편 테이블의 외래 키를 관리하는 특이한 구조이다.  

```java
@Entity
public class Member {

    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    @Column(name = "USERNAME")
    private String name;
}

@Entity
public class Team {

    @Id @GeneratedValue
    @Column(name = "TEAM_ID")
    private Long id;

    private String name;

    @OneToMany
    @JoinColumn(name = "TEAM_ID")
    private List<Member> members = new ArrayList<>();

}
```

@JoinColumn을 꼭 사용해야 한다.  
members와 매핑된 TEAM_ID를 JoinColumn으로 사용하면 된다.  
사용하지 않으면 조인 테이블을 하나 더 생성해서 관리된다.  

엔티티가 관리하는 외래키가 다른 테이블에 있으므로,  
연관관계 관리를 위해 아래와 같이 UPDATE SQL이 추가적으로 발생된다. 

```java
Member member = new Member();
member.setName("Memer1");

em.persist(member);


Team team = new Team();
team.setName("TeamA");
team.getMembers().add(member);

em.persist(team);

//

Hibernate: 
    /* insert me.gicheol.Member
        */ insert 
        into
            Member
            (USERNAME, MEMBER_ID) 
        values
            (?, ?)
Hibernate: 
    /* insert me.gicheol.Team
        */ insert 
        into
            Team
            (name, TEAM_ID) 
        values
            (?, ?)
Hibernate: 
    /* create one-to-many row me.gicheol.Team.members */ update
        Member 
    set
        TEAM_ID=? 
    where
        MEMBER_ID=?
```

일대다 단방향 매핑을 쓰는 것보단 다대일 양방향 매핑을 사용하는 것이 낫다.  

---

#### **일대다 양방향**  
공식적으로 지원하지 않는 매핑이나 읽기 전용 필드를 사용해서 양방향처럼 사용할수는 있다.  

```java
@Entity
public class Member {

    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    @Column(name = "USERNAME")
    private String name;

    @ManyToOne
    @JoinColumn(name = "TEAM_ID", insertable = false, updatable = false)
    private Team team;

}

@Entity
public class Team {

    @Id @GeneratedValue
    @Column(name = "TEAM_ID")
    private Long id;

    private String name;

    @OneToMany
    @JoinColumn(name = "TEAM_ID")
    private List<Member> members = new ArrayList<>();

}
```

마찬가지로 복잡하기 때문에 다대일 양방향을 사용하는 것이 낫다.  

---

#### **일대일 관계**  
- 일대일 관계는 그 반대도 일대일이다.  
- 주 테이블이나 대상 테이블 중 외래키 선택을 할 수 있다.  
- 외래키에 데이터베이스 Unique 제약조건이 추가된다.  

##### **일대일 주 테이블에 외래키 단방향**

![error](/assets/images/kyh-jpa-basic/14-one-to-one.png)   

```java
@Entity
public class Member {

    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    @Column(name = "USERNAME")
    private String name;

    @OneToOne
    @JoinColumn(name = "LOCKER_ID")
    private Locker locker;

}


@Entity
public class Locker {
    
    @Id @GeneratedValue
    private Long id;
    
    private String name;

}
```


다대일 단방향 매핑과 사용방법이 비슷하다.  
그저 OneToOne Annotation을 사용할 뿐이며 마찬가지로, 꼭 JoinColumn을 사용해줘야한다.  

---

##### **일대일 주 테이블에 외래키 양방향**

![error](/assets/images/kyh-jpa-basic/15-one-to-one.png)   

```java
@Entity
public class Member {

    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    @Column(name = "USERNAME")
    private String name;

    @OneToOne
    @JoinColumn(name = "LOCKER_ID")
    private Locker locker;

}


@Entity
public class Locker {
    
    @Id @GeneratedValue
    private Long id;
    
    private String name;

    @OneToOne(mappedBy = "locker")
    private Member member;

}
```

마찬가지로 다대일 양방향 매핑과 비슷하다.  
왜래키가 있는곳이 연관관계의 주인이며, 반대편은 mappedBy를 사용한다.  

---

##### **일대일 대상 테이블에 외래키 단방향**

![error](/assets/images/kyh-jpa-basic/16-one-to-one.png)   

일대다 단방향과 비슷하다.  
그러나 이러한 매핑은 JPA에서 사용 불가하다.  

단 양방향은 가능하다.  

---

##### **일대일 대상 테이블에 외래키 양방향**

![error](/assets/images/kyh-jpa-basic/17-one-to-one.png)   

사실 일대일 주 테이블에 외래키 양방향과 매핑하는 방법은 동일하다.  

---

##### **일대일 정리**  
- 주 테이블에 외래키
    - 주 객체가 대상 객체의 참조를 가지는 것처럼 주 테이블에 외래키를 두고 대상 테이블을 찾을 수 있다.  
    - JPA 매핑이 편리하다.  
    - 값이 없다면 외래키에 null을 허용한다.  
- 대상 테이블에 외래키
    - 대상 테이블에 외래키가 존재한다.
    - 주 테이블과 대상 테이블을 일대일에서 일대다 관계로 변경할 때 테이블 구조가 유지된다.  
    - 프록시 기능의 한계로 지연 로딩으로 설정해도 항상 즉시 로딩된다.  

---