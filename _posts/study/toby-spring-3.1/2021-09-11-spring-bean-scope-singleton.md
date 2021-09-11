---
title : "Spring Bean Scope - Singleton Scope"
categories : 
    - java
tag :
    - Mockito
toc : true
---

[토비의 스프링 3.1](http://www.yes24.com/Product/Goods/7516911)를 공부한 내용입니다.    

---

#### **Bean**  
보통 Bean이라 하면 **Java Bean**, **Spring Bean**을 떠올릴 수 있다.  

Java Bean은 Application을 개발함에 있어 생산성을 늘리기 위한 일종의 규약이다.  
[자세한 것은 JAVA Bean 위키백과 참고](https://ko.wikipedia.org/wiki/%EC%9E%90%EB%B0%94%EB%B9%88%EC%A6%88)  

Spring Bean은 Spring IoC 컨테이너에 의해 관리되는 대상이다.  

---

#### **Spring Bean Scope**  
Spring Bean Scope의 종류는 아래와 같다.  
- **singleton** 
- prototype 
- request 
- session
- application
- websocket

이 중 추가적으로 다른 설정을 하지 않는다면 Singleton을 기본전략으로 사용한다.    

즉 Application 처음 실행 시 생성되는 Spring Bean은 동일성을 보장한다.  

---

##### **동일성 , 동등성**  
만약 String 타입의 객체를 비교한다고 가정하자.  
자바를 조금이라도 공부를 했다면 아래 두개의 출력이 어떻게 나올지 쉽게 알 수 있을 것이다.  

```java
public static void main(String[] args) {
    String str1 = "Hello!";
    String str2 = "Hello!";

    System.out.println(str1 == str2);
    System.out.println(str1.equals(str2));
}
```

String 타입의 객체는 equals 메서드를 사용하면 true를 반환한다.  
그러나 == 비교 시 두 개의 값이 동일하더라도 false를 반환한다.  
 

여기서 알 수 있는 사실이 == 비교는 값 뿐만 아니라,  
두 개의 객체가 완전히 **동일**한지를 비교하며,    

equals 메서드는 두 개의 객체가 동일한 값을 가진지 비교하는 것인데, 이것을 **동등**성 비교라 한다.  

<br>

이전에 JPA를 공부했을 때 Entity Manager에 의해 관리되는,  
즉 Persist 상태의 객체는 == 비교시 항상 true를 반환했었다.  

이런 경우 객체의 동일성을 보장한다고 한다.  

---

#### **Spring Bean의 동일성**
Spring Bean은 기본전략으로 Singleton Scope를 사용하기 때문에 동일성을 보장한다고 했다.  

```java
public static void main(String[] args) {
    Something newSomething1 = new Something();
    Something newSomething2 = new Something();

    System.out.println(newSomething1 == newSomething2); // false


    ApplicationContext context = new AnnotationConfigApplicationContext(SomethingConfiguration.class);

    Something springSomething1 = context.getBean("something", Something.class);
    Something springSomething2 = context.getBean("something", Something.class);

    System.out.println(springSomething1 == springSomething2); // true
}
```

이러한 방식을 채택한 이유가 무엇인지 알아보자.  

---

#### **스프링 빈이 기본 전략으로 Singleton을 사용하는 이유**     
Singleton이 기본 전략으로 사용되는 이유는  
스프링을 통해 개발하는 서비스는 보통 자바 엔터프라이즈 기술을 사용하는 서버 환경이기 때문이다.  

이러한 환경은 브라우저나 다른 서비스로부터 초당 수십 혹은 수백번의 요청을 받아  
처리해야하는 높은 성능이 요구되었다.  

요청을 처리하는 구조가 계층형으로 구성 되어있고, (ex. MVC 패턴)  
비즈니스 로직이 복잡한 경우도 많다.  

MVC 패턴을 사용하는 웹서비스의 경우 아무리 간단하더라도,  
Controller -> Service -> DAO 구조로  
최소 3개의 객체가 생성되어야 하는데,  

초당 500번의 요청이 들어온다면 1초에 1500개의 새로운 객체가 생성된다.  

클라이언트의 매 요청마다 새로운 객체를 만든다는 것은 서버가 감당하기 매우 힘들 것 이다.  

이러한 이유로 인해 스프링은 빈의 기본 전략을 Singleton으로 채택했다.  

---

#### **Singleton Pattern의 한계**  
싱글톤을 구현하는 방법은 보통 아래와 같다.  

- 생성자를 private으로 만들어 외부에서 객체를 생성하지 못하도록 막는다.  
- 생성된 객체를 저장할 수 있는 자신과 같은 타입의 static 필드를 정의한다.  
- static factory method인 getInstance 메서드를 만들고, 최초 호출 시점에 단 한번 객체를 생성 후 static 필드에 저장한다.  
- 객체가 만들어진 후엔 이미 만들어져 있는 static 필드의 객체를 전달한다.  


```java
public class SingletonExam {
    
    private static SingletonExam INSTANCE;

    private SingletonExam() {}

    public SingletonExam getInstance() {
        if (INSTANCE == null) {
            INSTANCE = new SingletonExam();
        } 

        return INSTANCE;
        
    }
}
```

이러한 방식은 사실 문제점이 있다.  

1. 생성자가 private이기 때문에 상속이 불가하다.  
2. 객체가 만들어지는 방식이 제한적이기 때문에 테스트가 어렵거나 불가능하다.  
3. 싱글톤 패턴을 적용하더라도 멀티스레드 환경 등 싱글톤이 항상 보장되는 것은 아니다.  
4. 싱글톤 객체는 전역상태로 사용되기 쉽기 때문에 아무 객체에서나 자유롭게 접근해 수정할 수 있다.  

---

#### **Singleton Registry**
이런 문제점이 있기 때문에 스프링은 ApplicationContext를 통해  
싱글톤 객체를 만들고, 관리까지 해주는 기능을 제공한다.  

이것을 바로 **Singleton Registry**라고 부른다.  

싱글톤 레지스트리를 사용하면 싱글톤 객체도 public 생성자를 가질 수 있다.  

---