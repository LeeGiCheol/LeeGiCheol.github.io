---
title : "WhiteShip Live Study 5주차. Class"
categories : 
    - whiteship-live-study
tag :
    - whiteship-live-study
toc : true
---

## WhiteShip Live Study 5주차. Class

---

### 학습할 것
- 클래스 정의하는 방법
- 객체 만드는 방법 (new 키워드 이해하기)
- 메소드 정의하는 방법
- 생성자 정의하는 방법
- this 키워드 이해하기
- super 키워드 이해하기

### 과제
- int 값을 가지고 있는 이진 트리를 나타내는 Node 라는 클래스를 정의하세요.
- int value, Node left, right를 가지고 있어야 합니다.
- BinrayTree라는 클래스를 정의하고 주어진 노드를 기준으로 출력하는 bfs(Node node)와 dfs(Node node) 메소드를 구현하세요.
- DFS는 왼쪽, 루트, 오른쪽 순으로 순회하세요.

---

### 클래스(Class)란

- 자바에서 말하는 클래스란 객체를 정의하는 
  틀 또는 설계도와 같은 의미로 사용된다.
- 변수, 메서드를 가진 집단이다.
- class 키워드로 정의할 수 있다.

**클래스를 정의하는 방법**

```java
public class 클래스이름 {

}
```

위와 같이 class 키워드를 사용하여 정의할 수 있다.


**Object Class**
- 자바의 모든 클래스는 기본적으로 Object 클래스를 상속받는다.

```java
public class Main {
    public static void main(String[] args) {
        System.out.println(Main.class.getSuperclass());
    }
}
```

위의 코드를 실행하면 이러한 결과를 얻을 수 있다.

```
class java.lang.Object
```
<br>

**그런데 자바는 다중상속이 불가능하지 않나?**

다중상속 불가능하다.

```java
public class A extends B {

}
```

하지만 위와같이 B클래스를 상속받는 A클래스가 있다면,  
A는 B를 상속받고 B는 Object 클래스를 상속받아 
결과적으로 A또한 Object 클래스를 상속받게 된다.

### 객체를 만드는 방법

new 키워드를 사용하여 객체를 만들 수 있다.

클래스 변수명 = new 클래스();

```java
public class Person {
    private String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```

```java
public class Main {
    public static void main(String[] args) {
        Person person = new Person(); // Person 객체 생성
        person.setName("gicheol"); 
        
        System.out.println(person.getName()); // gicheol
    }
}
```

이름을 저장하는 Person 클래스를 만들고  
Main 클래스에서 객체를 생성했다.  
new 키워드는 Heap 영역의 메모리에 데이터를 저장할 공간을 할당받고  
그 공간의 참조값을 객체에게 반환해 주고 생성자를 호출하게 된다.  

```java
public class Main {
  public Main();
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: return

  public static void main(java.lang.String[]);
    Code:
       0: new           #2                  // class Person
       3: dup
       4: invokespecial #3                  // Method Person."<init>":()V
       7: astore_1
       8: aload_1
       9: ldc           #4                  // String gicheol
      11: invokevirtual #5                  // Method Person.setName:(Ljava/lang/String;)V
      14: getstatic     #6                  // Field java/lang/System.out:Ljava/io/PrintStream;
      17: aload_1
      18: invokevirtual #7                  // Method Person.getName:()Ljava/lang/String;
      21: invokevirtual #8                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
      24: return
}
```

온라인스터디에서 바이트코드가 핫하길래 나도 한번 확인해보았다.  

main 메서드의 new는 해당 클래스의 새 인스턴스에 필요한 메모리를 Heap안에 할당하고,  
할당된 위치를 가리키는 참조를 스택에 쌓는다.  

invokespecial은 생성자, 현재 클래스, 수퍼클래스의 메서드를 호출한다.  

사실 바이트코드는 딥한 영역이다 보니  
본적도 없을 뿐더러, 해석도 쉽지않다.  
그렇지만 new 연산자가 실제 작동하는 모습을 눈으로 보니 색다른 느낌이었다.  

### 메서드(Method)란
메서드는 특정 기능을 정의한 코드들의 집합이다.  
함수와는 다르게 독립적으로 존재하지 못하고 클래스에 종속되어야만 한다.  
두 가지는 엄연히 다르지만 함수와 메서드를 혼용해서 사용하는 경우가 많긴 한 것 같다.  

**메서드 정의하는 방법**

![error](/assets/images/method.png)

요소 
1. **접근제어자** : 해당 메서드에 접근할 수 있는 범위  
2. **리턴타입** : 메서드가 작업을 마치고 반환하는 데이터의 타입  
3. **메서드이름** : 메서드를 호출하기 위한 이름  
4. **파라미터** : 메서드 호출 시 전달되는 변수  
이 값은 지역변수로 메서드 호출이 끝나면 소멸되어 사용할 수 없다.  
5. **리턴 값** : 메서드가 작업을 마치고 반환하는 데이터  

리턴타입이 void가 아닌 경우 반드시 return문이 필요하다.   
그러나 void인 경우 컴파일러가 메서드의 마지막에 'return;'을   
자동적으로 추가해주기 때문에 리턴값은 생략 가능하다.  



### 생성자란

