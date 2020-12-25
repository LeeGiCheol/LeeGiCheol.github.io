---
title : "WhiteShip Live Study 6주차. 상속"
categories : 
    - whiteship-live-study
tag :
    - whiteship-live-study
toc : true
---

## WhiteShip Live Study 6주차. 상속

---

### 학습할 것
- 자바 상속의 특징
- super 키워드
- 메소드 오버라이딩
- 다이나믹 메소드 디스패치 (Dynamic Method Dispatch)
- 추상 클래스
- final 키워드
- Object 클래스

---

### 자바 상속의 특징

상속은 기존 클래스의 변수와 메서드를 재사용해 새로운 클래스를 만드는 것을 말한다.  
보통 부모와 자식 관계, 혹은 조상과 자손 관계 라고 부른다.

---

### 상속을 하기 위한 키워드

자바에서는 extends 라는 키워드를 클래스에 붙여 상속을 받을 수 있다.

```java
public class Parent {
    int number = 10;
    
    void print() {
        System.out.println("Parent");
    }
}

public class Child extend Parent {

}

public class Main {
    Child child = new Child();
    System.out.println(child.number); // 10
    child.print(); // Parent
}
```

Child 클래스는 보다시피 아무것도 정의 되어있지 않고, 그저 Parent 클래스만 상속받았다.  
신기하게도 Child 클래스에 있지도 않은 number를 호출 할 수 있고, print 메서드를 호출 할 수 있다.  
**단 생성자와 초기화 블럭은 제외이다.**

<br>

**다중상속은 불가하다.**

```java
public class GrandFather {
    void eyes() {
        System.out.println("Black");
    }
}

public class Father {
    void eyes() {
        System.out.println("Brown");
    }
}

public class Me extends GrandFather, Father {

}
```

같은 객체지향언어인 C++에서는 다중상속을 허용하지만 자바는 다중상속이 불가능하다.  
위와 같은 모양새로는 사용할 수 없다는 뜻이다.  
다중상속이 가능하다면 여러 클래스를 상속 받을 수 있기 때문에 복합적인 기능을 가진 클래스를 쉽게 작성할 수 있지만,  
상속받은 클래스의 멤버의 이름이 같다면 구분할 방법이 없다.  

위의 예제에서 Me 클래스가 eyes라는 메서드를 사용하려는데  
이게 GrandFather 클래스의 eyes인지, Father 클래스의 eyes인지 구분할 방법이 없기 때문이다.

---

5주차 클래스에서 잠깐 다뤘던 얘기이지만 자바에서의 모든 클래스는 Object 클래스를 상속받는다.  
위와같이 다중상속이 불가능한 자바에서 Object는 어떻게 상속 받을까?


```java
public class Parent {
    int number = 10;
    
    void print() {
        System.out.println("Parent");
    }
}

public class Child extend Parent {

}

public class Main {
    Child child = new Child();
    System.out.println(child.number); // 10
    child.print(); // Parent
}
```

다시 위에서 봤던 예제이다.  
이미 어떤 클래스를 상속받은 클래스는 컴파일러가 **extends Object**를 추가하지 않는다. (예제에서의 Child)
대신 Child 클래스는 Parent 클래스를 상속받고 Parent 클래스는 Object 클래스를 상속받는다.  
이렇게 모든 클래스는 Object 클래스를 상속받을 수 있게 된다.

---

### super키워드

```java
public class Parent {
    int number = 10;
    
    void print() {
        System.out.println("Parent");
    }
}

public class Child extend Parent {
    int number = 100;

    void superKeyword() {
        super.print();
        System.out.println(number);
        System.out.println(super.number);
        System.out.println(this.number);
    }
}

public class Main {
    Child child = new Child();
    child.superKeyword();
}

// 결과
Parent
100
10
100
```
---

지난시간 알아봤던 this 키워드와 조금 다르다.  
super 키워드를 통해 부모 클래스의 멤버에 접근할 수 있다.

**super()**

