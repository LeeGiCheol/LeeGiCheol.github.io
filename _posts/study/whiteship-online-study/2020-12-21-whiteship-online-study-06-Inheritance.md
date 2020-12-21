---
title : "WhiteShip Live Study 5주차. Class"
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
다중상속이 가능하다면 여러 클래스를 상속 받을 수 있기 때문에 복합적인 기능을 가진 클래스를 쉽게 작성할 수 있지만, 상속받은 클래스의 멤버의 이름이 같다면 구분할 방법이 없다.

위의 예제에서 Me 클래스가 eyes라는 메서드를 사용하려는데 이게 GrandFather 클래스의 eyes인지, Father 클래스의 eyes인지 구분할 방법이 없기 때문이다.

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
위와같이 Parent 클래스의 생성자가 여러개라면 마치 오버로딩된 메서드를 호출하듯이, 파라미터의 타입이나, 개수에 맞는 생성자를 찾아 호출한다.

---

### 메서드 오버라이딩

오버로딩과 이름이 비슷하여 헷갈릴 수 있지만 차이는 명백하다.
오버로딩은 이름이 같은 새로운 메서드를 추가하는 것이고, 오버라이딩은 부모클래스의 메서드를 자식클래스에서 재정의 하는것을 의미한다.

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

이처럼 @Override 를 붙여서 안전장치 역할을 해줄 수 있고, 오버라이드 된 메서드라는 걸 명시적으로 보여줄 수 있다.

---

다시 본문으로 돌아가서 getLocation을 보면 Point 클래스는 점을 표현하기 위해서 2개의 좌표가 필요했지만, Point3D 클래스는 3차원으로 3개의 좌표가 필요하다. 이런 경우 z 좌표를 추가하여 오버라이딩 할 수 있다.

물론 당연히 완전히 새롭게 만드는 것도 가능하며, 
위와 같은 경우라면 조금 전에 공부했던 super 키워드를 통해 조금 더 간단하게 표현할 수도 있다.

