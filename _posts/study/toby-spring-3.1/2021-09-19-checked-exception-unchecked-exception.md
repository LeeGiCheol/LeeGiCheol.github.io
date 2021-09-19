---
title : "Checked Exception, UnChecked Exception"
categories : 
    - java
tag :
    - java
toc : true
---
    
---

#### **Exception**  
에러는 컴파일 에러와 런타임 에러 크게 두 가지로 구분된다.  

컴파일 에러는 주로 문법적인 오류로 컴파일 시점에 알 수 있는 에러이다.  
IDE를 사용한다면, 보통 컴파일 에러가 발생하는 라인에 빨간 밑줄이 생긴다.  

```java
public static void main(String[] args) {
    int number = "int형 변수에 문자를 담으면 컴파일 에러가 발생합니다";
    String str = "세미콜론이 없으니 컴파일 에러가 발생했습니다."
}
```

런타임 에러는 컴파일 이후에 발생하는 오류이다.  
문법상으로는 이상이 없지만, 아래와 같이 실행할 때 이상이 발생한 경우이다.  

```java
public static void main(String[] args) {
    System.out.println(1 / 0); // ArithmeticException: / by zero 발생!
}
```

---

#### **Checked Exception, UnChecked Exception**  
이 둘을 구분하는 것은 바로 RuntimeException이다.  

다양한 Exception 클래스는 결국 Exception 클래스를 상속받는데,  
그 중 RuntimeException을 상속받은 클래스들이 있다.  

가령 바로 위에서 사용한 ArithmeticException은 RuntimeException을 상속받는다.  
반대로 IOException은 Exception을 상속받는다.  

여기서 어떤 차이가 발생하는지 알아보자.  

먼저 RuntimeException을 상속받은 예외클래스는 UnChecked Exception,  
그 외의 예외클래스는 Checked Exception이다.  

---

##### **Checked Exception**
![error](/assets/images/study/toby-spring-3.1/04-checked-exception-1.png)   

IOException은 Exception을 상속받은 Checked Exception이다.  
이러한 에러는 대비가 가능하고, 복구가 가능하기 때문에  
말 그대로 체크를 해야된다는 의미이다.  

위의 경우 checkedException 메서드에서 throw를 했지만  
메서드 선언부에 throws IOException 이라는 문구가 사용되지 않았기 때문에  
처리할 수 없는 예외처리라며 IDE가 미리 컴파일 에러를 만들어준다.  

![error](/assets/images/study/toby-spring-3.1/05-checked-exception-2.png)   

메서드 선언부에 throws IOException을 작성한다면  
이제는 checkedException 메서드를 사용하는 메서드에서 동일한 에러가 발생한다.  

두 경우 모두 던져진 에러를 처리하지 않았기 때문에 발생한 것인데,  

사용하는 메서드에서도 try-catch, 혹은 throws IOExcetpion을 사용해야 한다.  

![error](/assets/images/study/toby-spring-3.1/06-checked-exception-3.png)   

---

##### **UnChecked Exception**  
반면 Unchecked Exception은 RuntimeException을 상속 받은 예외 클래스로  
프로그램에 오류가 있다는 의미를 내포한다.  

대체적으로 예외처리를 할 수 있지만, 개발자의 실수로 발생한 경우로,  
예상할 수 있는 예외이다.  

![error](/assets/images/study/toby-spring-3.1/07-unchecked-exception.png)   

그렇기 때문에 unCheckedException 메서드는 예외를 던지지만,  
메서드 선언부에서 상위 메서드로 예외를 던지지 않았고,  

사용하는 메서드에서도 예외처리를 하지 않았음을 알 수 있다.  

---