```java
public class Parent  {
    int x;
    int y;

    Parent() {
        System.out.println(this.x);
    }

    Parent(int x) {
        this.x = x;
        System.out.println(this.x);
    }

    Parent(int x, int y) {
        this.x = x;
        this.y = y;
        System.out.println(this.x);
        System.out.println(this.y);
    }
}

public class Child extends Parent {
    Child() {
        super();
    }

    Child(int x) {
        super(x);
    }

    Child(int x, int y) {
        super(x, y);
    }
}

public class Main {
    public static void main(String[] args) {
        Child child1 = new Child();
        Child child2 = new Child(10);
        Child child3 = new Child(100, 200);
    }
}

// 결과
0
10
100
200
```

super()는 상속받은 바로위의 클래스의 생성자를 호출한다.  
위와같이 Parent 클래스의 생성자가 여러개라면 마치 오버로딩된 메서드를 호출하듯이,  
파라미터의 타입이나, 개수에 맞는 생성자를 찾아 호출한다.

---

### 메서드 오버라이딩

오버로딩과 이름이 비슷하여 헷갈릴 수 있지만 차이는 명백하다.  
오버로딩은 이름이 같은 새로운 메서드를 추가하는 것이고,  
오버라이딩은 부모클래스의 메서드를 자식클래스에서 재정의 하는것을 의미한다.

```java
public class Point  {
    int x;
    int y;

    String getLocation() {
        return "x : " + x + ", y : " + y;
    }

}

public class Point3D extends Parent {
    int z;

    @Override
    String getLocation() {
        return "x : " + x + ", y : " + y + ", z : " + z;
        // return super.getLocation() + ", z : " + z;
    }
}
```

<br>

**@Override**  
우선 @Override 라는 것은 Annotation 이라는 것이다.  
JDK 1.5 부터 생긴 오버라이드 애노테이션은 사실 붙여도 안붙여도 동작에는 문제가 되진 않는다. 

하지만 개발자가 Point 클래스의 getLocation 메서드를 getPoint로 변경하게 된다면 컴파일 에러가 발생한다.  
또한 IDE는 똑똑하기 때문에 IntelliJ는 애노테이션에, Eclipse는 메서드에 컴파일 전부터 에러가 났음을 인식한다.

IntelliJ  
![error](/assets/images/whiteship-live-study/2020-12-21/IntelliJ.png)

Eclipse  
![error](/assets/images/whiteship-live-study/2020-12-21/Eclipse.png)

이처럼 @Override 를 붙여서 안전장치 역할을 해줄 수 있고, 오버라이드 된 메서드라는 걸 명시적으로 보여줄 수 있다.

---

다시 본문으로 돌아가서 getLocation을 보면 Point 클래스는 점을 표현하기 위해서 2개의 좌표가 필요했지만,  
Point3D 클래스는 3차원으로 3개의 좌표가 필요하다.  
이런 경우 z 좌표를 추가하여 오버라이딩 할 수 있다.

물론 당연히 완전히 새롭게 만드는 것도 가능하며,  
위와 같은 경우라면 조금 전에 공부했던 super 키워드를 통해 조금 더 간단하게 표현할 수도 있다.

---

**오버라이드의 조건**
- 선언부가 부모 클래스의 메서드와 일치해야 한다.
- 접근제한자는 부모 클래스의 메서드보다 더 좁은 범위로 변경할 수 없다.
- 부모 클래스의 메서드보다 많은 수의 예외를 선언할 수 없다.


---

### 다이나믹 메서드 디스패치 (Dynamic Method Dispatch)
 
```text
1. (특히 특별한 목적을 위해) 보내다[파견하다]
2. (편지소포메시지를) 보내다[발송하다]
...
```
네이버 영어사전을 참고한 Dispatch의 뜻이다.

Dispatch는 보내다 라는 뜻을 가졌다.  
프로그래밍에서 메시지를 보낸다는 것은 곧 메서드 호출이 된다.  
그렇기에 Method Dispatch는 메서드를 호출하는 것을 의미한다고 생각하면 될 것 같다.  
Method Dispatch의 종류로는 Static Dispatch와 Dynamic Dispatch가 있다.

---

Static Dispatch와 Dynamic Dispatch를 알아보기 전에,  
Runtime과, Compile Time에 대해 간략하게 짚고 넘어가겠다.  

