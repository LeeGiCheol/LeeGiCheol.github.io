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

##### **@Table**  
@Table은 엔티티와 매핑할 테이블을 지정한다.  

속성 : name
- 매핑할 테이블 이름을 지정한다.  
- 기본값은 엔티티 이름이다.  

---