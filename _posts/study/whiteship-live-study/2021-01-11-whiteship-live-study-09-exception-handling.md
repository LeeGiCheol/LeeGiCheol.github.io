---
title : "WhiteShip Live Study 9주차. 예외처리"
categories :
- whiteship-live-study
tag :
- whiteship-live-study
toc : true
---

## WhiteShip Live Study 9주차. 예외 처리

---

## 목표
자바의 예외 처리에 대해 학습하세요.

## 학습할 것 (필수)
- 자바에서 예외 처리 방법 (try, catch, throw, throws, finally)
- 자바가 제공하는 예외 계층 구조
- Exception과 Error의 차이는?
- RuntimeException과 RE가 아닌 것의 차이는?
- 커스텀한 예외 만드는 방법

---

### 예외란?  
#### RuntimeException과 RE가 아닌 것의 차이는?

![error](/assets/images/whiteship-live-study/2021-01-11/architecture.png)  

모든 클래스의 조상은 Object 클래스이다. Exception, Error 클래스 또한 마찬가지이다.  

프로그램이 어떠한 이유로 비정상적으로 종료하거나 오작동하는 경우를 에러, 혹은 오류라 한다.  
이때 발생시점에 따라 에러를 구분할 수 있다.  

```컴파일 에러```  
컴파일 할 때 소스코드에 오타나 잘못된 문법을 사용할 경우 발생한다.  
IDE를 사용한다면 실행하기 전부터 빨간줄가 생긴다. (혹은 X 표시)  
아래를 보면 ;(세미콜론) 이 없는 문장과, int 변수에 String 값을 넣으려고 하면서 발생한 컴파일 에러이다.  
  
![error](/assets/images/whiteship-live-study/2021-01-11/compileError.png)    

```런타임 에러```  
컴파일은 됬지만 실행 중에 에러가 발생한 경우이다.  
null값을 참조해 발생하는 NullPointerException, 0으로 나눗셈을 하려다 발생하는 ArithmeticException 등등 다양하다.  


**NullPointerException**  
![error](/assets/images/whiteship-live-study/2021-01-11/null.png)

![error](/assets/images/whiteship-live-study/2021-01-11/null2.png)


**ArithmeticException**  
![error](/assets/images/whiteship-live-study/2021-01-11/arith.png)  

![error](/assets/images/whiteship-live-study/2021-01-11/arith2.png)    

```논리적 에러```  
컴파일도 되고, 실행도 됬지만 의도한 동작이 아닌 다른 동작을 하는 경우이다.  
만약 게임을 하는데 HP가 0 혹은 음수가 되었는데 죽지않는 경우를 논리적 에러라고 할 수 있다.  

---

### 에러, 예외

#### Exception과 Error의 차이는?

런타임에러는 다시 에러와(Error) 예외로(Exception) 나뉘어진다.  
에러는 OutOfMemoryError, StackOverFlowError 와 같은 발생 시 복구할 수 없는 심각한 오류를 의미한다.  
예외는 발생하더라도 수습할 수 있는 비교적 덜 심각한 오류를 말한다.  

---

### 자바에서 예외 처리 방법 (try, catch, throw, throws, finally)

### 예외처리란?
**예외처리란** 프로그램 실행 시 발생할 수 있는 예기치 못한 예외의 발생에 대비한 코드를 작성하는 것이다.  
OutOfMemoryError와 같이 프로그래머, 혹은 사용자의 실수가 아닌 '**에러**'는 어쩔 수 없지만,  
예외는 프로그래머가 미리미리 처리를 해두어야한다.  

이때 처리하지 못한 예외는 프로그램을 비정상적으로 종료시키며, JVM의 예외처리기가 (UncaughtExceptionHandler) 받아서 예외의 원인을 화면에 출력한다.  

### try-catch
예외를 처리하기 위해 try-catch문을 사용한다.  