```text
- Compile Time (컴파일)
    Java와 같은 프로그래밍 언어로 작성된 소스코드를  
    컴퓨터가 인식 할 수 있는 기계어로 변환되어 실행 가능한 프로그램이 되는 과정
        ex) java 파일을 javac Main.java 명령어를 사용하면 Main.class 파일이 생성되는 과정 

- RunTime (런타임)
    컴파일 과정을 마친 응용프로그램이 사용자에 의해 실행되는 때
        ex) class 파일을 java Main 명령어를 사용할 때
  
- Compile Time Error
    대표적으로 세미콜론을 쓰지않거나, 키워드의 맞춤법이 틀린 등 문법이 잘못됬을 때 발생하는 Syntax Error가 있다. 
    IDE를 사용 할 경우 실행 전부터 IDE가 빨간줄을 그어버린다. 
     
- RunTime Error 
    이미 컴파일이 완료되었으나 의도치 않은 예외 상황으로 발생하는 에러이다.
    대표적으로 객체를 생성하지 않고 Null Object를 참조하려고해서 발생하는 NPE (NullPointException),   
    정수를 0으로 나눗셈을 하면 발생하는 ArithmeticException 등이 있다.
```

---

**Static Dispatch**

```java
public class Main {
    public static void main(String[] args) {
        Dispatch dispatch = new Dispatch();
        dispatch.print();
    }
}

class Dispatch {
    void print() {
        System.out.println("Static Dispatch!");
    }
}
```

**Static Dispatch**는 구현클래스를 이용해 컴파일 타임에서부터 어떤 메서드가 호출될지 정해져있는 것이다.  
자바에서 객체 생성은 런타임시에 호출된다.  
컴파일 시점에 Dispatch라는 구현 클래스의 정보를 알 수 있는데,  
그렇기에 메서드를 호출하면 어떤 메서드가 호출될지 정적으로 정해질 수 있다.


**Dynamic Dispatch**

```java
public class Main {
    public static void main(String[] args) {
        Dispatch dispatch = new DispatchImpl();
        dispatch.print();
    }
}

class DispatchImpl implements Dispatch {
    @Override
    public void print() {
        System.out.println("Dynamic Dispatch!");
    }
}

interface Dispatch {
    void print();
}
```

반면 **Dynamic Dispatch**는 인터페이스를 이용해 호출되는 메서드가 동적으로 정해지는 것을 말한다.  
컴파일 시점에는 어떤 메서드가 실행되는지 모르고, 인터페이스의 (추상클래스) 메서드가 호출되는 것만 알고있다.  
런타임 시점에 Dispatch에 할당된 객체가 무엇인지 확인하고 메서드를 호출하게 된다.

**Double Dispatch**
Static이던, Dynamic이던 Dispatch가 한번 일어난 시점에서  
Dynamic Dispatch가 한번 더 일어나면 Double Dispatch가 된다.

정확히 맞는 예시인지는 모르겠지만 나는 여기서 팩토리패턴이 생각났다.  
팩토리패턴은 클래스의 인스턴스를 만드는 일을 서브클래스에게 맡기는 것을 의미한다.  

```java
public interface Shape {
	void draw();
}

public class Circle implements Shape {
  @Override
  public void draw() {
    System.out.println("Circle - draw() Method.");
  }
}


public class Square implements Shape {
  @Override
  public void draw() {
    System.out.println("Square - draw() Method.");
  }
}


public class Rectangle implements Shape {
  @Override
  public void draw() {
    System.out.println("Rectangle - draw() Method.");
  }
}


public class ShapeFactory {

  public Shape getShape(String shapeType) {
    if(shapeType == null) {
      return null;
    }
  
    if(shapeType.equalsIgnoreCase("CIRCLE")) {
      return new Circle();
    }
    else if(shapeType.equalsIgnoreCase("RECTANGLE")) {
      return new Rectangle();
    }
    else if(shapeType.equalsIgnoreCase("SQUARE")) {
      return new Square();
    }
  
    return null;
  }
}


public class FactoryPatternTest {
  public static void main(String[] args) {
    ShapeFactory shapeFactory = new ShapeFactory();

    Shape shape1 = shapeFactory.getShape("CIRCLE");
    shape1.draw();
    
    Shape shape2 = shapeFactory.getShape("RECTANGLE");
    shape2.draw();
    
    Shape shape3 = shapeFactory.getShape("SQUARE");
    shape3.draw();
  }
}

// 결과
Circle - draw() Method.
Rectangle - draw() Method.
Square - draw() Method.
```

