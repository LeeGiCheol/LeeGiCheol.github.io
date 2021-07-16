---
title : "JPA Cascade, 고아 객체"
categories : 
    - jpa
tag :
    - jpa
toc : true
---

[김영한 : 자바 ORM 표준 JPA 프로그래밍 - 기본편](https://www.inflearn.com/course/ORM-JPA-Basic#curriculum) 강의를 공부한 내용입니다.  

---

#### **Cascade (영속성 전이)**  
특정 엔티티를 영속 상태로 만들 때 연관된 엔티티도 함께 영속 상태로 만들고 싶을 때 사용된다.  

즉 부모 엔티티를 저장할 때 자식 엔티티도 같이 저장해야 할 경우 사용된다.  

---

##### **Cascade 사용하기**  
```java
@Entity
public class Parent {

    @Id
    @GeneratedValue
    private Long id;

    private String name;

    @OneToMany(mappedBy = "parent")
    private List<Child> childList = new ArrayList<>();


    public void addChild(Child child) {
        childList.add(child);
        child.setParent(this);
    }

}

@Entity
public class Child {

    @Id
    @GeneratedValue
    private Long id;

    private String name;

    @ManyToOne
    @JoinColumn(name = "parent_id")
    private Parent parent;

}
```

위와 같은 연관관계를 가진 Parent와 Child Entity가 있다.  
아래와 같은 경우는 생각보다 많이 발생할 것이다.  

```java
public class Main {
    public static void main(String[] args) {
        Child child1 = new Child();
        Child child2 = new Child();

        Parent parent = new Parent();
        parent.addChild(child1);
        parent.addChild(child2);

        em.persist(parent);
        em.persist(child1);
        em.persist(child2);
    }
}
```

현재는 어쩔 수 없이 세 번의 persist가 필요하다.  
Parent를 persist할 때 한 번에 child1, child2 까지 persist 된다면 좋겠다.  

이런 경우 Cascade를 사용할 수 있다.  

```java

@Entity
public class Parent {

    @Id
    @GeneratedValue
    private Long id;

    private String name;

    @OneToMany(mappedBy = "parent", cascade = CascadeType.ALL)
    private List<Child> childList = new ArrayList<>();

}
```

cascade = CascadeType.ALL 옵션을 사용하면 Cascade가 사용된다.  

```java
public class Main {
    public static void main(String[] args) {
        Child child1 = new Child();
        Child child2 = new Child();

        Parent parent = new Parent();
        parent.addChild(child1);
        parent.addChild(child2);

        em.persist(parent);
    }
}


//
Hibernate: 
    /* insert me.gicheol.Parent
        */ insert 
        into
            Parent
            (name, id) 
        values
            (?, ?)
Hibernate: 
    /* insert me.gicheol.Child
        */ insert 
        into
            Child
            (name, parent_id, id) 
        values
            (?, ?, ?)
Hibernate: 
    /* insert me.gicheol.Child
        */ insert 
        into
            Child
            (name, parent_id, id) 
        values
            (?, ?, ?)
```

Parent를 persist하면 연관된 Child를 모두 persist 해준다.  

---

##### **주의사항**  
주의사항으론 Cascade를 사용하면 엔티티를 영속화 할 때 연관된 엔티티도 함께 영속화 하는 것을 제공할 뿐,  
연관관계를 매핑하는 것과는 아무 관계가 없다.  

또한 Child와 Parent만이 관계가 있을 경우에만 사용해야 한다.  
다른 엔티티와 관계가 있다면, 사용하지 않아야 한다.   

---

##### **Cascade 종류**  
- ALL : 모두 적용
- PERSIST : 영속
- REMOVE : 삭제
- MERGE : 병합
- REFRESH : REFRESH
- DETACH : DETACH

---

#### **고아 객체**  
고아 객체 제거 : 부모 엔티티와 연관관계가 끊어진 자식 엔티티를 자동으로 삭제한다.  
orphanRemoval = true 옵션으로 사용할 수 있다.  

---

##### **고아 객체 사용하기**  
```java
@Entity
public class Parent {

    @Id
    @GeneratedValue
    private Long id;

    private String name;

    @OneToMany(mappedBy = "parent", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<Child> childList = new ArrayList<>();

}

public class Main {
    public static void main(String[] args) {
        Child child1 = new Child();
        Child child2 = new Child();

        Parent parent = new Parent();
        parent.addChild(child1);
        parent.addChild(child2);

        em.persist(parent);

        em.flush();
        em.clear();

        Parent findParent = em.find(Parent.class, parent.getId());
        findParent.getChildList().remove(0);
    }
}
```

orphanRemoval = true 옵션이 선언된 객체는 List에서 제거 됬을 때 DB에서도 삭제된다.  

Parent가 삭제된다면 연관된 Child는 모두 삭제된다.  

---

##### **고아 객체 주의사항**  
- 참조하는 곳이 하나일 때 사용해야한다.  
- Cascade와 마찬가지로 특정 엔티티가 개인 소유할 때 사용한다.  
- @OneToOne, @OneToMany만 사용 가능하다.  
- 부모를 제거할 때 자식도 함께 제거되는 고아 객체는 CascadeType.REMOVE처럼 동작한다.  

---

##### **CascadeType.ALL, orphanRemoval = true**  
이 두 옵션을 활성화 하면, 부모 엔티티를 통해서 자식 엔티티의 생명주기를 관리할 수 있다.  

---