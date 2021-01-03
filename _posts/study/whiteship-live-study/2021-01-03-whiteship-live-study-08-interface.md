---
title : "WhiteShip Live Study 8주차. 인터페이스"
categories :
- whiteship-live-study
tag :
- whiteship-live-study
toc : true
---

## WhiteShip Live Study 8주차. 인터페이스

---

## 목표
자바의 인터페이스에 대해 학습하세요.

## 학습할 것 (필수)
- 인터페이스 정의하는 방법
- 인터페이스 구현하는 방법
- 인터페이스 레퍼런스를 통해 구현체를 사용하는 방법
- 인터페이스 상속
- 인터페이스의 기본 메소드 (Default Method), 자바 8
- 인터페이스의 static 메소드, 자바 8
- 인터페이스의 private 메소드, 자바 9

---

### 인터페이스란?
6주차에 공부했던 추상클래스 기억날 것 이다.  
인터페이스란 일종의 추상클래스이며, 추상클래스보다 추상화의 정도가 더 높다.  
추상클래스는 추상메서드 이외에도 구현부가 있는 일반메서드나, 변수를 사용할 수 있는 반면,  
인터페이스는 **<span style="color: rgb(161, 65, 0)">오직 추상메서드와, 상수만을</span>** 가질 수 있다. (JDK 1.8 이전 기준)  

---

### 의문
인터페이스는 추상메서드와 상수만을 사용할 수 있다.  
그런데 추상메서드에 abstract 키워드를 붙였던 기억이 잘 나지 않아서 테스트를 해봤다.  
abstract을 붙이지 않은 메서드와, final을 붙이지 않은 변수를 선언해보았다.  
컴파일 에러가 날 것 같았지만 IDE에서 에러를 잡지 않았다.

```java
public interface InterfaceTest {
    int variable = 10;
    public static final int CONST = 100;
    void method();
}
```

무슨일일까?  
역시 이럴땐 바이트코드를 보면 알 수 있을 것이다.

![error](/assets/images/whiteship-live-study/2021-01-03/interface_byteCode_1.png)

method 메서드는 자동으로 public abstract이 붙었고,  
변수로 선언한 variable은 자동으로 public static final이 붙어 상수가 된걸 확인할 수 있었다.  
인텔리제이가 똑똑해서 그런가 싶어서 터미널로도 해봤다.

![error](/assets/images/whiteship-live-study/2021-01-03/interface_byteCode_2.png)

마찬가지였다. 실수를 방지하기 위해 컴파일러가 직접 붙여주는게 아닐까 생각이 든다.

---

### 인터페이스 정의하는 방법
1. 일반적으로 클래스를 정의할 때 사용하는 키워드인 class 대신 interface를 사용한다.
2. 모든 변수는(상수) public static final 이 붙어야하며, 생략 시 컴파일러가 자동으로 추가해준다.
3. 모든 메서드는 public abstract 이 붙어야하며, 생략 가능하다.
   - 단, static 메서드와 default 메서드는 예외이다. (JDK 1.8부터)

```java
interface 인터페이스 {
    public static final 타입 상수 = 값;
    public abstract 메서드(매개변수);
}
```

---

### 인터페이스 구현하는 방법
다시 추상클래스를 떠올려보자.  
추상클래스는 클래스를 확장한다는 의미의 extends 키워드를 통해 구현을 했었다.  
인터페이스는 이때 extends가 아닌 implements 키워드를 사용한다.

```java
public interface Animal {
   public void cry();
}

public class Cat implements Animal {
   @Override
   public void cry() {
      System.out.println("야옹");
   }
}

public class Dog implements Animal {
   @Override
   public void cry() {
      System.out.println("멍멍");
   }
}
```

---

### 인터페이스 레퍼런스를 통해 구현체를 사용하는 방법
인터페이스는 다형성을 이용해 구현할 수 있다.  
자식클래스의 인스턴스를 부모클래스의 참조변수로 참조하는 것이 가능하다는 의미이다.  

```java
public interface Speaker {
    void name();
}
```

```java
public class Boss implements Speaker {
   @Override
   public void name() {
      System.out.println("보스 스피커");
   }

   public void base() {
      System.out.println("보스 스피커는 저음 강화 스피커이다.");
   }
}
   
public class Yamaha implements Speaker {
   @Override
   public void name() {
      System.out.println("야마하 스피커");
   }

   public void high() {
      System.out.println("야마하 스피커는 고음 강화 스피커이다.");    
   }
}
```

```java
public class Main {
    public static void main(String[] args) {
        Speaker boss = new Boss();
        Speaker yamaha = new Yamaha();

        boss.name();
        yamaha.name();

        // boss.base();           사용불가
        // yamaha.high();         사용불가
        ((Boss) boss).base();      // 캐스팅으로 사용 가능
        ((Yamaha) yamaha).high();  // 캐스팅으로 사용 가능
    }
}

// 보스 스피커
// 야마하 스피커
// 보스 스피커는 저음 강화 스피커이다.
// 야마하 스피커는 고음 강화 스피커이다.
```

위 Main 클래스에서 보다시피 Speaker 인터페이스에 선언 되어있는 name 메서드는 호출 가능하지만,  
Boss나 Yamaha에 선언된 base나 high는 호출하지 못한다.  
Speaker 타입으로 선언이 됬기 때문인데, 이때 캐스팅을 해주면 호출 할 수 있게된다.  

또한 반대로 ```Boss boss = new Speaker();``` 와 같이는 선언하지 못하고 컴파일 에러가 발생한다.  

---

