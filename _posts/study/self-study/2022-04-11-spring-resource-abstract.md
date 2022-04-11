---
title : "Spring Resource 추상화"
categories : 
    - spring
tag :
    - spring
toc : true
---

[인프런 백기선-스프링 프레임워크 핵심 기술을 공부한 내용입니다.](https://www.inflearn.com/course/spring-framework_core)

---

#### **Resource 추상화**  
스프링에서 지원하는 Resource 인터페이스가 있다.  
이 Resource는 java.net.URL을 추상화 한 인터페이스로 스프링 내부에서 다양하게 사용되고 있다.  

---

#### **Resource가 추상화 된 이유**
- 클래스패스 기준으로 리소스를 읽어오는 기능이 없다.  
- ServletContext를 기준으로 상대 경로를 읽어오는 기능이 없다.  
- 새로운 핸들러를 구현해 특별한 URL 접두사를 만들어 사용할 수는 있지만, 구현이 복잡하고 편의성 메서드가 부족하다.  

---

#### **구현체**
- UrlResource
- ClassPathResource
- ServletContextResource : 웹 애플리케이션 루트에서 상대 경로로 리소스를 찾는다.
- ...

Resource는 location 문자열과 **ApplicationContext**의 타입에 따라 결정된다.  
- ClassPathXmlApplicationContext -> ClassPathResource
- FileSystemXmlApplicationContext -> FileSystemResource
- WebApplicationContext -> ServletContextResource

```java
@Component
public class AppRunner {

    public void run() {
        ApplicationContext context = ClassPathXmlApplicationContext("application.xml");
    }

}
```

위의 경우는 ClassPathXmlApplicationContext 타입을 사용하기 때문에 location 문자열에 굳이 `classpath:` 라는 접두어를 사용할 필요가 없다.  

```java
@Component
public class AppRunner {

    public void run() {
        ApplicationContext context = AnnotationConfigServletWebServerApplicationContext("application.xml");
    }

}
```

만약 이런 경우라면 웹 애플리케이션 루트에서 상대 경로로 application.xml을 찾게 된다.  
여기서 ApplicationContext의 타입과 무관하게 ResourceLoader의 구현체를 설정하려면 접두어를 사용해야 한다.  

```java
@Component
public class AppRunner {

    public void run() {
        ApplicationContext classpathContext = AnnotationConfigServletWebServerApplicationContext("classpath:application.xml");

        ApplicationContext fileContext = AnnotationConfigServletWebServerApplicationContext("file:///application.xml");
    }

}
```

웹개발을 할때 보통 ApplicationContext는 WebApplicationContext을 사용하기 때문에 ServletContextResource을 사용하겠지만,  
코드만으로는 알기 어렵다.  

접두어를 쓰는것이 더 명시적이므로 ApplicationContext 타입에 의존하기 보단 접두어를 사용하도록 하자.  

---