```java
try {
    System.out.println("일부러 예외를 발생시킨다.");
    System.out.println(5 / 0);
    System.out.println("이 문장은 어떻게 될까?");
} catch (Exception e) {
    System.out.println("Arithmetic Exception 발생!! 0으로 나눌 수 없습니다!!");
}
System.out.println("결과는?");

// 결과
일부러 예외를 발생시킨다
Arithmetic Exception 발생!! 0으로 나눌 수 없습니다!!
결과는?
```

0으로 나눗셈을 하려고하면 ArithmeticException이 발생한다.  
그렇기에 "이 문장은 어떻게 될까?" 는 출력되지 않고,  
catch문으로 들어가 "Arithmetic Exception 발생!! 0으로 나눌 수 없습니다!!" 를 출력한다.  
그 후 다시 catch문을 마치고, "결과는?" 이란 문장을 출력한다.  

반대로 try 문에 예외가 아닌 정상적인 코드를 삽입하면, catch문은 실행되지 않는다.  

### 어떻게 동작하는가?
try 블럭에 예외가 발생하는 문장이 포함되어있다면, 이 예외를 처리할 catch 블럭이 있는지를 먼저 찾는다.  
catch 블럭의 예외와, 실제 발생한 예외를 instaceof 연산자를 사용해 검사를 한다.  
검사 결과 true라면 catch문을 수행하고, false라면 예외는 처리되지 않는다.  

이때 catch 블럭의 괄호안에는 원하는 예외를 작성할 수 있으며, 하나의 try 블럭에 여러개의 catch 블럭을 작성할 수 있다.  
또한 Exception은 다양한 Exception의 부모 클래스로 Exception 클래스를 선언해두면 어떠한 예외도 이 블럭 안에서 처리가 된다.  

```java
try {
    System.out.println("일부러 예외를 발생시킨다.");
    System.out.println(5 / 0);
    System.out.println("이 문장은 어떻게 될까?");
} catch (NullPointerException e1) {
    System.out.println("Null Pointer Exception 발생!! null을 참조할 수 없습니다!!");
} catch (ArithmeticException e2) {
    System.out.println("Arithmetic Exception 발생!! 0으로 나눌 수 없습니다!!");    
} catch (Exception e3) {
    System.out.println("Exception!!!");
}
System.out.println("결과는?");

// 결과
일부러 예외를 발생시킨다
Arithmetic Exception 발생!! 0으로 나눌 수 없습니다!!
결과는?
```

지금은 NullPointerException이 아니라 ArithmeticException이기 때문에 두번째 catch문을 수행할 것이다.  
세번째 catch문인 Exception도 처리 할 수 있지만, 이미 처리가 완료된 상황이기 때문에 세번째 catch 블럭은 검사하지 않는다.  

---

### printStackTrace(), getMessage()
예외처리를 할 때 발생한 예외의 대한 정보가 담긴 메서드이다.  
catch 블럭의 참조변수를 통해 인스턴스에 접근할 수 있다.  

- printStackTrace()
    - 예외 발생 당시에 호출스택에 있었던 메서드의 정보와 예외 메시지를 화면에 출력한다.
- getMessage()
    - 발생한 예외클래스의 인스턴스에 저장된 메시지를 얻을 수 있다.  

---

### 멀티 catch

JDK 1.7 부터 추가된 기능이다.  
'|' 기호를 사용해 여러 catch 블럭을 하나로 합칠 수 있는데, 이것을 멀티 catch라고 한다.  
이때 합칠 수 있는 예외 클래스의 개수는 제한이 없다.  

그러나 연결되어있는 예외 클래스가 부모/자식 관계에 있다면, 컴파일 에러가 발생한다.  


```java
try{
    System.out.println(5 / 0);
} catch(ArrayIndexOutOfBoundsException | ArithmeticException e1) {   // OK
    e1.printStackTrace();
} catch(Exception | ArithmeticException e2) {                        // Compile Error
        e2.printStackTrace();
}
```

---

