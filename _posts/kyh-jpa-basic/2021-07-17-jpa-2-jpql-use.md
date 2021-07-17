---
title : "JPA JPQL 사용"
categories : 
    - jpa
tag :
    - jpa
toc : true
---

[김영한 : 자바 ORM 표준 JPA 프로그래밍 - 기본편](https://www.inflearn.com/course/ORM-JPA-Basic#curriculum) 강의를 공부한 내용입니다.  

---

### **JPQL 문법**  
SELECT m FROM Member as m WHERE m.age > 18
이전에 JPQL은 쿼리의 대상이 테이블이 아닌, 엔티티라고 공부했다.  
SELECT, FROM 등의 키워드는 대소문자 구분을 하지 않으나,  
엔티티와 속성은 대소문자를 구분한다.  

Alias를 주는 것 역시 필수이다.  

---

#### **반환 타입, 바인딩**  

##### **TypeQuery, Query**  
EntityManager.createQuery()의 반환 타입이다.  

- TypeQuery : 반환타입이 명확 할 경우 사용한다.  

```java
TypedQuery<Member> query = 
        em.createQuery("SELECT m FROM Member m", Member.class);
```

- Query : 반환 타입이 명확하지 않을 경우 사용한다.  

```java
Query query = 
        em.createQuery("SELECT m.username, m.age FROM Member m");
```

---

##### **getResultList, getSingleResult**  
TypeQuery 혹은 Query로 반환된 객체를 통해 리스트 또는 단일 객체를 반환받을 수 있다.  

- query.getResultList : 쿼리의 결과가 하나 이상일 때 리스트 반환  
    - 결과가 없으면 빈 리스트 반환  
- query.getSingleResult : 쿼리의 결과가 정확히 하나일때 단일 객체 반환
    - 결과가 없으면 : javax.persistence.NoResultException  
    - 둘 이상이면 : javax.persistence.NonUniqueResultException  

getSingleResult는 결과가 없더라도 예외가 발생하기 때문에 주의해서 사용해야 한다.  
Spring Data JPA는 결과가 없다면 null을 반환한다.  

---

##### **파라미터 바인딩**  
파라미터 바인딩을 해줄 때 이름을 기준으로, 위치를 기준으로 줄 수 있다.  
위치를 기준으로 바인딩을 하면 실수가 발생할 수 있기 때문에  
이름을 기준으로 바인딩 하는 것이 좋다.  

- 이름 기준
```java
Member singleResult = em.createQuery(
                    "SELECT m FROM Member m WHERE m.username = :username", Member.class)
                                    .setParameter("username", "LEE")
                                    .getSingleResult();

System.out.println("singleResult.getUsername() = " + singleResult.getUsername());
```

- 위치 기준
```java
Member singleResult = em.createQuery(
                    "SELECT m FROM Member m WHERE m.username = :1", Member.class)
                                    .setParameter(1, "LEE")
                                    .getSingleResult();

System.out.println("singleResult.getUsername() = " + singleResult.getUsername());
```

---

#### **프로젝션**  
프로젝션이란 SELECT절에 조회할 대상을 지정하는 것을 말한다.  

프로젝션의 대상은 엔티티, 임베디드 타입, 스칼라 타입이 있다.  
(스칼라 타입 : 숫자, 문자 등 기본 데이터 타입)  

- SELECT m FROM Member m : 엔티티 프로젝션

```java
public static void main(String[] args) {
    Member member = new Member();
    member.setUsername("LEE");
    member.setAge(26);
    em.persist(member);

    em.flush();
    em.clear();

    List<Member> result = em.createQuery(
            "SELECT m FROM Member m", Member.class)
            .getResultList();

    Member findMember = result.get(0);
    findMember.setAge(30);
}
```

getResultList()를 통해 조회해온 List\<Member\>는 모두 영속성 컨텍스트에서 관리된다.  

그 증거로 flush, clear로 영속성 컨텍스트를 비운 후 받아온 reulst에서  
값을 변경하더라도 UPDATE 쿼리가 발생한다.  

<br>  

- SELECT m.team FROM Member m : 엔티티 프로젝션

JPQL은 아래와 같은 쿼리가 가능해 굉장히 간편하다.  

```java
List<Team> resultList = em.createQuery(
            "SELECT m.team FROM Member m", Team.class)
        .getResultList();
```

그러나 사실 이 문법으로 실제 SQL은 조인이 발생하는데,  
코드 상으로 알아보기가 쉽진 않다.  

그렇기 때문에 아래와 같이 명시적으로 조인을 해주도록 한다.  

```java
List<Team> resultList = em.createQuery(
            "SELECT m.team FROM Member m JOIN m.team", Team.class)
        .getResultList();
```

- SELECT m.address FROM Member m : 임베디드 타입 프로젝션  

아래와 같이 임베디드 타입으로 객체를 반환 받을 수 있다.  

```java
List<Address> resultList = em.createQuery(
            "SELECT o.address FROM Order o", Address.class)
        .getResultList();
```

- SELECT m.username, m.age FROM Member m : 스칼라 타입 프로젝션

아래와 같이 기본 데이터 타입을 조회할 수 있다.  

```java
em.createQuery("SELECT m.username, m.age FROM Member m");
```

그런데 이런 쿼리는 어떤 객체로 반환받아야 할까?  

---

##### **스칼라 타입 프로젝션 반환 객체**  
1. Query 타입  
```java
Query query = 
    em.createQuery("SELECT m.username, m.age FROM Member m");
```

2. List<Object[]>  

```java
String query = "SELECT m.username, m.age FROM Member m";

List<Object[]> resultList = em.createQuery(query, Object[].class)
        .getResultList();

Object[] result = resultList.get(0);

System.out.println("username = " + result[0]);
System.out.println("age = " + result[1]);
```

3. new 키워드  
```java
public class MemberDto {

    public MemberDto(String username, int age) {
        this.username = username;
        this.age = age;
    }

    private String username;

    private int age;

}

public class JpaMain {

    public static void main(String[] args) {
        String query = "SELECT new me.gicheol.jpql.MemberDto(m.username, m.age) FROM Member m";

        List<MemberDto> resultList = 
            em.createQuery(query, MemberDto.class)
                    .getResultList();

        MemberDto memberDto = resultList.get(0);
        System.out.println("memberDto.getUsername() = " + memberDto.getUsername());
        System.out.println("memberDto.getAge() = " + memberDto.getAge());
    }

}
```

new 키워드를 사용하고, 반환받을 컬럼에 대한 DTO 클래스의 패키지를 적어준다.  
마치 생성자 호출하듯 사용하는 것이다.  

실제로 생성자를 호출하는 것이라, DTO 클래스에 해당하는 생성자가 필요하다.  


사실상 DTO를 생성하고 new 키워드를 통해  
객체를 반환받는 것이 가장 가독성이 좋은 방법이지만  
DTO클래스의 패키지를 전부 다 적어줘야한다는 점은 조금 아쉬운 부분이다.  

이런 부분도 QueryDSL을 사용하면 해결할 수 있다.  

---

#### **페이징**  
페이징 쿼리를 만드는건 상당히 귀찮은 작업이다.  
JPQL은 페이징 쿼리도 지원한다.  

```java
public static void main(String[] args) {
    for (int i = 0; i < 100; i++) {
        Member member = new Member();
        member.setUsername("LEE" + i);
        member.setAge(i);
        em.persist(member);
    }

    em.flush();
    em.clear();

    String query = "SELECT m FROM Member m ORDER BY m.age DESC";

    List<Member> resultList = em.createQuery(query, Member.class)
            .setFirstResult(1)
            .setMaxResults(10)
            .getResultList();

    System.out.println("resultList.size() = " + resultList.size());
    resultList.forEach(System.out::println);
}

// MySQL , H2
Hibernate: 
    /* SELECT
        m 
    FROM
        Member m 
    ORDER BY
        m.age DESC */ select
            member0_.id as id1_0_,
            member0_.age as age2_0_,
            member0_.TEAM_ID as TEAM_ID4_0_,
            member0_.username as username3_0_ 
        from
            Member member0_ 
        order by
            member0_.age DESC limit ? offset ?

// Oracle
Hibernate: 
    /* SELECT
        m 
    FROM
        Member m 
    ORDER BY
        m.age DESC */ select
            * 
        from
            ( select
                member0_.id as id1_0_,
                member0_.age as age2_0_,
                member0_.TEAM_ID as TEAM_ID4_0_,
                member0_.username as username3_0_ 
            from
                Member member0_ 
            order by
                member0_.age DESC ) 
        where
            rownum <= ?

// 결과
resultList.size() = 10
Member{id=99, username='LEE98', age=98}
Member{id=98, username='LEE97', age=97}
Member{id=97, username='LEE96', age=96}
Member{id=96, username='LEE95', age=95}
Member{id=95, username='LEE94', age=94}
Member{id=94, username='LEE93', age=93}
Member{id=93, username='LEE92', age=92}
Member{id=92, username='LEE91', age=91}
Member{id=91, username='LEE90', age=90}
Member{id=90, username='LEE89', age=89}
```

setFirstResult(조회할 페이지)  
setMaxResults(조회할 데이터의 개수)  

MySQL이나 H2의 쿼리는 그나마 괜찮지만,  
Oracle의 쿼리는 상당하다...   

페이징 쿼리를 굉장히 간편하게 사용할 수 있는 문법이다.  

---

#### **조인**  

##### **내부 조인**  
```sql
SELECT m FROM Member m [INNER] JOIN m.team t
```

INNER는 생략 가능하다.  


```java
public static void main(String[] args) {
    Team team = new Team();
    team.setName("LEE TEAM");
    em.persist(team);

    Member member = new Member();
    member.setUsername("LEE");
    member.setAge(26);
    member.changeTeam(team);

    em.persist(member);

    em.flush();
    em.clear();

    String query = "SELECT m FROM Member m INNER JOIN m.team t";
    List<Member> resultList = em.createQuery(query, Member.class)
            .getResultList();

    resultList.forEach(System.out::println);
}

//
Hibernate: 
    /* SELECT
        m 
    FROM
        Member m 
    INNER JOIN
        m.team t */ select
            member0_.id as id1_0_,
            member0_.age as age2_0_,
            member0_.TEAM_ID as TEAM_ID4_0_,
            member0_.username as username3_0_ 
        from
            Member member0_ 
        inner join
            Team team1_ 
                on member0_.TEAM_ID=team1_.id

Member{id=2, username='LEE', age=26}
```

이때 주의할 점은 Member 엔티티에 연관관계가 있는  
Team에 fetch 옵션이 즉시로딩(Eager)으로 되어있다면  
위의 코드에서 SELECT 쿼리가 두번 발생한다.  

꼭 지연로딩(Lazy)으로 변경해주자.  

```java
@Entity
public class Member {

    @Id @GeneratedValue
    private Long id;

    private String username;

    private int age;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "TEAM_ID")
    private Team team;

}
```
---

##### **외부 조인**  
```sql
SELECT m FROM Member m LEFT [OUTER] JOIN m.team t
```

OUTER는 생략 가능하다.  

```java
public static void main(String[] args) {
    Team team = new Team();
    team.setName("LEE TEAM");
    em.persist(team);

    Member member = new Member();
    member.setUsername("LEE");
    member.setAge(26);
    member.changeTeam(team);

    em.persist(member);

    em.flush();
    em.clear();

    String query = "SELECT m FROM Member m LEFT OUTER JOIN m.team t";
    List<Member> resultList = em.createQuery(query, Member.class)
            .getResultList();

    resultList.forEach(System.out::println);

//
Hibernate: 
    /* SELECT
        m 
    FROM
        Member m 
    LEFT OUTER JOIN
        m.team t */ select
            member0_.id as id1_0_,
            member0_.age as age2_0_,
            member0_.TEAM_ID as TEAM_ID4_0_,
            member0_.username as username3_0_ 
        from
            Member member0_ 
        left outer join
            Team team1_ 
                on member0_.TEAM_ID=team1_.id

Member{id=2, username='LEE', age=26}
}
```

마찬가지로 fetch 옵션이 즉시로딩으로 설정되어 있다면  
SELECT 쿼리가 두번 발생하니 꼭! 주의하도록 한다.  

---

##### **세타 조인**  
세타 조인은 연관관계가 없는 테이블간의 조인을 의미한다.  

```sql
SELECT COUNT(m) FROM Member m, Team t WHERE m.username = t.name
```

```java
public static void main(String[] args) {
    Team team = new Team();
    team.setName("LEE");
    em.persist(team);

    Member member = new Member();
    member.setUsername("LEE");
    member.setAge(26);
    member.changeTeam(team);

    em.persist(member);

    em.flush();
    em.clear();

    String query = "SELECT m FROM Member m, Team t WHERE m.username = t.name";
    List<Member> resultList = em.createQuery(query, Member.class)
            .getResultList();

    resultList.forEach(System.out::println);
}

//
Hibernate: 
    /* SELECT
        m 
    FROM
        Member m,
        Team t 
    WHERE
        m.username = t.name */ select
            member0_.id as id1_0_,
            member0_.age as age2_0_,
            member0_.TEAM_ID as TEAM_ID4_0_,
            member0_.username as username3_0_ 
        from
            Member member0_ cross 
        join
            Team team1_ 
        where
            member0_.username=team1_.name

Member{id=2, username='LEE', age=26}
```

---

##### **ON 절**  

JPA 2.1부터 ON절을 활용한 조인이 가능해져 조인 대상을 필터링 할 수 있다.  

Hibernate 5.1부터 세타조인과 같은 연관관계가 없는 엔티티를 외부 조인 할 수 있다.  
(기존에는 내부조인만 가능했다.)  

---

###### **ON절이란?**  
ON절은 테이블을 조인할 때 범위를 조건으로 주며,  
WHERE절은 조인을 한 후에 조건을 주어 최종 결과를 내기 위한 키워드이다.  

---

###### **조인 대상 필터링**  
ex) 회원과 팀을 조인하면서, 팀 이름이 A인 팀만 조인  

