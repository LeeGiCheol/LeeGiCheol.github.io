---
title : "JPA 즉시로딩과 지연로딩"
categories : 
    - jpa
tag :
    - jpa
toc : true
---

[김영한 : 자바 ORM 표준 JPA 프로그래밍 - 기본편](https://www.inflearn.com/course/ORM-JPA-Basic#curriculum) 강의를 공부한 내용입니다.  

---

#### **의문**  
단순히 Member 정보만을 조회하는 비즈니스 로직에서  
Member를 조회할 때 Team도 조회해야 할까?  

JPA는 이를 해결하고자 지연로딩을 지원한다.  
지연로딩을 사용해 프록시로 조회할 수 있다.  

---

##### **지연 로딩 (Lazy Loading)**  
지연로딩을 설정하는 방법은 간단하다.  
연관관계 Annotation에 fetch 옵션으로 FetchType.Lazy를 주면된다.  

```java
@Entity
public class Member {

    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    @Column(name = "USERNAME")
    private String name;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "TEAM_ID")
    private Team team;

}

public class Main {
    public static void main(String[] args) {
        Team team = new Team();
        team.setName("teamA");
        em.persist(team);
        
        Member member = new Member();
        member.setName("member");
        member.setTeam(team);
        em.persist(member);

        em.flush();
        em.clear();

        Member m = em.find(Member.class, member.getId());

        System.out.println("m.getTeam().getClass() = " + m.getTeam().getClass());
    }
}

//
Hibernate: 
    select
        member0_.MEMBER_ID as MEMBER_I1_3_0_,
        member0_.createdAt as createdA2_3_0_,
        member0_.createdBy as createdB3_3_0_,
        member0_.lastUpdatedAt as lastUpda4_3_0_,
        member0_.lastUpdatedBy as lastUpda5_3_0_,
        member0_.USERNAME as USERNAME6_3_0_,
        member0_.TEAM_ID as TEAM_ID7_3_0_ 
    from
        Member member0_ 
    where
        member0_.MEMBER_ID=?
m.getTeam().getClass() = class me.gicheol.Team$HibernateProxy$oBnJi6H9
```

기존에는 위와 같은 코드가 있다면,  
Member와 Team을 Join을 한 쿼리가 발생했다.  

그러나 지금은 지연로딩을 사용했기 때문에 이때 조회된 Team은 프록시 객체이며,  
그렇기 때문에 Team을 조회하지 않는다.  

만약 여기서 Team에 관련된 컬럼을 사용한다면 얘기가 달라진다.  

```java
public static void main(String[] args) {
    
    ...

    System.out.println("m.getTeam().getName() = " + m.getTeam().getName());

}


// 
Hibernate: 
    select
        member0_.MEMBER_ID as MEMBER_I1_3_0_,
        member0_.createdAt as createdA2_3_0_,
        member0_.createdBy as createdB3_3_0_,
        member0_.lastUpdatedAt as lastUpda4_3_0_,
        member0_.lastUpdatedBy as lastUpda5_3_0_,
        member0_.USERNAME as USERNAME6_3_0_,
        member0_.TEAM_ID as TEAM_ID7_3_0_ 
    from
        Member member0_ 
    where
        member0_.MEMBER_ID=?
m.getTeam().getClass() = class me.gicheol.Team$HibernateProxy$JmyAcRbI
Hibernate: 
    select
        team0_.TEAM_ID as TEAM_ID1_7_0_,
        team0_.name as name2_7_0_ 
    from
        Team team0_ 
    where
        team0_.TEAM_ID=?
m.getTeam().getName() = teamA
```

동일한 코드이지만 Team의 Name을 건들이는 순간 Team은 초기화 되어 SELECT문을 사용한다.  

지난 번 공부한 프록시를 잘 이해했다면, 지연로딩도 쉽게 이해 할 수 있을 것 이다.  

---

##### **즉시 로딩 (Eager Loading)**  
반대로 Member와 Team이 같이 조회되는 경우가 더 많은 경우도 있을 것이다.  
이런 상황에서 지연 로딩을 사용한다면, 쿼리가 괜히 두번 사용되게 되므로 오히려 손해를 볼 수 있다.  

이런 경우라면 즉시로딩을 사용할 수 있다.  

마찬가지로 fetch 옵션에 FetchType.EAGER를 사용해주면 된다.  

---

##### **주의사항**  
즉시 로딩을 적용했을 때 예상치 못한 쿼리가 발생할 수 있기 때문에  
사실 가급적 지연 로딩을 사용하는 것이 좋다.  

소스코드상에서 봤을 때 Member만을 조회하는데, Team이 Join이 되는 것 처럼 말이다.  

또한 즉시로딩은 JPQL을 사용했을 때 **N+1** 문제를 일으킨다.  

```java
@Entity
public class Member {

    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    @Column(name = "USERNAME")
    private String name;

    @ManyToOne(fetch = FetchType.EAGER)
    @JoinColumn(name = "TEAM_ID")
    private Team team;

}

public class Main {
    public static void main(String[] args) {
        Team team = new Team();
        team.setName("teamA");
        em.persist(team);

        Member member = new Member();
        member.setName("member");
        member.setTeam(team);
        em.persist(member);

        em.flush();
        em.clear();

        List<Member> members = em.createQuery("select m from Member m", Member.class)
                        .getResultList();
    }
}

//
Hibernate: 
    /* select
        m 
    from
        Member m */ select
            member0_.MEMBER_ID as MEMBER_I1_3_,
            member0_.createdAt as createdA2_3_,
            member0_.createdBy as createdB3_3_,
            member0_.lastUpdatedAt as lastUpda4_3_,
            member0_.lastUpdatedBy as lastUpda5_3_,
            member0_.USERNAME as USERNAME6_3_,
            member0_.TEAM_ID as TEAM_ID7_3_ 
        from
            Member member0_
Hibernate: 
    select
        team0_.TEAM_ID as TEAM_ID1_7_0_,
        team0_.name as name2_7_0_ 
    from
        Team team0_ 
    where
        team0_.TEAM_ID=?
```

분명히 EAGER인데, 쿼리가 조인되지 않고 두번 나간다.  
JPQL은 SQL로 번역이 되어 쿼리가 실행되는데,  
이 시기에서 JPQL은 SELECT문을 실행한 후에야 즉시로딩이라는 것을 알 수 있다.  
그렇기 때문에 뒤늦게 한번 더 쿼리를 한 것이다.  

JPQL은 실무에서 굉장히 많이 사용되는데, 이러한 쿼리는 상당히 위험할 수 있다.  

```java
public class Main {
    public static void main(String[] args) {
        Team teamA = new Team();
        teamA.setName("teamA");
        em.persist(teamA);

        Member memberA = new Member();
        memberA.setName("memberA");
        memberA.setTeam(teamA);
        em.persist(memberA);

        Team teamB = new Team();
        teamB.setName("teamB");
        em.persist(teamB);

        Member memberB = new Member();
        memberB.setName("memberB");
        memberB.setTeam(teamB);
        em.persist(memberB);

        em.flush();
        em.clear();

        List<Member> members = em.createQuery("select m from Member m", Member.class)
                .getResultList();
    }
}

//
Hibernate: 
    /* select
        m 
    from
        Member m */ select
            member0_.MEMBER_ID as MEMBER_I1_3_,
            member0_.createdAt as createdA2_3_,
            member0_.createdBy as createdB3_3_,
            member0_.lastUpdatedAt as lastUpda4_3_,
            member0_.lastUpdatedBy as lastUpda5_3_,
            member0_.USERNAME as USERNAME6_3_,
            member0_.TEAM_ID as TEAM_ID7_3_ 
        from
            Member member0_
Hibernate: 
    select
        team0_.TEAM_ID as TEAM_ID1_7_0_,
        team0_.name as name2_7_0_ 
    from
        Team team0_ 
    where
        team0_.TEAM_ID=?
Hibernate: 
    select
        team0_.TEAM_ID as TEAM_ID1_7_0_,
        team0_.name as name2_7_0_ 
    from
        Team team0_ 
    where
        team0_.TEAM_ID=?
```

이 정도로 쿼리가 길어지니 제대로 실감이 난다.  
이러한 경우를 N+1 문제라고 부른다.  

그냥 전부 지연로딩을 사용하고, 필요에 따라 JPQL의 fetch 조인이나, 엔티티 그래프 기능을 사용하도록 하자.

추가적으로 @OneToMany와 @ManyToMany는 기본 옵션이 지연로딩이다.  
그러나 @ManyToOne과 @OneToOne의 기본 옵션은 즉시로딩이다.  
사용 시 지연로딩으로 바꿔서 사용하도록 하자.  

![error](/assets/images/kyh-jpa-basic/26-many-to-one.png)   
![error](/assets/images/kyh-jpa-basic/27-one-to-many.png)   

---