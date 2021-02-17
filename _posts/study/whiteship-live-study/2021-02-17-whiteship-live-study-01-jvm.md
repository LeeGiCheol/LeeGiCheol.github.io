---
title : "WhiteShip Live Study 1주차. JVM은 무엇이며 자바 코드는 어떻게 실행하는 것인가."
categories :
- whiteship-live-study
tag :
- whiteship-live-study
toc : true
---

## WhiteShip Live Study 1주차. JVM은 무엇이며 자바 코드는 어떻게 실행하는 것인가.

---

## 목표
자바 소스 파일(.java)을 JVM으로 실행하는 과정 이해하기.

## 학습할 것
- JVM이란 무엇인가
- 컴파일 하는 방법
- 실행하는 방법
- 바이트코드란 무엇인가
- JIT 컴파일러란 무엇이며 어떻게 동작하는지
- JVM 구성 요소
- JDK와 JRE의 차이

---

### JVM이란 무엇인가

JVM이란 Java Virtual Machine의 약자이다.  
Virtual Machine은 가상 머신 (가상 컴퓨터)이다.  
즉 JVM은 '자바를 실행하기 위한 가상 머신' 이다.   

자바로 구현된 애플리케이션은 모두 JVM에서 실행되기 때문에  
반드시 JVM이 필요하다.  

![error](/assets/images/whiteship-live-study/2021-02-17/jvm.png)  

위의 그림에서 알 수 있듯이  
자바 애플리케이션은 JVM을 거치고 OS까지 거쳐야한다.  
이 때문에 속도가 느리다는 단점이 있다.  
그러나 OS에 독립적이기 때문에 다른 OS에서도    
해당 OS에서 실행 가능한 JVM이 존재한다면 프로그램의 변경없이 실행할 수 있다.  


---

### 애플리케이션 실행까지..

```java
public class Main {
    public static void main(String[] args) {
        System.out.println("Hello World!");
    }
}
```

위는 콘솔에 Hello World! 를 출력하는 자바 코드이다.  
이 프로그램을 실행하기 위해서는 먼저 '컴파일'이라는 작업을 거쳐야 한다.  
기본적으로 위와 같은 소스 코드는 xxx.java 라는 파일이다.  
컴파일을 거치게 된다면 xxx.class 라는 파일로 변경이 되고,  
개발자가 읽고 쓸 수 있는 소스코드에서 '바이트 코드'로 변경이 된다.  

그리고 인터프리터가 바이트 코드를 해석 해 애플리케이션이 실행된다.  

---

### 컴파일 하는 방법

바로 위의 Main 클래스를 컴파일 하고 실행해보겠다.  
터미널을 통해 컴파일 하는 방법이다.  
해당 경로로 이동을 한다. (cd 경로/경로/경로)  

![error](/assets/images/whiteship-live-study/2021-02-17/compile-1.png)   
 
이후 자바 컴파일러를 통해 Main.java 파일을 컴파일한다.  
위 명령어 javac의 c가 바로 컴파일러의 앞글자이다.  
컴파일에 성공하면 위와 같이 아무일도 발생하지 않은 듯 하며,    
Main.class라는 파일이 생성된다.  

![error](/assets/images/whiteship-live-study/2021-02-17/compile-2.png)  

만약 오타와 같은 실수를 했다면, 컴파일 과정에서 에러가 발생할 것이다.  
아래는 class를 clas 라고 쓴 오타가 있다.  

```java
public clas Main {
    public static void main(String[] args) {
        System.out.println("Hello World!")
    }
}
```

![error](/assets/images/whiteship-live-study/2021-02-17/compile-3.png)  

이렇게 에러가 발생하고, Main.class 파일은 생성되지 않는다.  
컴파일 할 때 모든 경우에 이렇게 친절하게 에러를 알려주진 않는다. (컴파일 에러)  

실행을 하고 어떤 행위가 발생했을 때만 발생하는 에러들도 있으니 (런타임 에러)  