```sql
JPQL
SELECT m, t 
FROM Member m 
LEFT JOIN m.team t 
ON t.name = 'A'
```

```sql
SQL
SELECT m.*, t.* 
FROM Member m 
LEFT JOIN Team t 
ON m.team_id = t.id 
AND t.name = 'A'
```


```java
public static void main(String[] args) {
    String query = "SELECT m FROM Member m LEFT JOIN m.team t on t.name = 'LEE'";
   
    List<Member> resultList = em.createQuery(query, Member.class)
            .getResultList();
}

//
Hibernate: 
    /* SELECT
        m 
    FROM
        Member m 
    LEFT JOIN
        m.team t 
            on t.name = 'LEE' */ select
                member0_.id as id1_0_,
                member0_.age as age2_0_,
                member0_.TEAM_ID as TEAM_ID4_0_,
                member0_.username as username3_0_ 
        from
            Member member0_ 
        left outer join
            Team team1_ 
                on member0_.TEAM_ID=team1_.id 
                and (
                    team1_.name='LEE'
                )
```

---

###### **연관관계가 없는 엔티티 외부 조인**
ex) 회원의 이름과 팀의 이름이 같은 대상 외부 조인  

```sql
JPQL
SELECT m, t 
FROM Member m 
LEFT JOIN Team t 
ON m.username = t.name
```