### finally
finally 블럭은 예외와 상관없이 실행되는 블럭이다.  
만약 try 블럭에서 예외가 발생하면 바로 catch 블럭을 찾기 때문에  
꼭 필요한 코드가 실행되지 않을 수 있다. 이러한 경우에 finally를 사용 할 수 있다.  

기본적으로 try-catch-finally 의 구조를 가지며,  
예외 발생 시 try-catch-finally, 예외 발생이 하지 않으면 try-finally 순서로 동작한다.  

```java
String NPE = null;

try {
    System.out.println("예외가 발생할 겁니다.");
    System.out.println(5 / 0);
    System.out.println("이 문장은 꼭 출력됬으면 좋겠어요.");
} catch (Exception e) {
    System.out.println("이 문장은 꼭 출력됬으면 좋겠어요.");
    e.printStackTrace();
}

// 결과
예외가 발생할 겁니다.
이 문장은 꼭 출력됬으면 좋겠어요.
```

위의 코드는 5 / 0 에서 예외가 발생할 것이다.  
그러나 예외가 발생하더라도 꼭 출력했음 하는게 있다면 위와 같이 할 수 있다.  
그렇지만 이런 코드는 굉장히 비효율적으로 보인다.    

```java
try {
    System.out.println("예외가 발생할 겁니다.");
    System.out.println(5 / 0);
} catch (Exception e) {
    e.printStackTrace();
} finally {
    System.out.println("이 문장은 꼭 출력됬으면 좋겠어요.!");
}
// 결과
예외가 발생할 겁니다.
이 문장은 꼭 출력됬으면 좋겠어요.!
```

결과는 똑같지만 finally 블럭을 활용한 코드가 훨씬 보기 좋고 간결한 것을 알 수 있다.   

---

### throw
throw 키워드를 사용하면 프로그래머가 고의적으로 예외를 발생시킬 수 있다.

```java
public class Person {

    private String userName;
    
    public String getUserName() {
        return userName;
    }
    
    public void setUserName(String userName) {
        this.userName = userName;
    }
    
    public void throwExam() {
        Person person = jpaRepository.findById("userName");
    
        if (person == null) {
            throw new NullPointerException();
        } else {
            System.out.println(person.getName());
        } 
    }
}
```

Person 클래스를 데이터베이스에서 조회한다고 가정하자.  
Person의 name은 값이 있을 수도, 없을 수도 있다.  
만약 값이 없는 경우를 대비해 if문과 함께 throw 키워드를 사용해 임의적으로 예외를 발생시킬 수 있다.  
또는 커스텀한 예외를 만들어서 사용할 수 있다.

---

### throws

![error](/assets/images/whiteship-live-study/2021-01-11/throws.png)

doSomething 메서드에서 호출한 parseInt 메서드는 왜 빨간 밑줄이 그어졌을까?  
아래 parseInt 메서드를 보자.  
String을 int로 변환을 해주는 메서드이다.  
단순히 '12ASD3' 이라는 인자가 숫자가 아니여서 컴파일 에러가 난 것이 아니다. (이런건 IDE가 잡아주지도 못한다.)  
parseInt 메서드 뒤의 throws Exception 때문이다.  

throws는 바로 예외를 던지는 (위임하는) 역할을 한다.

![error](/assets/images/whiteship-live-study/2021-01-11/throws2.png)  

인텔리제이가 추천하는 두가지 해결 방안이다.  
**메서드에 예외를 추가하거나,**  

![error](/assets/images/whiteship-live-study/2021-01-11/throws3.png)  

**try/catch를 사용하거나.**  

![error](/assets/images/whiteship-live-study/2021-01-11/throws4.png)  

---

### 왜 사용할까?

만약 위의 parseInt 메서드가 **팀 프로젝트**에서 굉장히 많이 사용하는 util 클래스라고 가정하자.    
메서드를 작성하면서 String을 int로 변환하는 일은 예외가 발생할 확률이 **굉장히** 크다는 것을 인지했다.  

물론 메서드안에서 try/catch문을 사용하는 방법도 있지만,  
예외처리를 함으로써 메서드가 복잡해져 읽기 어렵고, 추가적인 예외가 발생할 수 있다.  