### 인터페이스 상속
자바는 다중상속이 불가능하다. 그러나 인터페이스는 예외이다.  
아이폰이 첫 출시할 때 전화기, mp3, 인터넷을 한 기기에서 사용할 수 있다는 발표를 했었다.  
스마트폰은 이 모든게 조합된 새로운 기기였기 때문이다.  
이런 스마트폰을 간단하게 코드로 만든다고 하자.

이때 인터페이스 다중상속으로 구현을 한다면,  
이미 구현되있던 전화기, mp3, 인터넷 인터페이스를 활용해서 만들 수도 있을 것 이다.

```java
public interface Phone {
   void call();
}

public interface Internet {
    void internet();
}

public interface Mp3 {
   void mp3();
}
```

```java
public class Smartphone implements Phone, Internet, Mp3 {
   @Override
   public void call() { }
    
   @Override
   public void internet() { }
   
   @Override
   public void mp3() { }
}
```

**또한 인터페이스는 Object 클래스와 같은 최고 조상이 없다는 것을 참고하자.**

---

### 인터페이스의 기본 메소드 (Default Method), 자바 8
위에서 말했다시피 인터페이스는외 추상메서드만을 선언할 수 있었다.  
JDK 1.8 부터 Default Method가 추가되었다.  
인터페이스는 구현보다는 선언에 집중이 되어있는데,  

**무엇때문에 추가되었을까?**

인터페이스에 새로운 메서드를 추가한다는 것은 굉장히 복잡한 일이 될 수 있다.  
항상 추상메서드를 추가해야한다는 것이기 때문이다.  
해당 인터페이스를 구현한 모든 클래스에 새로운 메서드를 구현해줘야된다.  

이를 보완하기 위해 Default Method가 추가된 것이다.  

default 키워드를 앞에 붙여 사용하며, 일반 메서드처럼 구현부가 있어야 한다.  

default가 붙지만 접근제어자는 public이며 생략 가능하다.  

```java
public interface Keyboard {
    void type();     // 추상메서드
    
    default void typing() {
       System.out.println("키보드로 타이핑을 할 수 있다.");
   }
}
```

Keyboard 클래스에 typing 메서드가 새롭게 추가 되었다고 생각해보자.  
추상메서드가 아니고 일반메서드이기 때문에 이를 구현한 클래스는 변경할 것이 하나도 없다.  
       
### default 메서드가 충돌이 난다면

만약 메서드 이름이 중복되어 충돌되는 상황이라면 

1. 클래스에서 default 메서드를 재정의했다면 첫번째 우선 순위이다.  
2. 상속구조의 인터페이스에서 자식 인터페이스가 부모 인터페이스의 default 메서드를 재정의했다면 두번째 우선 순위이다.
3. 우선 순위가 없는 경우라면 구현 클래스에서 오버라이딩을 해주면 된다.  
   - 이 경우 선언할 때 명시적으로 인터페이스명.super.메서드명 으로 선언한다.   

---

### 인터페이스의 static 메소드, 자바 8

Static Method는 바로 위에서 공부한 디펄트 메서드와 같이 JDK 1.8 부터 추가되었다.  
오버라이딩이 가능한 디펄트 메서드와는 다르게 오버라이딩이 불가하다.  

또한 객체를 만들지 않고, 반드시 클래스명을 통해 호출해야한다.  

```java
public interface Calculator {
    static int sum(int num1, int num2) {
        return num1 + num2;
    }
    
    default int multiple(int num1, int num2) {
        return num1 * num2;
    }
} 
```

![error](/assets/images/whiteship-live-study/2021-01-03/interface_static_method_1.png)  
<br>

Calculator 클래스에서 오버라이딩 가능한 메서드는 multiple 밖에 없다.  

![error](/assets/images/whiteship-live-study/2021-01-03/interface_static_method_2.png)  
<br>

위 사진을 보면 객체를 생성해서 sum 메서드를 호출하는 것은 불가능하다.  
아래와 같이 반드시 **클래스명.메서드명();** 으로 호출해야 한다.     

```java
public class Main {
   public static void main(String[] args) {
      int sum = Calculator.sum(10, 20);
      System.out.println(sum);
   }
}
```

--- 

### 인터페이스의 private 메소드, 자바 9

```text
Private interface methods are supported. 
This support allows nonabstract methods of an interface to share code between them.
```

[출처 : docs.oracle.com](https://docs.oracle.com/javase/9/language/toc.htm#JSLAN-GUID-E409CC44-9A8F-4043-82C8-6B95CD939296)

JDK 9 버전이 되면서 인터페이스에 private 메서드와 private static 메서드를 사용할 수 있게 되었고,  
그로인해 default 메서드와 private 메서드 사이의 공통 코드를 공유할 수 있다.   

```java
public interface Caffe {
   default void americano() {
      start();
      System.out.println("분쇄한 원두를 물과 섞는다.");
   }

   default void caffeLatte() {
      start();
      System.out.println("분쇄한 원두를 우유와 섞는다.");
   }

   private void start() {
      System.out.println("원두를 분쇄한다.");
   }
}
```

다음은 private static 메서드이다.  
default 메서드에서는 static, instance 메서드를 호출 할 수 있지만,  
static 메서드에서는 private static 메서드만 호출 할 수 있다.   

```java

public interface Calculator {

    default int sum(int num1, int num2) {
        startSum();
        // startMultiple(); // 사용 가능
        return num1 + num2;
    }

    private void startSum() {
        System.out.println("덧셈 시작");
    }

    static int multiple(int num1, int num2) {
        //startSum();     // 컴파일 에러
        startMultiple();
        return num1 * num2;
    }
    
    private static void startMultiple() {
        System.out.println("곱셈 시작");
    }
    
}
```

---