```sql
SQL
SELECT m.*, t.* 
FROM Member m 
LEFT JOIN Team t 
ON m.username = t.name
```

```java
public static void main(String[] args) {
    String query = "SELECT m FROM Member m LEFT JOIN Team t on m.username = t.name";
    
    List<Member> resultList = em.createQuery(query, Member.class)
        .getResultList();
}

//
Hibernate: 
    /* SELECT
        m 
    FROM
        Member m 
    LEFT JOIN
        Team t 
            on m.username = t.name */ select
                member0_.id as id1_0_,
                member0_.age as age2_0_,
                member0_.TEAM_ID as TEAM_ID4_0_,
                member0_.username as username3_0_ 
        from
            Member member0_ 
        left outer join
            Team team1_ 
                on (
                    member0_.username=team1_.name
                )
```

---

#### **서브 쿼리**  

##### **서브 쿼리 지원 함수**  
- [NOT] EXISTS (subquery) : 서브쿼리에 결과가 존재하면 참  

- { ALL | ANY | SOME } (subquery)  
    - ALL : 모두 만족하면 참  
    - ANY, SOME : 같은 의미, 조건을 하나라도 만족하면 참   

- [NOT] IN (subquery) : 서브쿼리의 결과 중 하나라도 같은 것이 있으면 참  

