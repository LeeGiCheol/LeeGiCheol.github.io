---
title : "JPA 영속성 컨텍스트"
categories : 
    - jpa
tag :
    - jpa
toc : true
---

[김영한 : 자바 ORM 표준 JPA 프로그래밍 - 기본편](https://www.inflearn.com/course/ORM-JPA-Basic#curriculum) 강의를 공부한 내용입니다.  

---

#### **영속성 컨텍스트**  
영속성 컨텍스트는 **엔티티를 영구 저장하는 환경** 이라는 뜻이다.  

```java
EntityManager.persist(entity);
```

persist 메서드를 사용하는 것은 DB에 값을 저장하는 것이 아니라,  
EntityManager를 통해 JPA의 영속성 컨텍스트에 값을 저장한다.  

---

#### **엔티티의 생명주기**  
- 비영속 (new / transient)
    - 영속성 컨텍스트와 전혀 관계없는 새로운 상태  
- 영속 (managed)  
    - 영속성 컨텍스트에 관리되는 상태  
- 준영속 (detached)  
    - 영속성 컨텍스트에 저장되었다가 분리된 상태  
- 삭제 (removed)  
    - 삭제된 상태  

---

#### **비영속 상태**  
```java
Member member = new Member();
member.setId("member");
member.setUsername("LEE");
```

위와 같이 그저 객체 생성만 한 상태는 비영속 상태이다. (JPA에 관리대상이 아닌 경우)  

---

#### **영속 상태**  
```java
Member member = new Member();
member.setId("member");
member.setUsername("LEE");

EntityManager em = emf.createEntityManager();
EntityTransaction tx = em.getTransaction();
tx.begin();

em.persist(member);

tx.commit();
```

persist 메서드를 호출하면 member는 DB가 아닌 영속성 컨텍스트에 저장이 된다.  

이러한 상태가 영속상태이며, 이 상태에서 DB에 저장되지 않는다는 것을 증명해보자.  

```java
System.out.println("===== BEFORE =====");
em.persist(member);
System.out.println("===== AFTER =====");

tx.commit();
```

persist 호출 전/후로 출력을 하도록 코드를 작성했다.  
로그가 어떻게 찍혔을지 확인해보자.  

```java
===== BEFORE =====
===== AFTER =====
Hibernate: 
    /* insert me.gicheol.Member
        */ insert 
        into
            Member
            (name, id) 
        values
            (?, ?)
```

사실 이 부분에서 굉장히 큰 혼란이 왔다.  
BEFORE와 AFTER 사이에 persist가 있으니,  
당연히 INSERT 쿼리도 그 중간에 있을 것이라 생각했지만,  
엉뚱한 부분에서 쿼리가 발생됬다.  

JPA는 persist 메서드를 통해 영속성 컨텍스트에 객체를 보관하고,  
실제로 DB에 쿼리를 날리는 동작은 commit을 하면서 발생하게 된다.  

---

#### **준영속 상태**  
```java
em.persist(member);
em.detach(member);

tx.commit();
```

동일한 코드이지만, detach 메서드를 사용했다.  
detach는 인자에 기입된 객체를 영속성 컨텍스트에서 분리하는 역할을 한다.  

그렇기 때문에 위와 같은 코드는 commit을 하더라도 INSERT 쿼리가 동작하지 않는다.  

---

#### **삭제 상태**  
```java
em.remove(member);
```

remove 메서드는 실제 DB에서 값을 지우는 메서드이다.  

---

#### **영속성 컨텍스트의 이점**  
Application과 DB의 사이에 영속성 컨텍스트를 배치해둠으로써 얻는 이점은 다음과 같다.   

---

##### **1차 캐시**  
```java
Member member = new Member();
member.setId("member");
member.setUsername("LEE");

em.persist(member);

Member findMember = em.find(Member.class, "member");
```

만약 위와같은 상황에서 조회를 한다고 가정해보자.  

앞서 persist 메서드를 호출되면 영속성 컨텍스트에서 객체가 관리된다고 했었다.  
그렇다면 findMember는 데이터를 조회할 때 DB가 아닌 영속성 컨텍스트에서 충분히 조회할 수 있을 것이고, 이 편이 훨씬 이득일 것이다.  

만약 영속성 컨텍스트에 객체가 없다면 DB를 통해 객체를 조회한다.  
조회가 되었다면 1차 캐시에 객체를 저장한다.  
그리고 1차 캐시에서 객체를 반환한다.  

그러나 영속성 컨텍스트는 트랜잭션 단위의 영역이기 때문에 굉장히 작은 영역이다.    
그렇기 때문에 큰 이득을 보기는 힘들다.  

```java
 Member member = new Member();
member.setId(100L);
member.setName("LEE");

em.persist(member);

Member findMember = em.find(Member.class, 100L);
System.out.println("findMember.getId() = " + findMember.getId());
System.out.println("findMember.getName() = " + findMember.getName());


// 결과

findMember.getId() = 100
findMember.getName() = LEE
Hibernate: 
    /* insert me.gicheol.Member
        */ insert 
        into
            Member
            (name, id) 
        values
            (?, ?)
```

위의 코드를 실행해보면 SELECT문이 발생되지 않는다는 점을 알 수 있다.  

그렇다면 DB에서 조회하도록 해보자.  

```java
Member findMember1 = em.find(Member.class, 100L);
System.out.println("findMember1.getId() = " + findMember1.getId());
System.out.println("findMember1.getName() = " + findMember1.getName());

Member findMember2 = em.find(Member.class, 100L);
System.out.println("findMember2.getId() = " + findMember2.getId());
System.out.println("findMember2.getName() = " + findMember2.getName());


// 결과
Hibernate: 
    select
        member0_.id as id1_0_0_,
        member0_.name as name2_0_0_ 
    from
        Member member0_ 
    where
        member0_.id=?
findMember1.getId() = 100
findMember1.getName() = LEE
findMember2.getId() = 100
findMember2.getName() = LEE
```

이미 DB에 값이 저장됬다고 가정하고, find 메서드를 두번 사용하면  
SELECT문이 한번만 발생된 것을 알 수 있다.  
이미 DB에서 값을 조회하고, 1차 캐시에 저장을 해두었기 때문에  
한번의 SELECT문으로 조회할 수 있는 것이다.  

---

##### **영속 엔티티의 동일성 보장**
```java
Member findMember1 = em.find(Member.class, 100L);
Member findMember2 = em.find(Member.class, 100L);

System.out.println(findMember1 == findMember2);
```

위의 결과는 마치 객체를 비교하는 것처럼 true가 나온다.  
이는 1차 캐시에 의한 동일성이다.  

---

##### **트랜잭션을 지원하는 쓰기 지연**  
```java
em.persist(memberA);
em.persist(memberB);

tx.commit();
```

영속성 컨텍스트에는 1차 캐시 말고도 **"쓰기 지연 SQL 저장소"** 라는 공간이 있다.  
위와 같이 persist 메서드가 호출되면 1차 캐시에 객체를,  
쓰기 지연 SQL 저장소에 쿼리를 저장한다.  
memberA와 memberB 모두 동일하다.  

![error](/assets/images/kyh-jpa-basic/6-write_lazy_SQL_storage.png)  

![error](/assets/images/kyh-jpa-basic/7-write_lazy_SQL_storage.png)  

이와 같이 쓰기 지연 SQL 저장소에 쿼리가 저장되다가,  
commit을 하면 저장된 쿼리가 한번에 발생하게 된다.  

이러한 저장소가 장점인 이유는,  
위와 같은 memberA와 memberB를 Batch Insert를 할 수 있다는 점이다. (혹은 Bulk)  

Batch Insert를 하기 위해서는 persistence.xml 파일에 옵션을 추가해줘야한다.  

```xml
<property name="hibernate.jdbc.batch_size" value="여기에 사이즈를 int로 기입"/>
```

---

##### **변경 감지 (Dirty Checking)**
```java
Member member = em.find(Member.class, 150L);
member.setName("ABC");
System.out.println("========= SET NAME =========");

tx.commit();


// 결과
Hibernate: 
    select
        member0_.id as id1_0_0_,
        member0_.name as name2_0_0_ 
    from
        Member member0_ 
    where
        member0_.id=?
========= SET NAME =========
Hibernate: 
    /* update
        me.gicheol.Member */ update
            Member 
        set
            name=? 
        where
            id=?
```

JPA는 update 메서드가 따로 없다.  
persist가 저장을 해주니 member.setName을 해준 후 persist를 해줘야 할 것 같지만,  
절대 그렇지 않다.  

위 결과를 보면 SET NAME 주석이 발생한 후에 update문이 발생한 것이 보인다.  
이는 commit 이후에 객체가 변경됨을 감지하고, update문을 발생한 것이다.  

그렇기 때문에 만약 같은 내용으로 다시 한번 실행을 한다면, select문은 발생하겠지만, update문은 발생하지 않는다.  

어떻게 가능한 일인지 알아보자.  

![error](/assets/images/kyh-jpa-basic/8-dirty-checking.png)  

1차 캐시엔 Id와 Entity말고도 Snapshot을 저장한다.  
값을 최초로 읽어온 시점의 객체를 스냅샷으로 저장하는 것이다.  

이후 commit을 하면 내부적으로 flush 메서드가 호출되어 엔티티와 스냅샷을 비교한다.  
비교에 차이가 있다면 UPDATE 쿼리를 쓰기 지연 SQL 저장소에 저장한 후,  
다시 flush 메서드가 호출되어 DB에 값을 UPDATE 한다음에서야 commit을 완료 하게된다.  

그렇기 때문에 JPA는 객체의 값이 변경된다면 트랜잭션이 커밋되는 시점에 자동으로 update되는 것이다.  

---

#### **flush**  
영속성 컨텍스트의 변경사항을 데이터베이스에 반영한다.  

DB Transaction이 commit 되면 자동으로 발생한다.  
변경을 감지하며, 수정된 엔티티를 쓰기 지연 SQL 저장소에 등록하고, 데이터베이스로 전송한다.  

flush 하는방법은 다음과 같다.  

- em.flush();
- 트랜잭션 커밋
- JPQL 쿼리 실행

flush는 변경내용을 DB로 동기화 하는 것뿐이지, 영속성 컨텍스트를 비우지 않는다.  

---