컴파일을 하고 아무런 에러가 발생하지 않는다고 안심할 순 없다.  

---

### 실행하는 방법

컴파일에 성공하면 같은 경로에 Main.class 파일이 생성되있을 것이다.  

실행을 하기 위해서는 인터프리터를 사용해야 한다.  

![error](/assets/images/whiteship-live-study/2021-02-17/run.png)  

굉장히 별거 없다.  
인터프리터는 java 라는 파일이고, (윈도우는 java.exe)  
이를 통해 Main.class 파일을 실행하는 것 뿐이다.  

여기서 주의 할 점은 컴파일 할 때는 확장자까지 입력을 해야하지만,  
실행할때는 어떠한 확장자도 입력하면 안된다.  

![error](/assets/images/whiteship-live-study/2021-02-17/run-error-1.png)  

그런데 Main.java에 당연히 에러가 발생할 것이라 생각했는데 아니었다.  
환경변수가 JDK 11로 되어있어 그럴까 싶어, 테스트해보았다.  

![error](/assets/images/whiteship-live-study/2021-02-17/run-error-2.png)    

JDK 1.8 이고, 디렉토리에는 Main.java 만 있다.  
이때 java Main.java 명령어를 입력하니 처음에 원했던 에러가 발생했다.  

다음은 JDK 11이다.  

![error](/assets/images/whiteship-live-study/2021-02-17/run-error-3.png)

동일한 상황이지만 JDK 버전이 11이다.  
컴파일을 하지 않았음에도 불구하고 실행이 됬다.  
정확히 어떤 버전부터인지는 모르겠으나,  
확장자가 java인 파일도 인터프리터를 통해서  
컴파일 + 실행을 할 수 있게 된 것 같다.  

---

### 바이트코드란 무엇인가

바이트코드란 JVM이 읽고 해석할 수 있는 언어이다.  
아까도 잠깐 말했지만, 어떤 운영체제에서도 JVM만 있다면  
Main.class 파일을 실행 시킬 수 있다.  

바이트 코드는 다음과 같다.  

![error](/assets/images/whiteship-live-study/2021-02-17/byte-code.png)  

javap 명령어는 disassembler이다. (역 어셈블러)  
p가 어떤것을 의미하는지는 정확히 모르겠다...  
-c 옵션을 주고 Main 혹은 Main.class를 입력하면  
바이트코드를 볼 수 있다.  
(JDK 11 버전도 javap -c Main.java 는 안된다!!)  

바이트코드를 해석하는 일은 쉽지 않고, 꼭 알아야 하는 건 아니지만,    
대체 어떻게 동작되는건지 전혀 모르겠는 코드 혹은 문법은 바이트 코드를 보는 것도 도움이 된다.  

예를 들어 12주차 애노테이션 스터디에서 학습한 자료이다.  
(시간 상 안맞지만 4주차 이전 스터디는 거꾸로 진행해서 가지고 있다..)    

```java
@Retention(RetentionPolicy.CLASS)
public @interface MyAnnotaion {}


public class Example {

  @MyAnnotaion
  public static void main(String[] args) {
  }
  
}
```

사용자 정의 애노테이션이다.  

```java
@Retention(RetentionPolicy.CLASS)
```

위 옵션은  
컴파일 단계까지만 애노테이션의 기능을 수행한다. 즉 실행 후엔 작동하지 않는다.  
바이트 코드를 살펴보자.  

![error](/assets/images/whiteship-live-study/2021-02-05/retention_class.png)   

메인 메서드에 @MyAnnotation 이 붙어있지만 주석으로 invisible이 되어있다.  

```java
@Retention(RetentionPolicy.CLASS)
```

위 옵션은 소스코드까지만 애노테이션의 기능을 수행하는데,  

![error](/assets/images/whiteship-live-study/2021-02-05/retention_source.png)