---

##### **JPA 서브 쿼리의 한계**  
JPA는 WHERE, HAVING절에서만 서브쿼리를 사용할 수 있다.  
Hibernate에서는 SELECT절에서도 서브쿼리를 사용할 수 있다.  
대부분의 경우 Hibernate를 구현한 JPA를 쓰기 때문에  
JPA에서 SELECT절에 서브쿼리도 가능하다고 볼 수 있다.  

다만 FROM절의 서브쿼리는 JPQL에서 사용 불가능하다.  
가능하다면 최대한 조인으로 해결해야 한다.  

해결이 불가하다면 소스코드 상에서 해결하거나, 쿼리를 두번 날려서,  
혹은 Native SQL을 사용해서 해결해야 한다.  

---

#### **JPQL의 타입 표현**  
- 문자 : 'HELLO', 'She''s' (She's)  
- 숫자 : 10L (Long), 10D (Double), 10F (Float)  
- Boolean : TRUE, FALSE  
- Enum : me.gicheol.jpql.MemberType.USER (패키지명 포함)  
- 엔티티 타입 : TYPE (m) = Member (상속 관계에서 사용)  

---

##### **JPQL 타입 표현 예시 (문자, 숫자)**  
```java
public static void main(String[] args) {
    String query = "SELECT m.username, 'HELLO', 'She''s', TRUE FROM Member m";
            
    List<Object[]> resultList = em.createQuery(query, Object[].class)
            .getResultList();

    for (Object[] result : resultList) {
        System.out.println("result[0] = " + result[0]);
        System.out.println("result[1] = " + result[1]);
        System.out.println("result[2] = " + result[2]);
        System.out.println("result[3] = " + result[3]);
    }
}

//
result[0] = LEE
result[1] = HELLO
result[2] = She's
result[3] = true
```

---


##### **JPQL 타입 표현 예시 (Enum)**  
```java
public static void main(String[] args) {
    String query = "SELECT m.username FROM Member m " +
                   "WHERE m.memberType = me.gicheol.jpql.MemberType.USER";
            
    List<Object[]> resultList = em.createQuery(query, Object[].class)
            .getResultList();

    for (Object[] result : resultList) {
        System.out.println("result[0] = " + result[0]);
    }
}

//
Hibernate: 
    /* SELECT
        m.username
    FROM
        Member m 
    WHERE
        m.memberType = :memberType */ select
            member0_.username as col_0_0_
        from
            Member member0_ 
        where
            member0_.memberType=?

result[0] = LEE
```

Enum의 경우 패키지명을 전부 작성해야 하니 불편하다.  
파라미터 바인딩을 통해 조금 더 편하게 작성할 수 있다.  

```java
String query = "SELECT m.username FROM Member m " +
               "WHERE m.memberType = :memberType";

List<Object[]> resultList = em.createQuery(query, Object[].class)
                .setParameter("memberType", MemberType.USER)
                .getResultList();
```

---


##### **JPQL 타입 표현 예시 (Entity)**  
엔티티 타입의 경우 상속관계의 엔티티에 사용할 수 있다.  
예를들어 Item 엔티티를 상속받은 Book 엔티티가 있다고 치자.  
아래와 같이 다형성을 사용할 수 있다.  

```java
public static void main(String[] args) {
    Book book = new Book();
    book.setName("JPA");
    book.setAuthor("LEE");

    em.persist(book);

    List<Item> resultList = em.createQuery("SELECT i FROM Item i WHERE TYPE(i) = Book", Item.class)
            .getResultList();
}
```

---

##### **JPQL 기타 문법**  
- EXISTS, IN
- AND, OR, NOT
- =, >, >=, <, <=, <>
- BETWEEN, LIKE, IS NULL

SQL 문법은 대부분 사용 가능하다.  

---

#### **조건식 - CASE식**  
- 기본 CASE식 : 조건을 사용할 수 있다.  

```java
String query =
                "SELECT " +
                "    CASE WHEN m.age <= 10 THEN '학생요금' " +
                "         WHEN m.age >= 60 THEN '경로요금' " +
                "         ELSE '일반요금' " +
                "    END " +
                "FROM Member m";

List<String> resultList = em.createQuery(query, String.class)
        .getResultList();

for (String result : resultList) {
    System.out.println("result = " + result);
}

//
Hibernate: 
    /* SELECT
        CASE 
            WHEN m.age <= 10 THEN '학생요금'          
            WHEN m.age >= 60 THEN '경로요금'          
            ELSE '일반요금'     
        END 
    FROM
        Member m */ select
            case 
                when member0_.age<=10 then '학생요금' 
                when member0_.age>=60 then '경로요금' 
                else '일반요금' 
            end as col_0_0_ 
        from
            Member member0_

result = 일반요금
```

- 단순 CASE식 : 정확히 맞아야 true가 된다.

```java
String query =
                "SELECT " +
                "    CASE t.name " +
                "         WHEN '팀A' THEN '인센티브 110%' " +
                "         WHEN '팀B' THEN '인센티브 120%' " +
                "         ELSE '인센티브 105%' " +
                "    END " +
                "FROM Team t";

List<String> resultList = em.createQuery(query, String.class)
        .getResultList();

for (String result : resultList) {
    System.out.println("result = " + result);
}

//
Hibernate: 
    /* SELECT
        CASE t.name          
            WHEN '팀A' THEN '인센티브 110%'          
            WHEN '팀B' THEN '인센티브 120%'          
            ELSE '인센티브 105%'     
        END 
    FROM
        Team t */ select
            case team0_.name 
                when '팀A' then '인센티브 110%' 
                when '팀B' then '인센티브 120%' 
                else '인센티브 105%' 
            end as col_0_0_ 
        from
            Team team0_

result = 인센티브 105%
```

- COALESCE : 하나씩 조회해서 Null이 아니면 반환  

```java
public static void main(String[] args) {
    Member member = new Member();
    member.setAge(26);
    
    em.persist(member);

    String query = "SELECT COALESCE(m.username, '이름 없는 회원') FROM Member m";

    List<String> resultList = em.createQuery(query, String.class)
            .getResultList();

    for (String result : resultList) {
        System.out.println("result = " + result);
    }
}

//
Hibernate: 
    /* SELECT
        COALESCE(m.username,
        '이름 없는 회원') 
    FROM
        Member m */ select
            coalesce(member0_.username,
            '이름 없는 회원') as col_0_0_ 
        from
            Member member0_

result = 이름 없는 회원
```

- NULLIF : 두 값이 같으면 Null 반환, 다르면 첫 번째 값 반환  

```java
public static void main(String[] args) {
    Member member = new Member();
    member.setUsername("관리자");

    em.persist(member);

    String query = "SELECT NULLIF(m.username, '관리자') FROM Member m";

    List<String> resultList = em.createQuery(query, String.class)
            .getResultList();

    for (String result : resultList) {
        System.out.println("result = " + result);
    }
}

//
Hibernate: 
    /* SELECT
        NULLIF(m.username,
        '관리자') 
    FROM
        Member m */ select
            nullif(member0_.username,
            '관리자') as col_0_0_ 
        from
            Member member0_

result = null
```

---

#### **JPQL 함수**  

##### **JPQL 기본 함수**  
- CONCAT
- SUBSTRING
- TRIM
- LOWER, UPPER
- LENGTH
- LOCATE
- ABS, SQRT, MOD
- SIZE (엔티티의 컬렉션의 size), INDEX

---

##### **JPQL 사용자 정의 함수**  
사용자 정의 함수를 사용하기 위해서는 직접 함수를 추가해줘야한다.  

먼저 H2 DB 기준으로 H2Dialect 클래스를 상속받고, 생성자에 함수를 추가해준다.  

```java
public class MyH2Dialect extends H2Dialect {

    public MyH2Dialect() {
        registerFunction("group_concat", new StandardSQLFunction("group_concat", StandardBasicTypes.STRING));
    }

}
```

persistence.xml 파일의 Dialect를 MyH2Dialect로 수정한다.  
(추가가 아닌 수정이다.)  

```xml
<property name="hibernate.dialect" value="me.gicheol.dialect.MyH2Dialect"/>
```

여기까지가 등록 과정이다.  

FUNCTION 함수를 사용하고, 그 안에 매개변수로 추가한 함수명을 작성해서 사용한다.  

```java
String query = "SELECT FUNCTION('GROUP_CONCAT', m.username) FROM Member m";

List<String> resultList = em.createQuery(query, String.class)
                    .getResultList();

for (String result : resultList) {
    System.out.println("result = " + result);
}

//
Hibernate: 
    /* SELECT
        FUNCTION('GROUP_CONCAT',
        m.username) 
    FROM
        Member m */ select
            group_concat(member0_.username) as col_0_0_ 
        from
            Member member0_
result = MemberA,MemberB
```

---