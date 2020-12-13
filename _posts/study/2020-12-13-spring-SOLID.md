---
title : "객체지향 설계원칙 (SOLID)"
categories : 
    - study
tag :
    - SOLID
toc : true
---

김영한님의 강의인 스프링 핵심 원리 - 기본편을 볼 기회가 생겨 공부해보았다.<br><br>

- **Spring 핵심 기술** : Spring DI Container, AOP, Event ...

- **Spring Boot** : Tomcat 같은 웹서버를 내장해서 별도의 웹 서버를 설치하지 않아도 된다  

- **SOLID**
    - SRP (Single Responsibility Principle)
        - 한 클래스는 하나의 책임만 가져야 한다.
        - 중요한 기준은 변경이다. 변경이 있을 때 파급 효과가 적으면 단일 책임 원칙을 잘 따른 것
    - OCP (Open Closed Principle)
        - 소프트웨어 요소는 **확장에는 열려** 있으나 **변경에는 닫혀** 있어야 한다.
        - 그러나 구현 객체를 변경하려면 클라이언트 코드를 변경해야한다.
        - 해결하기 위해 **객체를 생성하고, 연관관계를 맺어주는 별도의 조립, 설정자**가 필요하다.
    - LSP (Liskov Substitution Principle)
        - 프로그램의 객체는 프로그램의 정확성을 깨뜨리지 않으면서 하위 타입의 인스턴스로 바꿀 수 있어야 한다.
        - 예를들어 자동차의 엑셀을 밟았을때 후진하게끔 구현할 수는 있으나 이는 인터페이스 규약을 위반하게된다. 이런 경우가 LSP를 위반한 것이다.
    - ISP (Interface Segregation Principle)
        - 특정 클라이언트를 위한 인터페이스 여러개가 범용 인터페이스 하나보다 낫다.
    - DIP (Dependency Inversion Principle)
        - 프로그래머는 **"추상화에 의존해야지, 구체화에 의존하면 안된다."**
        - 구현 클래스에 의존하지 말고, 인터페이스에 의존하라는 뜻