메인 메서드 어디에도 @MyAnnotation을 찾을 수 없다.  

이처럼 코드로 찾을 수 없거나, 이해가 난해한 동작을 하는 코드 혹은 문법은  
바이트 코드를 통해 조금 더 심도있게 이해 할 수 있다.  

---

### JVM 구성 요소

#### Class Loader
JVM 내로 클래스 파일을 load하고 link를 통해 배치를 수행하는 모듈이다.  
런타임 시 동적으로 클래스를 로드한다.  

- load : 클래스를 읽어오는 과정
- link : 레퍼런스를 연결하는 과정

#### Execution Engine
클래스 로더를 통해 배치된 클래스를 실행시킨다.  
위에서 보았듯 클래스 파일은 (바이트코드) 비교적 사람이 보기 쉬운 형태이다.  
이를 기계가 실행할 수 있는 형태로 변환시키는데,  
이때 인터프리터, 혹은 JIT 방식을 사용한다.  

#### Interpreter
실행 엔진은 바이트 코드를 명령어 단위로 실행한다.  
한 줄 씩 수행하기 때문에 느리다는 단점이 있다.   

#### Jit Compiler
Just In Time Compiler의 약자로 기존 인터프리터 방식의 단점인 속도를 보완하기 위해 도입된 방식이다.  
인터프리터 방식으로 실행하다 적절한 시점에 바이트 코드 전체를 컴파일하여 네이티브 코드로 변경하고,  
더 이상 인터프리팅 하지않고 직접 네이티브 코드로 실행하는 방식이다.  
네이티브 코드는 캐시에 보관되기 때문에 한번 컴파일 된 코드는 빠르게 실행할 수 있다.  

그러나 이러한 방식도 JIT 컴파일러가 컴파일 하는 과정이 인터프리팅하는 것 보다 더 오래걸리므로  
한 번만 실행되는 코드는 컴파일하지 않고 인터프리팅을 하는 것이 유리하다.  
이를 위해 JIT 컴파일러를 사용하는 JVM은 내부적으로 해당 메서드가 얼마나 자주 실행되는지를 체크하고  
일정 정도를 넘을 때에만 컴파일을 수행한다.  

#### Garbage Collector
Heap 메모리에 올라가는 객체들을 관리한다.  
더 이상 참조되지 않는 메모리를 반환해준다.  
C와 같은 언어에서는 객체를 사용하지 않는다면 필수적으로 해제를 해주어야하는데,  
자바는 가비지 컬렉터에 의해 반환되기 때문에 개발자의 일이 줄어든다.  

JVM이 OS에 메모리를 추가적으로 요청했을 때 실행되며,  
24시간 내내 돌아가는 서버 프로그램의 경우, JVM이 한가한 시점에 실행된다.  

---

### JDK와 JRE의 차이

#### JRE란
- 자바 애플리케이션을 실행 할 수 있도록 구성된 배포판이다.  
- JVM과 핵심 라이브러리 및 자바 런타임 환경에서 사용하는 프로퍼티 세팅이나, 리소스 파일을 가지고 있다.  
- 그러나 개발 관련 도구는 포함되어있지 않다.  

즉 JRE는 자바 프로그램 실행용 도구이다.  

#### JDK란
- JRE를 포함하며 개발에 필요한 도구인 javac, javap를 비롯한 javadoc, jconsole 등이 포함되어 있다.    
- JAVA 11 버전 부터는 JDK만 제공하며 JRE는 제공하지 않는다.  

실질적으로 JDK라는 이름하에 JRE가 포함되어 있다.  

---

# 참고
[선장님 - 인프런 강의 더 자바, 코드를 조작하는 다양한 방법](https://www.inflearn.com/course/the-java-code-manipulation?inst=c160e128)  
[Jbee님 블로그 - #자바가상머신, JVM(Java Virtual Machine)이란 무엇인가?](https://asfirstalways.tistory.com/158)  

---





