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
단방향 연관관계에서는 Member.getTeam()은 가능하지만, Team.getMember()는 불가하다.  
두가지를 모두 가능하게 하는 관계를 양방향 연관관계라고 한다.  

![error](/assets/images/kyh-jpa-basic/11-one-to-many.png)    

기본적으로 테이블은 외래키 하나로 양방향 연관관계를 가진다.  
사실상 방향이라는 것이 없기에 테이블은 수정하지 않아도 된다.  

그러나 객체는 그렇지 않다.  
참조를 통해 연관관계를 가지기때문에 Team 엔티티에 Member를 추가해줘야한다.  

```java
@Entity
public class Team {

    @Id @GeneratedValue
    @Column(name = "TEAM_ID")
    private Long id;

    private String name;

    private List<Member> members = new ArrayList<>();
}
```

Team은 Member를 여러개 가질 수 있으니 List를 사용해서 만든다.  
그러나 단방향 연관관계와 동일하게 컴파일에러가 발생한다.  
이때 @OneToMany Annotation을 사용한다.   

```java
@Entity
public class Team {

    ...

    @OneToMany
    private List<Member> members = new ArrayList<>();

}
```

컴파일에러는 사라졌지만, 실행시키면 아래와 같이 정상적인 양방향 관계가 아닐 것이다.  

```java
Hibernate: 
    create table Member (
       MEMBER_ID bigint not null,
        USERNAME varchar(255),
        TEAM_ID bigint,
        primary key (MEMBER_ID)
    )

Hibernate: 
    create table Team (
       TEAM_ID bigint not null,
        name varchar(255),
        primary key (TEAM_ID)
    )

Hibernate: 
    create table Team_Member (
       Team_TEAM_ID bigint not null,
        members_MEMBER_ID bigint not null
    )

Hibernate: 
    alter table Team_Member 
       add constraint UK_tgr3dv1t34w4hlaxcn7hw0h2l unique (members_MEMBER_ID)

Hibernate: 
    alter table Member 
       add constraint FKl7wsny760hjy6x19kqnduasbm 
       foreign key (TEAM_ID) 
       references Team

Hibernate: 
    alter table Team_Member 
       add constraint FKpsjmea2e134ab3x3gsdoyxepg 
       foreign key (members_MEMBER_ID) 
       references Member

Hibernate: 
    alter table Team_Member 
       add constraint FK4u1npo283vgqfk8lyxclihgnl 
       foreign key (Team_TEAM_ID) 
       references Team
```

엔티티로 설정하지 않은 Team_Member 테이블이 생성되었다.  
왜 그럴까?  

먼저 객체와 테이블이 관계를 맺는 방식의 차이를 이해해야한다.  

객체는 두개의 객체가 연관관계를 맺기 위해선, 두개의 단방향 연관관계가 필요하다.  

```java
public class A {
    B b;
}

public class B {
    A a;
}
```

그러나 테이블은 FK 하나로 양방향 연관관계를 가질 수 있다.  


다시 돌아가서 위의 경우 객체 입장에선 사실 양방향이 아니라 각각의 단방향 연관관계인 것이다.  

그렇기 때문에 두 테이블의 PK인 MEMBER_ID와 TEAM_ID를 FK로 가진 조인 테이블이 생성된 것이다.  

그렇다면 어떻게 해결해야 할까?  

만약 멤버 중 한명의 팀이 변경되었다.  
그러면 MEMBER의 TEAM을 수정해야할까? TEAM의 MEMBER를 수정해야 할까?   

테이블을 다시 생각해보자.  

![error](/assets/images/kyh-jpa-basic/10-many-to-one.png)  

두 테이블을 연결하는 TEAM_ID는 두 테이블이 모두 가지고 있다.  
그럼 이 관계에 주인은 누구일까?  

---

##### **연관관계의 주인**
이런 애매한 상황이 생기기 때문에 JPA는 연관관계의 주인을 설정해줘야한다.  

TEAM 테이블은 TEAM_ID를 PK로 가지기 때문에 주인일 것 같아보이지만,  
PK인 TEAM_ID를 통해 조회 시 MEMBER가 여러명일 수 있기 때문에 연결된 하나의 관계를 찾을 수 없다.  

반대로 MEMBER는 TEAM_ID가 FK이다.  
TEAM_ID와 MEMBER_ID가 있다면, 연결된 하나의 관계를 찾을 수 있다.  

그렇기때문에 FK를 가진 쪽이 주인이 된다.  

다시말해 "다" 방향이 주인이 된다.  

이때 주인을 알리기 위해 mappedBy 옵션을 사용한다.  


```java
@Entity
public class Member {

    ...

    @ManyToOne
    @JoinColumn(name = "TEAM_ID")
    private Team team;

    ...
}

@Entity
public class Team {

    ...

    @OneToMany(mappedBy = "team")
    private List<Member> members = new ArrayList<>();

}
```

위 관계에서 Member.team이 관계의 주인이다.  
mappedBy 옵션엔 주인이 되는 필드의 변수명을 사용하면 된다.  


```java
Team team = new Team();
team.setName("TEAM_A");

em.persist(team);

Member member = new Member();
member.setName("MEMBER1");
member.setTeam(team);

em.persist(member);

em.flush();
em.clear();

Member findMember = em.find(Member.class, member.getId());
List<Member> members = findMember.getTeam().getMembers();

for (Member m : members) {
    System.out.println("member = " + m.getName());
}

// 결과
Hibernate: 
    create table Member (
       MEMBER_ID bigint not null,
        USERNAME varchar(255),
        TEAM_ID bigint,
        primary key (MEMBER_ID)
    )

Hibernate: 
    create table Team (
       TEAM_ID bigint not null,
        name varchar(255),
        primary key (TEAM_ID)
    )

Hibernate: 
    alter table Member 
       add constraint FKl7wsny760hjy6x19kqnduasbm 
       foreign key (TEAM_ID) 
       references Team

Hibernate: 
    /* insert me.gicheol.Team
        */ insert 
        into
            Team
            (name, TEAM_ID) 
        values
            (?, ?)
Hibernate: 
    /* insert me.gicheol.Member
        */ insert 
        into
            Member
            (USERNAME, TEAM_ID, MEMBER_ID) 
        values
            (?, ?, ?)
Hibernate: 
    select
        member0_.MEMBER_ID as MEMBER_I1_0_0_,
        member0_.USERNAME as USERNAME2_0_0_,
        member0_.TEAM_ID as TEAM_ID3_0_0_,
        team1_.TEAM_ID as TEAM_ID1_1_1_,
        team1_.name as name2_1_1_ 
    from
        Member member0_ 
    left outer join
        Team team1_ 
            on member0_.TEAM_ID=team1_.TEAM_ID 
    where
        member0_.MEMBER_ID=?
Hibernate: 
    select
        members0_.TEAM_ID as TEAM_ID3_0_0_,
        members0_.MEMBER_ID as MEMBER_I1_0_0_,
        members0_.MEMBER_ID as MEMBER_I1_0_1_,
        members0_.USERNAME as USERNAME2_0_1_,
        members0_.TEAM_ID as TEAM_ID3_0_1_ 
    from
        Member members0_ 
    where
        members0_.TEAM_ID=?
member = MEMBER1
```

이번에는 Member 테이블에 조인 테이블도 생성되지 않았고, TEAM_ID 라는 FK가 추가되었다.  

그리고 자유롭게 Member와 Team을 탐색할 수 있다는 것을 알 수 있다.  

---