이런 상황에서 throws를 사용하면 다른 팀원들이 메서드를 사용하는 상황에 맞게 예외처리를 할 수 있어진다.  
또한 메서드 이름 옆에 발생할 수 있는 예외가 명시적으로 작성이 되므로 코드작성에 주의를 할 수 도 있다.  

하지만 throws를 계속해서 사용하다보면 main 메서드까지 가게 되는데,  
main 메서드 또한 예외를 throws 하게 된다면 JVM이 대신 처리하게 된다.  
결국 예외를 처리하지 않은 것과 다름없어지는 것이다.  

때문에 그저 예외처리가 귀찮아서 throws를 사용하는 것은 금물이다.    

---

### 커스텀한 예외 만드는 방법

때때로 직접 예외를 커스텀할 경우가 생긴다.  
예를들어 어떤 홈페이지에서 회원가입을 하려고 하는데 나이를 입력하던 도중 실수로 음수 나이를 적었다.  
예외처리를 하고 싶지만, 마땅한 예외가 보이지 않는다.  
이런 경우 커스텀한 예외를 만들어 던질 수 있다.  
(물론 if문이나 정규표현식 등 해결할 수 있는 방법은 다양하다.)  

예외를 커스텀하기 위해선 Exception을 상속받아야한다.  
그 후 생성자를 만들어 출력할 메시지를 받아 출력할 수 있다.  
이때 기본 생성자를 만들어 사용하면 메시지없이 어떤 예외인지만 볼 수 있다.  


```java
public class AgeException extends Exception {
    AgeException() {}
    
    AgeException(String message) {
        super(message);
    }
}
```

```java
public class SignUpService {
    public void setAge(int age) throws AgeException {
        if (age < 0) {
            throw new AgeException("나이는 음수일 수 없습니다!!!");
        }
    }
}
```

```java
public class SignUp {
    public static void main(String[] args) throws AgeException {
        SignUpService signUp = new SignUpService();
        
        System.out.println("회원가입을 시작합니다~");
        ...
        signUp.setAge(-26);
        ...
    }
}
```

![error](/assets/images/whiteship-live-study/2021-01-11/custom-exception.png)  

기본 생성자로 예외 처리를 한 경우  

![error](/assets/images/whiteship-live-study/2021-01-11/custom-exception2.png)  

메시지를 받는 생성자로 예외 처리를 한 경우  

Exception 이름이 AgeException인 것과 내가 작성한 예외 메시지를 볼 수 있다. (SignUpService에서 작성함)  

커스텀 예외 클래스의 생성자에서 ```super(message);``` 를 사용한다.  
따라 올라가보면 Throwable 클래스의 생성자를 볼 수 있다.  
Throwable 클래스에서 예외 메시지 처리를 하고 있는 것이다.  

--- 

### 예외처리 비용

![error](/assets/images/whiteship-live-study/2021-01-11/custom-exception3.png)    

위 코드는 Throwable 클래스의 생성자이다. 

보통 예외처리는 처리 비용이 비싸다고 한다.  
try-catch를 동작 하면서 발생하는 검사들도 하나의 원인이겠지만,  
Throwable 생성자의 fillInStackTrace() 메서드가 주 원인이다.  
이 메서드는 예외가 발생한 메서드의 **Stack Trace**를 모두 출력해주기 때문이다.

```text
StackTrace란 Application이 실행된 시점부터 현재 실행 위치까지의 메서드 호출 목록이다.
```


커스텀 예외에서 이 메서드를 오버라이딩해 스택 트레이스를 최소화 해줄 수 있다.  

```java
@Override
public synchronized Throwable fillInStackTrace() {
    return this;
}
```

![error](/assets/images/whiteship-live-study/2021-01-11/custom-exception4.png)  

하지만 예외 처리 비용을 아끼겠다고 굳이 사용할 일이 생길까 싶긴하다...  

---