생성자는 인스턴스가 생성될 때 호출되는 '인스턴스 초기화 메서드' 이다.  
메서드처럼 클래스 내에 선언되며, 리턴값이 없다.  
모든 생성자는 리턴값이 없기 때문에 void는 생략할 수 있다.  

- 생성자의 조건
    - 생성자의 이름은 클래스의 이름과 같아야 한다.
    - 생성자는 리턴 값이 없다.

모든 클래스에는 생성자가 반드시 하나 이상 정의되어야 한다.  
그러나 생성자를 정의하지 않으면 컴파일러에 의해   
**기본 생성자**가 추가되기 때문에 생략 가능하다.  
주의할 점은 컴파일러가 기본 생성자를 만들어주는 경우는  
**생성자가 하나도 없을 때** 뿐이다.   

```java
public class Person {
    // public Person() {}  컴파일러가 만들어주는 기본 생성자
}
```

```java
public class Person {
    String name;

    // 직접 정의한 생성자
    public Person(String personName) {
        name = personName;
    }
}
```

위와 같이 기본생성자 없이 정의한 생성자만을 사용한다면   
객체를 생성할 때 주의해야한다.  

```java
public class Main {
    
    public static void main(String[] args) {
        Person person1 = new Person(); // compile error

        Person person2 = new Person("gicheol"); // OK
    }
}
```

**인수(argument)와 매개변수(parameter)**

인수는 메서드나 생성자를를 호출할때, 매개변수는 선언할때 괄호안에 적는 값을 의미한다.  

인수는 매개변수에 대입되는 실제로 메모리에 할당되어 있는 변수이며,  
매개변수는 실제로 값이 존재하지 않는다.  

```java
public class Main {

    public static void main(String[] args) {
        argsAndParam("인수 - argument"); // 호출할때는 인수
    }

    public static void argsAndParam(String parameter){ // 선언할때는 매개변수
        System.out.println("매개변수 - parameter");
    }
}
```

### this 키워드 이해하기

this는 **인스턴스 자기 자신**을 가리키는 참조변수이다.  

this 키워드를 이용해 생성자나 setter 메서드에 인스턴스의 필드 데이터에 접근할 수 있다.  

```java
public class Person {

    public Person getThis() {
        return this;
    }
}

public class Main {
    public static void main(String[] args) {
        Person person = new Person();
        
        System.out.println(person);
        System.out.println(person.getThis());
    }
}
```

```java
homework.livestudy.bst.Person@17c68925
homework.livestudy.bst.Person@17c68925
```

this가 인스턴스 자기 자신을 가리키는 참조변수라 했는데,  
위의 예제를 출력해보면 Person 객체의 참조변수인  
person과 getThis()의 값은 동일하다는 것을 알 수 있다.  

**참조변수**

this가 자기 자신을 가리킨다는 의미를 받아들이기 좋은 예제가  
Call By Value, Call By Reference 라고 생각했다.   
예제는 swap으로 준비했다.  

```java
public class CallByValue {
    static void swap(int num1, int num2) {
        int temp = num1;
        num1 = num2;
        num2 = temp;
    }

    public static void main(String[] args) {

        int num1 = 10;
        int num2 = 20;

        System.out.println("swap 호출 전 : " + num1 + ", " + num2);

        swap(num1, num2);

        System.out.println("swap 호출 후 : " + num1 + ", " + num2);
    }
}
```

```java
swap 호출 전 : 10, 20
swap 호출 후 : 10, 20
```

메인 메서드의 num1과 num2가 당연히 바뀌어야 할 것 같지만 바뀌지 않는다.  
위에서 잠깐 다뤘지만 매개변수의 값은 실제하는 값이 아니고 인자 값이 복사되는 값이기 때문이다.   

```java
public class CallByReference {

    int value;

    public CallByReference(int value) {
        this.value = value;
    }

    static void swap(CallByReference num1, CallByReference num2) {
        int temp = num1.value;
        num1.value = num2.value;
        num2.value = temp;
    }

    public static void main(String[] args) {
        CallByReference num1 = new CallByReference(10);
        CallByReference num2 = new CallByReference(20);
        System.out.println("swap 호출 전 : " + num1.value + ", " + num2.value);

        swap(num1, num2);

        System.out.println("swap 호출 후 : " + num1.value + ", " + num2.value);
    }
}
```

```java
swap 호출 전 : 10, 20
swap 호출 후 : 20, 10
```

Call By Reference는 주소의 값이 전달되기때문에 참조변수의 값이 변경이 된다.  

최근에 Hard Link, Symbolic Link에 대해 잠깐 보게되었는데 비슷한 느낌을 받았다.  

### super 키워드 이해하기
super 키워드는 자식 클래스에서 상속받은 부모 클래스의 멤버변수를 참조할때 사용된다.  

```java
public class Parent {
    int number = 10;
}

public class Child extends Parent {
    int number = 100;

    public void getNumber() {
        System.out.println(number);       // 100
        System.out.println(this.number);  // 100
        System.out.println(super.number); // 10
    }
}

public class Main {
    public static void main(String[] args) {
        Child child = new Child();
        child.getNumber();
    }
}
```

Parent를 상속받은 Child 클래스는 위의 코드와 같이  
super.number를 통해 Parent의 멤버변수에 접근할 수 있다.   
