---
title : "JPA JPQL 소개"
categories : 
    - jpa
tag :
    - jpa
toc : true
---

[김영한 : 자바 ORM 표준 JPA 프로그래밍 - 기본편](https://www.inflearn.com/course/ORM-JPA-Basic#curriculum) 강의를 공부한 내용입니다.  

---

#### **JPQL (Java Persistence Query Language)**  
JPA가 정말 다양하고 유용한 기능을 제공하지만, 한계가 있다.  
바로 검색 쿼리인데, 검색 쿼리는 다른 쿼리에 비해 훨씬 복잡할 수 있다.  
이때 JPA에 정의된 객체지향쿼리 언어인 JPQL을 사용할 수 있다.     
SQL문을 추상화했기때문에 문법이 비슷하지만  
쿼리의 대상이 테이블이 아닌, 엔티티라는 부분이 다르다.  
또한 특정 데이터베이스 SQL 문법에 의존적이지 않다.  

---

##### **JPQL 문법**  
```java
public static void main(String[] args) {
    List<Member> memberList = em.createQuery(
                    "SELECT m FROM Member m WHERE m.name LIKE '%LEE%'", Member.class
                ).getResultList();

    memberList.forEach(System.out::println);
}
```

여기서 보통 SQL문은 FROM뒤의 Member 테이블을 대상으로 * 혹은 컬럼명을 사용해 조회하는데,   
JPQL은 Member 엔티티에 Alias를 주고 엔티티를 대상으로 쿼리한다.  
단점은 쿼리를 문자열을 통해 작성하기 때문에 오타가 발생할 수 있고,  
동적쿼리를 사용하기 불편하다.  

---

##### **Criteria**  
위에서 말한 JPQL의 단점을 보완하기 위해 Criteria가 있다.  
JPQL을 자바 코드로 사용할 수 있는 문법이다.  

```java
public static void main(String[] args) {
    CriteriaBuilder cb = em.getCriteriaBuilder();
    CriteriaQuery<Member> query = cb.createQuery(Member.class);

    Root<Member> m = query.from(Member.class);
    CriteriaQuery<Member> cq = query.select(m).where(cb.equal(m.get("username"), "LEE"));

    em.createQuery(cq);
}
```

Criteria를 사용하면 안전할 수 있고, 동적쿼리를 사용하기 쉽지만,  
아쉬운건 사용 및 유지보수가 복잡하다는 문제가 있다.   

---

##### **Query DSL**  
Criteria와 마찬가지로 자바코드로 JPQL을 작성할 수 있다.  
컴파일 시점에 문법 오류를 작성할 수 있어 오타의 걱정이 없고,  
동적쿼리를 사용하기 편하다.  

여기까진 Criteria의 장점과 동일하지만 QueryDSL은 사용이 단순하고 쉽다.  
다만 세팅이 번거롭다는 단점이 있다.  

---

##### **Native SQL**  
JPQ가 제공하는 SQL을 직접 사용하는 기능이다.  
JPQL로 해결할 수 없는 특정 데이터베이스에 의존적인 기능을 사용해야 할때 쓸 수 있다.  
ex) 오라클 Connect By, SQL 힌트..  

EntityManager.createNativeQuery() 를 사용해서 간단하게 쓸 수 있다.  

```java
public static void main(String[] args) {
    List<Member> resultList = em.createNativeQuery(
                "SELECT MEMBER_ID, USERNAME FROM MEMBER", Member.class)
            .getResultList();

    resultList.forEach(System.out::println);
}
```

이때 사용되는 MEMBER는 엔티티 대상이 아닌 테이블 대상이다.  

---

##### **JDBC를 직접 사용**  
JPA를 사용하면서 Spring JdbcTemplate, MyBatis 등을 함께 사용할 수 있다.  

주의할 점은 JPA는 persist 시점이 아닌 commit 시점에 flush가 되어 쿼리가 실행된다.  
JPA의 기능인 NativeQuery같은 경우 쿼리를 실행하는 시점에 (createNativeQuery())  
자동으로 flush가 발생한다.  

그러나 Spring JdbcTemplate, MyBatis는 JPA의 기능이 아니다.  
그렇기 때문에 자동으로 flush가 되지 않기 때문에 수동으로 flush를 해주어야 한다.  

---