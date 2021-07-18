---
title : "JPA JPQL 사용하기 2"
categories : 
    - jpa
tag :
    - jpa
toc : true
---

[김영한 : 자바 ORM 표준 JPA 프로그래밍 - 기본편](https://www.inflearn.com/course/ORM-JPA-Basic#curriculum) 강의를 공부한 내용입니다.  

---

#### **JPQL 경로 표현식**  
경로 표현식이란 점(.)을 찍어 객체 그래프를 탐색하는 것을 의미한다.  

```sql
SELECT m.username       -> 상태 필드
  FROM Member m
  JOIN m.team t         -> 단일 값 연관 필드
  JOIN m.orders o       -> 컬렉션 값 연관 필드
 WHERE t.name = '팀A'
```

---

##### **상태 필드 (state field)**  
단순히 값을 저장하기 위한 필드이다.  
상태 필드는 경로 탐색의 끝이므로 탐색이 더이상 되지 않는다.  

---

##### **연관 필드 (association field)**  
연관관계를 위한 필드이다.  

- 단일 값 연관 필드
    - 묵시적 내부조인이 발생하며, 탐색이 가능하다.  
    - @ManyToOne, @OneToOne, 대상이 엔티티 (ex. m.team)  

```java
String query = "SELECT m.team FROM Member m";

List<Team> resultList = em.createQuery(query, Team.class)
        .getResultList();


//
Hibernate: 
    /* SELECT
        m.team 
    FROM
        Member m */ select
            team1_.id as id1_3_,
            team1_.name as name2_3_ 
        from
            Member member0_ 
        inner join
            Team team1_ 
                on member0_.TEAM_ID=team1_.id
```

주석안의 SQL문이 JPQL문법인데,  
이는 주석 다음으로 발생하는 SQL문으로 해석된다.  
소스코드상으로 예상치 못한 조인이 발생하는데,  
이를 묵시적 내부조인이라 한다.  

<br>

- 컬렉션 값 연관 필드  
    - 묵시적 내부조인이 발생하며, 탐색이 불가능하다.  
    - @OneToMany, @ManyToMany, 대상이 컬렉션 (ex. m.orders)

```java
String query = "SELECT t.members FROM Team t";

Collection resultList = em.createQuery(query, Collection.class)
        .getResultList();

//
Hibernate: 
    /* SELECT
        t.members 
    FROM
        Team t */ select
            members1_.id as id1_0_,
            members1_.age as age2_0_,
            members1_.memberType as memberTy3_0_,
            members1_.TEAM_ID as TEAM_ID5_0_,
            members1_.username as username4_0_ 
        from
            Team team0_ 
        inner join
            Member members1_ 
                on team0_.id=members1_.TEAM_ID
```

컬렉션은 경로 탐색의 끝으로,  
사용하기 위해선 명시적 조인을 통해 별칭을 얻어야한다.  

```java
String query = "SELECT m.username FROM Team t JOIN t.members m";

List<String> resultList = em.createQuery(query, String.class)
        .getResultList();
```

---

##### **명시적 조인, 묵시적 조인**  
- 명시적 조인 : JOIN 키워드를 직접 사용하는 것  

```sql
SELECT m FROM Member m JOIN m.team t 
```

- 묵시적 조인 : 경로 표현식에 의해 묵시적으로 SQL 조인 발생  
    - 내부조인만 가능하다.  

```sql
SELECT m.team from Member m
```

---

##### **경로 표현식 예제**  

```sql
SELECT o.member.team FROM Order o                -> 성공
```

```sql
SELECT t.members FROM Team t                     -> 성공
```

```sql
SELECT t.umembers.username FROM Team t           -> 실패
```

```sql
SELECT m.username FROM Team t JOIN t.members m   -> 성공
```

---

##### **묵시적 조인 시 주의사항**  
- 항상 내부조인이다.  
- 컬렉션은 경로 탐색의 끝으로, 사용하기 위해선 명시적 조인을 통해 별칭을 얻어야한다.  
- 경로 탐색은 주로 SELECT, WHERE 절에 사용하지만  
  묵시적 조인으로 인해 FROM (JOIN)절에 영향을 준다.  

뭇기적 조인은 조인이 일어나는 상황을 한눈에 파악하기 힘들다.  
**가급적 명시적 조인을 사용하자.**  

---

#### **페치 조인 (fetch join)**  
페치 조인은 SQL 조인의 종류가 아니다.  
JPQL에서 성능 최적화를 위해 제공하는 기능이다.  

연관된 엔티티나 컬렉션을 쿼리 한번에 조회할 수 있는 기능이다.  

---

##### **엔티티 페치 조인**  
SQL 한번에 멤버을 조회하면서 연관된 팀도 함께 조회한다.  

```sql
[JPQL]
SELECT m 
FROM Member m 
JOIN FETCH m.team
```

```sql
[SQL]
SELECT m.*, t.* 
FROM MEMBER m 
INNER JOIN TEAM t 
ON m.TEAM_ID = t.ID
```

---

##### **엔티티 페치 조인 예제**  
```java
public static void main(String[] args) {
    Team teamA = new Team();
    teamA.setName("팀A");
    em.persist(teamA);

    Team teamB = new Team();
    teamB.setName("팀B");
    em.persist(teamB);

    Member member1 = new Member();
    member1.setUsername("멤버1");
    member1.setTeam(teamA);
    em.persist(member1);

    Member member2 = new Member();
    member2.setUsername("멤버2");
    member2.setTeam(teamA);
    em.persist(member2);

    Member member3 = new Member();
    member3.setUsername("멤버3");
    member3.setTeam(teamB);
    em.persist(member3);

    em.flush();
    em.clear();

    String query = "SELECT m FROM Member m";

    List<Member> resultList = em.createQuery(query, Member.class)
            .getResultList();

    for (Member member : resultList) {
        System.out.println("member = " + member.getUsername() + ", " + member.getTeam().getName());
    }
}


//
Hibernate: 
    /* SELECT
        m 
    FROM
        Member m */ select
            member0_.id as id1_0_,
            member0_.age as age2_0_,
            member0_.memberType as memberTy3_0_,
            member0_.TEAM_ID as TEAM_ID5_0_,
            member0_.username as username4_0_ 
        from
            Member member0_
Hibernate: 
    select
        team0_.id as id1_3_0_,
        team0_.name as name2_3_0_ 
    from
        Team team0_ 
    where
        team0_.id=?
member = 멤버1, 팀A
member = 멤버2, 팀A
Hibernate: 
    select
        team0_.id as id1_3_0_,
        team0_.name as name2_3_0_ 
    from
        Team team0_ 
    where
        team0_.id=?
member = 멤버3, 팀B
```

먼저 Member 엔티티의 Team은 지연로딩 설정이 되어있다.  
JPQL으로 쿼리를 날린 후 member.getTeam().getName()을 하는데,  
이때 Team은 프록시 객체이므로 이 과정에서 SELECT문이 실행된다.  

멤버1을 출력하고 이때 팀A는 1차 캐시에 등록된다.  
멤버2는 1차 캐시에 등록 된 팀A를 조회한다.  

그리고 멤버 3을 출력하려고 보니 Team이 프록시 객체이다.  
여기서 한번 더 SELECT문이 발생된다.  

벌써부터 상당한 성능문제가 발생할 수 있어보인다.  
이런 경우 페치 조인을 통해 해결할 수 있다.  

Team과 Member는 위와 동일하다고 가정한다.  

```java
public static void main(String[] args) {
    
    ...

    String query = "SELECT m FROM Member m JOIN FETCH m.team";

    List<Member> resultList = em.createQuery(query, Member.class)
            .getResultList();

    for (Member member : resultList) {
        System.out.println("member = " + member.getUsername() + ", " + member.getTeam().getName());
    }
}


//
Hibernate: 
    /* SELECT
        m 
    FROM
        Member m 
    JOIN
        FETCH m.team */ select
            member0_.id as id1_0_0_,
            team1_.id as id1_3_1_,
            member0_.age as age2_0_0_,
            member0_.memberType as memberTy3_0_0_,
            member0_.TEAM_ID as TEAM_ID5_0_0_,
            member0_.username as username4_0_0_,
            team1_.name as name2_3_1_ 
        from
            Member member0_ 
        inner join
            Team team1_ 
                on member0_.TEAM_ID=team1_.id
member = 멤버1, 팀A
member = 멤버2, 팀A
member = 멤버3, 팀B
```

이번엔 페치 조인을 사용했기 때문에  
Team이 프록시 객체가 아닌 진짜 엔티티이다.  

그렇기 때문에 쿼리도 JOIN을 통해 한번에 Team 엔티티를 조회할 수 있는 것 이다.    

---

##### **컬렉션 페치 조인**  
일대다 관계, 컬렉션 페치 조인

```sql
[JPQL]
SELECT t 
FROM Team t
JOIN FETCH t.members
WHERE t.name = '팀A'
```

```sql
[SQL]
SELECT t.*, m.* 
FROM TEAM t
INNER JOIN MEMBER m
ON t.ID = m.TEAM_ID
WHERE t.NAME = '팀A'
```

---

##### **컬렉션 페치 조인 예제**  
```java
public static void main(String[] args) {
    Team teamA = new Team();
    teamA.setName("팀A");
    em.persist(teamA);

    Team teamB = new Team();
    teamB.setName("팀B");
    em.persist(teamB);

    Member member1 = new Member();
    member1.setUsername("멤버1");
    member1.setTeam(teamA);
    em.persist(member1);

    Member member2 = new Member();
    member2.setUsername("멤버2");
    member2.setTeam(teamA);
    em.persist(member2);

    Member member3 = new Member();
    member3.setUsername("멤버3");
    member3.setTeam(teamB);
    em.persist(member3);

    em.flush();
    em.clear();

    String query = "SELECT t FROM Team t JOIN FETCH t.members";

    List<Team> resultList = em.createQuery(query, Team.class)
            .getResultList();

   
    for (Team team : resultList) {
        System.out.println("team = " + team.getName() + " | members = " + team.getMembers().size());

        for (Member member : team.getMembers()) {
            System.out.println("-> member = " + member);
        }
    }
}


//
Hibernate: 
    /* SELECT
        t 
    FROM
        Team t 
    JOIN
        FETCH t.members */ select
            team0_.id as id1_3_0_,
            members1_.id as id1_0_1_,
            team0_.name as name2_3_0_,
            members1_.age as age2_0_1_,
            members1_.memberType as memberTy3_0_1_,
            members1_.TEAM_ID as TEAM_ID5_0_1_,
            members1_.username as username4_0_1_,
            members1_.TEAM_ID as TEAM_ID5_0_0__,
            members1_.id as id1_0_0__ 
        from
            Team team0_ 
        inner join
            Member members1_ 
                on team0_.id=members1_.TEAM_ID
team = 팀A | members = 2
-> member = Member{id=3, username='멤버1', age=0}
-> member = Member{id=4, username='멤버2', age=0}
team = 팀A | members = 2
-> member = Member{id=3, username='멤버1', age=0}
-> member = Member{id=4, username='멤버2', age=0}
team = 팀B | members = 1
-> member = Member{id=5, username='멤버3', age=0}
```

팀A가 두개가 나왔다는 것으로 보아   
원했던 값과는 조금 다른 값이 나왔다는 것을 알 수 있다.  

![error](/assets/images/kyh-jpa-basic/32-fetch.png)    

사진과 같이 DB 상으로는 팀A가 두개 나오는게 당연하지만,  
역시 원했던 결과와는 다르게 중복값이 생겼다.  

이때 DISTINCT 키워드를 사용하면  
식별자가 같은 객체를 삭제해준다.  

```java
public static void main(String[] args) {
    String query = "SELECT DISTINCT t FROM Team t JOIN FETCH t.members";

    List<Team> resultList = em.createQuery(query, Team.class)
            .getResultList();

    for (Team team : resultList) {
        System.out.println("team = " + team.getName() + " | members = " + team.getMembers().size());

        for (Member member : team.getMembers()) {
            System.out.println("-> member = " + member);
        }
    }
}


//
Hibernate: 
    /* SELECT
        DISTINCT t 
    FROM
        Team t 
    JOIN
        FETCH t.members */ select
            distinct team0_.id as id1_3_0_,
            members1_.id as id1_0_1_,
            team0_.name as name2_3_0_,
            members1_.age as age2_0_1_,
            members1_.memberType as memberTy3_0_1_,
            members1_.TEAM_ID as TEAM_ID5_0_1_,
            members1_.username as username4_0_1_,
            members1_.TEAM_ID as TEAM_ID5_0_0__,
            members1_.id as id1_0_0__ 
        from
            Team team0_ 
        inner join
            Member members1_ 
                on team0_.id=members1_.TEAM_ID
team = 팀A | members = 2
-> member = Member{id=3, username='멤버1', age=0}
-> member = Member{id=4, username='멤버2', age=0}
team = 팀B | members = 1
-> member = Member{id=5, username='멤버3', age=0}
```

JPQL에서 발생한 쿼리를 직접 실행해보면 결과가 다르다.  

SQL에서는 Distinct를 사용하더라도,  
두 테이블의 데이터가 다르기 때문에 중복제거가 불가능하다.  

![error](/assets/images/kyh-jpa-basic/33-sql-distinct.png)    

---

##### **페치 조인과 일반 조인의 차이**  
- JPQL은 결과를 반환할 때 연관관계를 고려하지 않는다.  
  단지 SELECT 절에 지정한 엔티티만 조회하는 것이다.  
- 일반조인은 실행시 조인은 하지만 연관된 엔티티를 조회하지 않는다.  
- 페치 조인을 사용할 때만 연관된 엔티티도 함께 조회한다. (즉시로딩)  
- 페치 조인은 객체 그래프를 SQL 한번에 조회하는 개념이다.  

```sql
[JPQL Inner Join]
SELECT t FROM Team t JOIN t.members m

[실제 실행된 SQL]
select
            team0_.id as id1_3_,
            team0_.name as name2_3_ 
        from
            Team team0_ 
        inner join
            Member members1_ 
                on team0_.id=members1_.TEAM_ID
```

```sql
[JPQL Fetch Join]
SELECT t FROM Team t JOIN FETCH t.members

[실제 실행된 SQL]
select
            team0_.id as id1_3_0_,
            members1_.id as id1_0_1_,
            team0_.name as name2_3_0_,
            members1_.age as age2_0_1_,
            members1_.memberType as memberTy3_0_1_,
            members1_.TEAM_ID as TEAM_ID5_0_1_,
            members1_.username as username4_0_1_,
            members1_.TEAM_ID as TEAM_ID5_0_0__,
            members1_.id as id1_0_0__ 
        from
            Team team0_ 
        inner join
            Member members1_ 
                on team0_.id=members1_.TEAM_ID
```

---

#### **페치 조인의 특징**
- 연관된 엔티티들을 SQL 한번으로 조회할 수 있어 성능 최적화를 할 수 있다.  
- 엔티티에 직접 적용되는 글로벌 로딩 전략보다 우선순위이다.  
    - @OneToMany(fetch = FetchType.LAZY) 
- 최적화가 필요한 곳은 페치 조인을 적용한다.  
- 객체 그래프를 유지할 때 사용하면 효과적이다.  
- 여러 테이블을 조인해서 엔티티와 모양이 전혀 다른 결과가 나온다면,  
  페치 조인을 사용하는 것보다 일반 조인을 사용하고,  
  필요한 데이터들만 조회해서 DTO로 반환하는 것이 오히려 효과적이다.  

---

##### **패치 조인의 한계**

###### **페치 조인의 대상에는 별칭을 주지 않는 것이 좋다.**
우선 JPA의 설계 의도는 Team.members를 조회했을 때  
모든 member를 조회할 수 있어야 하는데,  
아래의 쿼리는 그렇게 동작하지 않는다.  

이게 무슨 의미인지 코드를 통해 알아보자.  

```java
public static void main(String[] args) {
    Team team = new Team();
    team.setName("팀A");
    em.persist(team);

    Member member1 = new Member();
    member1.setUsername("멤버1");
    member1.setTeam(team);
    em.persist(member1);

    Member member2 = new Member();
    member2.setUsername("멤버2");
    member2.setTeam(team);
    em.persist(member2);

    String query = "SELECT t FROM Team t JOIN FETCH t.members m WHERE m.username = '멤버1'";

    List<Team> result = em.createQuery(query, Team.class)
                    .getResultList();
}
```

팀A는 멤버1과 멤버2를 가진다.  
그러나 페치조인의 대상인 t.members를 WHERE절으로 제한을 시켰다.  
이 결과를 출력해보면 팀A에게서 멤버2가 조회되지 않는다.  
객체 그래프에 문제가 생긴다는 걸 알 수 있다.  

이런 부분을 염려해 JPA는 페치조인의 별칭을 권장하지 않는다.  
(헷갈릴 수 있는데 페치조인의 대상이 아닌 Team에게 조건을 주는 것은 관계가 없다.)   
이런 경우엔 별도의 쿼리를 작성하는것이 더욱 안전하다.  

문제가 발생할 수 있으므로 가급적 사용하지 않는 것이 좋다.  

---

###### **둘 이상의 컬렉션은 페치 조인 할 수 없다.**  

---

###### **컬렉션을 페치 조인하면 페이징API를 사용할 수 없다.**  
```java
public static void main(String[] args) {
    String query = "SELECT t FROM Team t JOIN FETCH t.members";

    List<Team> resultList = em.createQuery(query, Team.class)
            .setFirstResult(0)
            .setMaxResults(1)
            .getResultList();

    for (Team team : resultList) {
        System.out.println("team = " + team.getName() + " | members = " + team.getMembers().size());

        for (Member member : team.getMembers()) {
            System.out.println("-> member = " + member);
        }
    }
}

//
WARN: HHH000104: firstResult/maxResults specified with collection fetch; applying in memory!
Hibernate: 
    /* SELECT
        t 
    FROM
        Team t 
    JOIN
        FETCH t.members */ select
            team0_.id as id1_3_0_,
            members1_.id as id1_0_1_,
            team0_.name as name2_3_0_,
            members1_.age as age2_0_1_,
            members1_.memberType as memberTy3_0_1_,
            members1_.TEAM_ID as TEAM_ID5_0_1_,
            members1_.username as username4_0_1_,
            members1_.TEAM_ID as TEAM_ID5_0_0__,
            members1_.id as id1_0_0__ 
        from
            Team team0_ 
        inner join
            Member members1_ 
                on team0_.id=members1_.TEAM_ID
team = 팀A | members = 2
-> member = Member{id=3, username='회원1', age=0}
-> member = Member{id=4, username='회원2', age=0}
```

먼저 Warnning이 발생했다.  

쿼리의 결과를 모두 메모리에 적재한 후  
애플리케이션 레벨에서 페이징 작업을 하기 때문에 위험하다는 경고 로그이다.  

쿼리를 확인 해보면 이전에 배운것과 다르게 limit가 설정되어있지 않다.  

이유는 역시 페이징 Team의 객체 그래프를 유지하지 못하기 때문이다.  

실제 쿼리를 보면서 이해해보자.  

```text 
전체 조회
```
![error](/assets/images/kyh-jpa-basic/34-fetch-paging-nolimit.png)    

```text 
LIMIT문 추가
```
![error](/assets/images/kyh-jpa-basic/35-fetch-paging-limit.png)    

팀A에 회원2가 없는 것처럼 보이게 되며  
팀A의 객체 그래프는 유지되지 못하는 것을 알 수 있다.  
이를 방지하기 위해 전체를 조회해서  
애플리케이션 레벨에서 페이징 작업을 하는 것이다.  

---

###### **fetch join paging 해결방법**  
**1. 일대일 다대일 같은 단일 값 연관 필드는 페치 조인해도 페이징이 가능하다.**
조회의 방향을 변경한다.  
Member를 조회하고 Team을 Fetch Join 하는 것이다.  
Member.team은 단일 객체이기 때문에 페치 조인을 해도 페이징 처리가 가능하다.  

```java
String query = "SELECT m FROM Member m JOIN FETCH m.team";

List<Member> resultList = em.createQuery(query, Member.class)
        .setFirstResult(0)
        .setMaxResults(1)
        .getResultList();
```

이 경우는 문제 없이 동작한다.  

**2. 페치 조인을 포기하고 페이징을 한다.**  

```java
String query = "SELECT t FROM Team t";

List<Team> resultList = em.createQuery(query, Team.class)
        .setFirstResult(0)
        .setMaxResults(2)
        .getResultList();

for (Team team : resultList) {
    System.out.println("team = " + team.getName() + " | members = " + team.getMembers().size());

    for (Member member : team.getMembers()) {
        System.out.println("-> member = " + member);
    }
}


//
Hibernate: 
    /* SELECT
        t 
    FROM
        Team t */ select
            team0_.id as id1_3_,
            team0_.name as name2_3_ 
        from
            Team team0_ limit ?
Hibernate: 
    select
        members0_.TEAM_ID as TEAM_ID5_0_0_,
        members0_.id as id1_0_0_,
        members0_.id as id1_0_1_,
        members0_.age as age2_0_1_,
        members0_.memberType as memberTy3_0_1_,
        members0_.TEAM_ID as TEAM_ID5_0_1_,
        members0_.username as username4_0_1_ 
    from
        Member members0_ 
    where
        members0_.TEAM_ID=?
team = 팀A | members = 2
-> member = Member{id=3, username='회원1', age=0}
-> member = Member{id=4, username='회원2', age=0}
Hibernate: 
    select
        members0_.TEAM_ID as TEAM_ID5_0_0_,
        members0_.id as id1_0_0_,
        members0_.id as id1_0_1_,
        members0_.age as age2_0_1_,
        members0_.memberType as memberTy3_0_1_,
        members0_.TEAM_ID as TEAM_ID5_0_1_,
        members0_.username as username4_0_1_ 
    from
        Member members0_ 
    where
        members0_.TEAM_ID=?
team = 팀B | members = 1
-> member = Member{id=5, username='회원3', age=0}
```

이렇게 하면 쉽게 해결은 가능하지만 성능상의 문제가 발생할 수 있다.  


**3. 페치 조인을 포기하고 @BatchSize를 사용해 페이징을 한다.**  
이럴 때 @BatchSize를 사용할 수 있다.  

```java
@Entity
public class Team {

    @Id @GeneratedValue
    private Long id;

    private String name;

    @BatchSize(size = 100)
    @OneToMany(mappedBy = "team")
    private List<Member> members = new ArrayList<>();

}
```

문제가 되는 컬렉션 위에 @BatchSize를 사용한다.  

```java
public static void main(String[] args) {
    String query = "SELECT t FROM Team t";

    List<Team> resultList = em.createQuery(query, Team.class)
            .setFirstResult(0)
            .setMaxResults(2)
            .getResultList();

    for (Team team : resultList) {
        System.out.println("team = " + team.getName() + " | members = " + team.getMembers().size());

        for (Member member : team.getMembers()) {
            System.out.println("-> member = " + member);
        }
    }
}


//
Hibernate: 
    /* SELECT
        t 
    FROM
        Team t */ select
            team0_.id as id1_3_,
            team0_.name as name2_3_ 
        from
            Team team0_ limit ?
Hibernate: 
    /* load one-to-many me.gicheol.jpql.Team.members */ select
        members0_.TEAM_ID as TEAM_ID5_0_1_,
        members0_.id as id1_0_1_,
        members0_.id as id1_0_0_,
        members0_.age as age2_0_0_,
        members0_.memberType as memberTy3_0_0_,
        members0_.TEAM_ID as TEAM_ID5_0_0_,
        members0_.username as username4_0_0_ 
    from
        Member members0_ 
    where
        members0_.TEAM_ID in (
            ?, ?
        )
team = 팀A | members = 2
-> member = Member{id=3, username='회원1', age=0}
-> member = Member{id=4, username='회원2', age=0}
team = 팀B | members = 1
-> member = Member{id=5, username='회원3', age=0}
```

먼저 Team에 limit을 설정하고 조회한다.  
두번째로 Member를 조회하는데  
이때 관계가 있는 Team을 IN절을 통해 한번에 모두 조회한다.  

아까 설정했던 @BatchSize의 옵션인 size가  
IN절 안에 들어갈 수 있는 ?의 개수이다.  

참고로 @BatchSize는 글로벌 설정도 가능하다.  
persistence.xml 파일에 아래와 같이 설정하면  
애노테이션을 사용하지 않아도 된다.  

```xml
<property name="hibernate.default_batch_fetch_size" value="100"/>
```

---

#### **다형성 쿼리**  
다음과 같은 상속관계의 테이블이 있다.  

![error](/assets/images/kyh-jpa-basic/18-inheritance-mapping.png)    

**Type**
이전에 공부했던 그 Type이다.  
조회 대상을 특정 자식으로 한정할 수 있다.  

ex. Item중에 Book, Movie를 조회한다.  

```sql
[JPQL]
SELECT i 
FROM Item i
WHERE TYPE (i) IN (Book, Movie)
```

```sql
[SQL]
SELECT i.*
FROM Item i
WHERE i.DTYPE IN ('B', 'M')
```

**Treat** 
자바의 타입 캐스팅과 유사하다.  
상속 구조에서 부모 타입을 특정 자식으로 다룰 때 사용한다.  

ex. 부모 Item과 자식 Book이 있다.  

```sql
[JPQL]
SELECT i 
FROM Item i
WHERE TREAT (i as Book).author = 'LEE'
```

```sql
[SQL]
SELECT i.*
FROM Item i
WHERE i.DTYPE = 'B' 
AND i.author = 'LEE'
```

---

#### **엔티티 직접 사용**
##### **기본 키 값**  
JPQL에서 엔티티를 직접 사용하면  
SQL에서 해당 엔티티의 기본 키 값을 사용한다.  

```sql
[JPQL]
SELECT COUNT(m.id) FROM Member m           -> 엔티티의 아이디를 사용
SELECT COUNT(m) FROM Member m              -> 엔티티를 직접 사용
```

```sql
[SQL]
SELECT COUNT(m.id) AS cnt FROM Member m    -> 동일한 SQL을 실행한다.
```


이런 경우에도 마찬가지이다.  

<br>

**엔티티를 파라미터로 전달**
```java
String query = "SELECT m FROM Member m WHERE m = :member";

Member findMember = em.createQuery(query, Member.class)
        .setParameter("member", member)
        .getSingleResult();
```

**식별자를 파라미터로 전달**
```java
String query = "SELECT m FROM Member m WHERE m.id = :memberId";

Member findMember = em.createQuery(query, Member.class)
        .setParameter("memberId", memberId)
        .getSingleResult();
```

**실행된 SQL**
```sql
SELECT m.* FROM Member m WHERE m.id = ?
```

---

##### **외래 키 값**  
여기서 외래키라 하면 @JoinColumn에 name으로 설정된 컬럼이다.  

**엔티티를 파라미터로 전달**
```java
String query = "SELECT m FROM Member m WHERE m.team = :team";

List<Member> resultList = em.createQuery(query, Member.class)
        .setParameter("team", member.getTeam())
        .getResultList();

for (Member findMember : resultList) {
    System.out.println("findMember = " + findMember);
}
```

**식별자를 파라미터로 전달**
```java
String query = "SELECT m FROM Member m WHERE m.team.id = :teamId";

List<Member> resultList = em.createQuery(query, Member.class)
        .setParameter("teamId", member1.getTeam().getId())
        .getResultList();

for (Member findMember : resultList) {
    System.out.println("findMember = " + findMember);
}
```

**실행된 SQL**
```sql
SELECT m.* FROM Member m WHERE m.team_id = ?
```

---

#### **Named 쿼리**
네임드 쿼리는 쿼리에 이름을 주어 사용할 수 있는 방식이다.  
Annotation이나 XML에 설정해서 사용하며,  
미리 선언했기 때문에 동적쿼리는 불가능하다.  

애플리케이션 로딩 시점에 초기화 후 재사용을 하므로 파싱하는 비용을 줄일 수 있다.  

가장 큰 장점은 JPQL은 쿼리를 문자열로 사용하다보니,  
오타를 내서 발생한 에러는 잡아내기 쉽지 않다  
네임드 쿼리는 애플리케이션 로딩 시점에 초기화 되기 때문에  
오타 발생 시 컴파일에러가 발생하게 되어 쿼리를 검증할 수 있다.  

---

##### **Named 쿼리 예제**  
Annotation 설정

```java
@Entity
@NamedQuery(
        name = "Member.findByUsername",
        query = "SELECT m FROM Member m WHERE m.username = :username"
)
public class Member {

    @Id @GeneratedValue
    private Long id;

    private String username;

    private int age;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "TEAM_ID")
    private Team team;

}


public class Main {
    public static void main(String[] args) {
        List<Member> resultList = em.createNamedQuery("Member.findByUsername", Member.class)
                    .setParameter("username", "회원1")
                    .getResultList();

        for (Member member : resultList) {
            System.out.println("member = " + member);
        }
    }
}


//
Hibernate: 
    /* Member.findByUsername */ select
        member0_.id as id1_0_,
        member0_.age as age2_0_,
        member0_.memberType as memberTy3_0_,
        member0_.TEAM_ID as TEAM_ID5_0_,
        member0_.username as username4_0_ 
    from
        Member member0_ 
    where
        member0_.username=?
member = Member{id=3, username='회원1', age=0}
```


XML 설정

```xml
[ META-INF/persistence.xml ]
...

<persistence-unit name="hello">
        <mapping-file>META-INF/ormMember.xml</mapping-file>

...
```

```xml
[ META-INF/ormMember.xml ]
<?xml version="1.0" encoding="UTF-8"?>
<entity-mappings xmlns="http://xmlns.jcp.org/xml/ns/persistence/orm" version="2.1">
    <named-query name="Member.findByUsername">
        <query>
            <![CDATA[
                SELECT m
                FROM Member m
                WHERE m.username = :username
            ]]>
        </query>
    </named-query>
</entity-mappings>
```

만약 두 가지가 모두 설정이 되어 있다면 XML이 우선순위를 가진다.  
애플리케이션 운영 환경에 따라 쿼리가 다른 부분이 있다면  
각각 다른 XML을 배포할 수 있다.  

---

##### **컴파일 시 쿼리 검증**  
```java
@Entity
@NamedQuery(
        name = "Member.findByUsername",
        query = "SELECT mmm FROM Memberr m WHERE m.username = :username"
)
public class Member {

    @Id @GeneratedValue
    private Long id;

    private String username;

    private int age;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "TEAM_ID")
    private Team team;

}


//
ERROR: HHH000177: Error in named query: Member.findByUsername
org.hibernate.hql.internal.ast.QuerySyntaxException: Memberr is not mapped [SELECT mmm FROM Memberr m WHERE m.username = :username]
	at org.hibernate.hql.internal.ast.QuerySyntaxException.generateQueryException(QuerySyntaxException.java:79)
	at org.hibernate.QueryException.wrapWithQueryString(QueryException.java:103)
	at org.hibernate.hql.internal.ast.QueryTranslatorImpl.doCompile(QueryTranslatorImpl.java:219)
	at org.hibernate.hql.internal.ast.QueryTranslatorImpl.compile(QueryTranslatorImpl.java:143)
	at org.hibernate.engine.query.spi.HQLQueryPlan.<init>(HQLQueryPlan.java:119)
	at org.hibernate.engine.query.spi.HQLQueryPlan.<init>(HQLQueryPlan.java:80)
	at org.hibernate.engine.query.spi.QueryPlanCache.getHQLQueryPlan(QueryPlanCache.java:153)
	at org.hibernate.query.spi.NamedQueryRepository.checkNamedQueries(NamedQueryRepository.java:157)
	at org.hibernate.internal.SessionFactoryImpl.checkNamedQueries(SessionFactoryImpl.java:574)
	at org.hibernate.internal.SessionFactoryImpl.<init>(SessionFactoryImpl.java:321)
	at org.hibernate.boot.internal.SessionFactoryBuilderImpl.build(SessionFactoryBuilderImpl.java:467)
	at org.hibernate.jpa.boot.internal.EntityManagerFactoryBuilderImpl.build(EntityManagerFactoryBuilderImpl.java:939)
	at org.hibernate.jpa.HibernatePersistenceProvider.createEntityManagerFactory(HibernatePersistenceProvider.java:56)
	at javax.persistence.Persistence.createEntityManagerFactory(Persistence.java:79)
	at javax.persistence.Persistence.createEntityManagerFactory(Persistence.java:54)
	at me.gicheol.jpql.JpaMain.main(JpaMain.java:9)
Caused by: org.hibernate.hql.internal.ast.QuerySyntaxException: Memberr is not mapped
	at org.hibernate.hql.internal.ast.util.SessionFactoryHelper.requireClassPersister(SessionFactoryHelper.java:169)
	at org.hibernate.hql.internal.ast.tree.FromElementFactory.addFromElement(FromElementFactory.java:91)
	at org.hibernate.hql.internal.ast.tree.FromClause.addFromElement(FromClause.java:79)
	at org.hibernate.hql.internal.ast.HqlSqlWalker.createFromElement(HqlSqlWalker.java:331)
	at org.hibernate.hql.internal.antlr.HqlSqlBaseWalker.fromElement(HqlSqlBaseWalker.java:3695)
	at org.hibernate.hql.internal.antlr.HqlSqlBaseWalker.fromElementList(HqlSqlBaseWalker.java:3584)
	at org.hibernate.hql.internal.antlr.HqlSqlBaseWalker.fromClause(HqlSqlBaseWalker.java:720)
	at org.hibernate.hql.internal.antlr.HqlSqlBaseWalker.query(HqlSqlBaseWalker.java:576)
	at org.hibernate.hql.internal.antlr.HqlSqlBaseWalker.selectStatement(HqlSqlBaseWalker.java:313)
	at org.hibernate.hql.internal.antlr.HqlSqlBaseWalker.statement(HqlSqlBaseWalker.java:261)
	at org.hibernate.hql.internal.ast.QueryTranslatorImpl.analyze(QueryTranslatorImpl.java:271)
	at org.hibernate.hql.internal.ast.QueryTranslatorImpl.doCompile(QueryTranslatorImpl.java:191)
	... 13 more
```

실수로 인해서 @NamedQuery의 쿼리에 오타를 작성했다.  
쿼리는 문자열이기 때문에 애플리케이션 실행할때 에러가 발생하지 않고,  
해당 쿼리를 실행할 시점에 에러가 발생한다.  

그러나 @NamedQuery는 로딩 시점에 쿼리 검증을 하기 때문에  
로딩 과정에서 쿼리에 문제가 있다며 QuerySyntaxException이 발생한다.  

---

#### **벌크 연산**  
UPDATE, DELETE문을 한번에 여러번 동작하는 것을 벌크 연산이라고 한다.  
JPA는 Dirty Checking을 하기 때문에  
변경된 데이터가 100건이라면 100번의 UPDATE문을 실행한다.  

이처럼 JPA는 실시간성의 쿼리에 최적화 되어있지만  
벌크 연산도 제공을 한다.  

##### **벌크 연산 예제**  
```java
public static void main(String[] args) {
    Member member1 = new Member();
    member1.setUsername("회원1");
    member1.setAge(26);
    em.persist(member1);

    Member member2 = new Member();
    member2.setUsername("회원2");
    member2.setAge(36);
    em.persist(member2);

    Member member3 = new Member();
    member3.setUsername("회원3");
    member3.setAge(46);
    em.persist(member3);


    String query = "UPDATE Member m SET m.age = 20";

    int resultCount = em.createQuery(query)
            .executeUpdate();

    System.out.println("resultCount = " + resultCount);
}

//
Hibernate: 
    /* UPDATE
        Member m 
    SET
        m.age = 20 */ update
            Member 
        set
            age=20
resultCount = 3
```

이때 반환되는 resultCount는 업데이트 된 데이터의 개수이다.  

![error](/assets/images/kyh-jpa-basic/36-bulk.png)    

업데이트가 정상적으로 잘 된 것을 알 수 있다.  

벌크연산은 UPDATE, DELETE 말고도  
INSERT SELECT문도 지원한다.  

---

##### **벌크 연산 주의사항**  
벌크 연산은 영속성 컨텍스트를 무시하고 데이터베이스에 직접 쿼리한다.  
그렇기 때문에 DB에는 업데이트가 됬지만  
애플리케이션 상에서는 업데이트가 되지 않는 경우가 발생할 수 있다.   
주의해서 사용해야한다.  

아래 코드를 통해 무슨 말인지 이해하자.  

```java
public static void main(String[] args) {
    Member member1 = new Member();
    member1.setUsername("회원1");
    member1.setAge(26);
    em.persist(member1);

    Member member2 = new Member();
    member2.setUsername("회원2");
    member2.setAge(36);
    em.persist(member2);

    Member member3 = new Member();
    member3.setUsername("회원3");
    member3.setAge(46);
    em.persist(member3);


    String query = "UPDATE Member m SET m.age = 20";

    int resultCount = em.createQuery(query)
            .executeUpdate();

    System.out.println("resultCount = " + resultCount);

    Member findMember = em.find(Member.class, member1.getId());
    System.out.println("findMember.getAge() = " + findMember.getAge());
}

//
Hibernate: 
    /* UPDATE
        Member m 
    SET
        m.age = 20 */ update
            Member 
        set
            age=20
resultCount = 3
findMember.getAge() = 26
```

업데이트는 정상적으로 수행했으나,  
findMember를 새로 조회해오더라도 조회 대상인 member1은  
이미 영속성 컨텍스트에 올라가 있던 엔티티 이기 때문에    
findMember의 age를 조회하면 변경이 되기 전의 값이 26이 나온다.  

해결을 하기 위한 방법은 두 가지가 있다.  

**어떠한 영속성 컨텍스트 작업을 하기 전 벌크 연산을 먼저 실행하거나,  
벌크 연산을 수행한 후 영속성 컨텍스트를 초기화 해야한다.**

```java
public static void main(String[] args) {
    
    // ... 위와 동일함

    String query = "UPDATE Member m SET m.age = 20";

    int resultCount = em.createQuery(query)
            .executeUpdate();

    System.out.println("resultCount = " + resultCount);

    em.clear();

    Member findMember = em.find(Member.class, member1.getId());

    System.out.println("findMember = " + findMember.getAge());

}

//
Hibernate: 
    /* UPDATE
        Member m 
    SET
        m.age = 20 */ update
            Member 
        set
            age=20
resultCount = 3
Hibernate: 
    select
        member0_.id as id1_0_0_,
        member0_.age as age2_0_0_,
        member0_.memberType as memberTy3_0_0_,
        member0_.TEAM_ID as TEAM_ID5_0_0_,
        member0_.username as username4_0_0_ 
    from
        Member member0_ 
    where
        member0_.id=?
findMember = 20
```

어차피 em.createQuery(query).executeUpdate() 을 하는 시점에서  
flush는 발생하기 때문에 굳이 해주지 않아도 된다.  

업데이트가 완료된 후 em.clear()를 해줌으로써  
영속성 컨텍스트를 비워준다.  

그리고 em.find()를 실행하면 영속성 컨텍스트에 엔티티가 없기 때문에  
새로 DB에서 조회를 해오기 때문에 아예 새로운 엔티티가 조회된다.  

그렇기 때문에 변경된 값이 출력이 되는 것이고,  
출력하기 전에 SELECT문이 발생하는 것이다.  

잘 모른다면 고생하기 딱 좋은 장애인 것 같다.  
주의하도록 하자.  

---