위의 예제를 보면 ShapeFactory에 의해서 도형의 모양이 정해진다.  
구현클래스인 ShapeFactory에 의해 Static Dispatch 한번 발생하고, 
draw() 메서드를 찾기 위해 Dynamic Dispatch 한번 발생하게 된다.

---



### 추상클래스
지난주 스터디 주제인 클래스는 보통 설계도로 예시를 많이 든다.  
추상 클래스같은 경우 미완성 설계도라고 생각하면 좋을 것 이다.  
자체적인 클래스 역할보다는 새로운 클래스를 작성하는데 있어 바탕이 되는 조상 클래스로서의 의미가 크다.

예를들어 강아지 클래스와 고양이 클래스가 있다고 가정하자.  
둘 다 '운다' 라는 공통점은 있지만 울음소리는 분명히 다르다.  
이럴 경우 '동물' 이라는 추상 클래스를 만들고 '운다' 라는 행위를 추상 메서드로 만들어 사용할 수 있다.


```java
public abstract class Animal {
    abstract void cry();
}

public class Dog extends Animal {
    void cry() {
      System.out.println("멍멍!");
    }
}

public class Cat extends Animal {
    void cry() {
      System.out.println("야옹~");
    }
}
```

이처럼 abstract 키워드를 사용해 추상클래스를 만들 수 있다.  
추상메서드는 abstract 키워드를 사용하고 뒤는 중괄호 '{}' 가 아닌 세미콜론 ';'을 사용하며, 구현부는 존재하지 않는다.

추상클래스는 추상메서드를 가질 수 있는 것을 제외하면  
일반 클래스처럼 생성자, 일반메서드, 멤버변수를 가질 수 있다는 특징이 있다.

---

### final 키워드

final 키워드는 **변경될 수 없는** 이라는 의미를 가지고 있다.  
대부분의 대상에 붙일 수 있는 키워드이다.

![error](/assets/images/whiteship-live-study/2020-12-21/final.png)

변수에 final 키워드를 붙여 상수로 만들어 사용 할 경우  
변수명은 일반적으로 상수라는 것을 알기 쉽게 하기위해 모두 대문자로 작성한다.

```java
public class Main {
    final int IMMUTABLE = 10;
}
```

대표적인 final 클래스는 String이다.  
String은 불변이라는 특징을 가지고 있다.  

```java
public class Main { 
    public static void main(String[] args) {
        String str = "One ";
        str += "Two";
        System.out.println(str);
  }
}
```
출력되는 결과는 당연히 "One Two" 일 것이다.

그러나 첫번째 줄의 str과 두번째 줄의 str이 참조하는 주소는 다르다.  
String은 불변의 특징을 가지기 때문에 문자열 연산과 같은 작업 시 새로운 문자열을 만들기 때문이다.

```java
public class Main {
    public static void main(String[] args) {
        String str = "One ";
        System.out.println(str.hashCode());
        System.out.println(str.hashCode());

        str += "Two";
        System.out.println(str.hashCode());
        System.out.println(str.hashCode());
    }
}
// 결과
2462362     // str(One)
2462362     // str(One)
341866834   // str(Two)
341866834   // str(Two)
```
---

### Object 클래스

java.lang 패키지는 자바에서 가장 기본이 되는 클래스들이 포함되어 있으며, 그렇기 때문에 import문 없이도 사용가능하다.  
별 생각없이 사용해왔던 System.out.println 의 System, String, Math와 같은 클래스들은 java.lang 패키지에 포함되어있다.

Object 클래스 역시 java.lang 패키지에 속해있다.

Object 클래스는 모든 클래스의 최고 조상이기 때문에 모든 클래스에서 Object 클래스의 멤버를 사용할 수 있다.

![error](/assets/images/whiteship-live-study/2020-12-21/Object.png)

위 사진과 같이 분명 Object를 상속받지 않았지만, Object 클래스의 멤버를 Override 할 수도 있다.

문자열 비교할때 많이 사용했던 equals()나  
바로 위에서 객체의 해시코드를 확인할 수 있었던 hashcode(),  
객체의 정보를 문자열로 반환하는 toString(),  
클래스 인스턴스를 반환하는 getClass() 와 같이 익숙한 메서드들이 많이 보인다.

---