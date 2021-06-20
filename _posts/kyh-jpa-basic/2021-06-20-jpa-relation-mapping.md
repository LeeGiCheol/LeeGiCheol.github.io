---
title : "JPA 연관관계 매핑"
categories : 
    - jpa
tag :
    - jpa
toc : true
---

[김영한 : 자바 ORM 표준 JPA 프로그래밍 - 기본편](https://www.inflearn.com/course/ORM-JPA-Basic#curriculum) 강의를 공부한 내용입니다.  

---

#### **객체와 테이블의 연관관계**  

```java
@Entity @Data
public class Member {

    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    @Column(name = "USERNAME")
    private String name;

    @Column(name = "TEAM_ID")
    private Long teamId;
}


@Entity @Data
public class Team {

    @Id @GeneratedValue
    @Column(name = "TEAM_ID")
    private Long id;

    private String name;
}
```

위와 같이 두 개의 엔티티가 있다고 가정하자.  
객체를 테이블에 맞춰 설계를 한 상황인데, 이게 생각보다 문제가 많다.  

```java
Team team = new Team();
team.setName("TEAM_A");

em.persist(team);

Member member = new Member();
member.setName("MEMBER1");
member.setTeamId(team.getId());

em.persist(member);
```

객체가 아닌 외래키를 set 해주는 것이 객체지향적이지 않다.  

만약 Member의 Team을 구하려한다면 더더욱 마음에 들지 않아진다.  

```java
Member findMember = em.find(Member.class, member.getId());
Long findTeamId = findMember.getTeamId();

Team findTeam = em.find(Team.class, findTeamId);
```

Member를 구해왔으니, findMember.getTeam() 을 통해 Team을 구할 수 있을 것 같으나 그렇지 않다.  
ORM의 취지랑 전혀 어울리지 않는다.  

테이블은 외래키로 조인을 사용해 연관된 테이블을 찾지만,  
객체는 참조를 통해 연관된 객체를 찾는다.  

이처럼 둘은 큰 차이가 있다.

---

##### **단방향 연관관계**  
위의 상황을 해결하기 위해 Member 엔티티는 teamId 대신 team을 컬럼으로 가지려한다.  

```java
@Entity
public class Member {

    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    @Column(name = "USERNAME")
    private String name;
    
    private Team team;      // 컴파일 에러

}
```

컴파일 에러가 발생하는 이유는 JPA는 이러한 경우 둘의 관계가 어떤지 모르기 때문이다.  

아래는 순서대로 객체 연관관계와 테이블 연관관계이다.  

![error](/assets/images/kyh-jpa-basic/9-many-to-one.png)  
![error](/assets/images/kyh-jpa-basic/10-many-to-one.png)  


위의 사진과 같이 하나의 Team에는 여러명의 Member가 존재한다.  
그렇기 때문에 Member 입장에서는 Member가 다, Team이 일이므로,   
@ManyToOne Annotation을 사용한다.  


```java
@Entity
public class Member {

    ...
    
    @ManyToOne
    private Team team;

}
```

이제 객체지향적이다.  
그러나 여전히 문제는 있다.  
DB는 외래키를 통해 조인을 하는데, Team 객체를 사용하니 외래키를 알 수 없다는 점이다.  
그렇기때문에 Team의 외래키를 직접 설정해줘야한다.  
이때 사용되는 Annotation이 @JoinColumn이다.  

```java
@Entity
public class Member {

    ...
    
    @ManyToOne
    @JoinColumn(name = "TEAM_ID")
    private Team team;

}
```

---

##### **양방향 연관관계**  
