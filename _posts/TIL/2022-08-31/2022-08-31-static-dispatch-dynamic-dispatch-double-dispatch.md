---
title : "Static Dispatch, Dynamic Dispatch, Double Dispatch"
categories : 
    - java
tag :
    - java
toc : true
---

# Static Dispatch, Dynamic Dispatch, Double Dispatch


## Dependency

- 의존관계 (dependency relationship)
    - Supplier의 변화가 Client에 영향을 주는 경우 의존 관계이다.
        
        ![Untitled](/assets/images/study/2022-08-31-static-dispatch-dynamic-dispatch-double-dispatch/01-dependency.png)
        

- 객체지향
    - 객체지향은 재사용하기 유용하다.
    - Client가 Supplier를 의존하는 관계라면, Client는 재사용이 어렵다.
    - Client는 컴포넌트 / 서비스가 될 수 없다.
    - 재사용성을 고려한 설계를 잘 담아낸 패턴을 모아둔 것이 GoF의 Design Pattern이다.
    
    - **Component**
        
        💡 *“Component는 이를 만든 개발자의 손이 미치지 않는 곳에서도 
        아무 변경 없이 필요에 따라 확장할 수 있는 소프트웨어 덩어리이다.”*
        
        ![Untitled](/assets/images/study/2022-08-31-static-dispatch-dynamic-dispatch-double-dispatch/02-martin-fowler.png)
        
        ***- Martin Fowler***
        
        [bliki: SoftwareComponent](https://martinfowler.com/bliki/SoftwareComponent.html)
        
        - **Object Pattern**
            
            💡 “***오브젝트 패턴**은 **런타임시에** 바뀔 수 있는, 
            (상속 관계보다) 더 동적인 오브젝트 (의존) 관계를 다룬다.”
            
            **- GoF
            (Erich Gamma, Richard Helm, Ralph Johnson, John Vlissides)***
            
            **⭐ Object Pattern**
            
            ![Untitled](/assets/images/study/2022-08-31-static-dispatch-dynamic-dispatch-double-dispatch/03-object-pattern.png)
            
            

Dependency는 컴파일 타임이 아니라, 런타임시에 `결정 / 구성`되는 오브젝트 의존 관계이다.

1. 구현 대신 인터페이스를 사용한다.
    1. 클래스 의존 관계 제거
    2. 클래스에 대한 의존성은 생성 피턴처럼 3자에게 위임한다.
2. 오브젝트 합성 (Composition) 사용
    1. 오브젝트 합성은 has-a 관계를 말한다.
    2. 각 클래스가 캡슐화되고 자기의 역할에 충실해진다.
    
- Inversion of Control
    
    💡 *Designing Reusable Classes (**Ralph Johnson)***
    
    

---

## Static Dispatch

```java
public class Dispatch {

    public static void main(String[] args) {
        new Service().run();
    }

}

public class Service {
		void run() {
				System.out.println("run()");
		}
}
```

메인 메서드의 run 메서드는 컴파일 시에 실행될지 알고 있다.

컴파일러도 알고, 바이트 코드에도 정보가 남는다.

이런 경우가 `Static Dispatch`이다.

런타임 시점이 되지 않아도 어느 메서드가 실행될지 결정한다.

```java
public class Dispatch {

		public static void main(String[] args) {
        new Service().run(1);
        new Service().run("Dispatch");
    }

}

public class Service {
		void run(String msg) {
				System.out.println("run(" + msg + ")");
		}

		void run(int number) {
				System.out.println("run(" + number + ")");
		}
}
```

마찬가지로 위 코드는 이름이 같은 run 메서드가 두개 있지만 컴파일 시 

컴파일러가 어떤 메서드를 사용할 지 알고 있기 때문에 `Static Dispatch`이다.

---

### Dynamic Dispatch

```java
public class Dispatch {

    public static void main(String[] args) {
        MyService myService1 = new MyService1();
        myService1.run();

        MyService myService2 = new MyService2();
        myService2.run();
    }

}

public abstract class Service {
		abstract void run();
}

public class MyService1 extends Service {

		@Override
		void run() {
				System.out.println("run1");
		}

}

public class MyService2 extends Service {

		@Override
		void run() {
				System.out.println("run2");
		}

}
```

이번에는 추상 클래스를 사용했으며, 각 구현 클래스를 생성자로 만들어 `run` 메서드를 호출하였다.

이것 또한 자료형이 명확하게 정의되어 있기 때문에 

컴파일 시점에 어떤 메서드를 호출할 지 알고 있다. 이 역시 `Static Dispatch`이다.

```java
public class Dispatch {

    public static void main(String[] args) {
        Service service = new MyService1();
        service.run();
    }

}

public abstract class Service {
		abstract void run();
}

public class MyService1 extends Service {

		@Override
		void run() {
				System.out.println("run1");
		}

}

public class MyService2 extends Service {

		@Override
		void run() {
				System.out.println("run2");
		}

}
```

이 경우는 어떨까?

추상 클래스 Service 타입인 `service` 변수는 컴파일 시 run 메서드가 

어떤 구현 클래스의 메서드를 호출해야할지 알 수 없다.

하지만 런타임시에는 `MyService1`이 실행됨을 알 수 있다.

런타임 시 `service` 변수에 할당되어있는 Object가 무엇인지 확인 후,

그 Object에 의해 실행되는 방식이다.

이때 Object에 의해 실행되는 방식이라는 것은 `Receiver Parameter`에 의한 것인데,

`Receiver Parameter`는 다음과 같다.

![Untitled](/assets/images/study/2022-08-31-static-dispatch-dynamic-dispatch-double-dispatch/04-receiver-parameter.png)

자바는 위와 같이 `this` 키워드를 사용할 수 있는데, this는 인스턴스 자기 자신을 가리키는 키워드이다.

![Untitled](/assets/images/study/2022-08-31-static-dispatch-dynamic-dispatch-double-dispatch/05-receiver-parameter.png)

이런식으로 this를 정의 할 수도 있으며, this 키워드는 클래스 내부에 기본적으로 제공된다.

아래 코드를 통해 알 수 있다.

![Untitled](/assets/images/study/2022-08-31-static-dispatch-dynamic-dispatch-double-dispatch/06-receiver-parameter.png)

분명히 같은 이름의 메서드이지만, 파라미터의 개수가 다르기 때문에 정의가 되어야하지만,

키워드로 설정되어있는 this는 보이진 않지만, JVM에 의해 첫 번째 파라미터로 제공되는 것을 알 수 있다.

![Untitled](/assets/images/study/2022-08-31-static-dispatch-dynamic-dispatch-double-dispatch/07-receiver-parameter.png)

그렇기 때문에 this 파라미터를 추가하여도, 실제 메서드를 사용할 때는 인자를 사용하지 않으며,

오히려 사용할 시 오류가 발생하는 것을 볼 수 있다.

위 코드를 바이트코드로 확인해보아도, this 파라미터는 존재하지 않는다.

```java
// class version 61.0 (61)
// access flags 0x21
public class me/gicheol/dynamicdispatchdoubledispatch/ReceiverParameter {

  // compiled from: ReceiverParameter.java

  // access flags 0x0
  Ljava/lang/String; something

  // access flags 0x1
  public <init>()V
   L0
    LINENUMBER 3 L0
    ALOAD 0
    INVOKESPECIAL java/lang/Object.<init> ()V
    RETURN
   L1
    LOCALVARIABLE this Lme/gicheol/dynamicdispatchdoubledispatch/ReceiverParameter; L0 L1 0
    MAXSTACK = 1
    MAXLOCALS = 1

  // access flags 0x0
  anything()V           // <-- 파라미터를 정의 했지만 존재하지 않는다.
   L0
    LINENUMBER 8 L0
    GETSTATIC java/lang/System.out : Ljava/io/PrintStream;
    ALOAD 0
    GETFIELD me/gicheol/dynamicdispatchdoubledispatch/ReceiverParameter.something : Ljava/lang/String;
    INVOKEVIRTUAL java/io/PrintStream.println (Ljava/lang/String;)V
   L1
    LINENUMBER 9 L1
    RETURN
   L2
    LOCALVARIABLE this Lme/gicheol/dynamicdispatchdoubledispatch/ReceiverParameter; L0 L2 0
    MAXSTACK = 2
    MAXLOCALS = 1

  // access flags 0x9
  public static main([Ljava/lang/String;)V
    // parameter  args
   L0
    LINENUMBER 12 L0
    NEW me/gicheol/dynamicdispatchdoubledispatch/ReceiverParameter
    DUP
    INVOKESPECIAL me/gicheol/dynamicdispatchdoubledispatch/ReceiverParameter.<init> ()V
    ASTORE 1
   L1
    LINENUMBER 13 L1
    ALOAD 1
    LDC "Bar"
    PUTFIELD me/gicheol/dynamicdispatchdoubledispatch/ReceiverParameter.something : Ljava/lang/String;
   L2
    LINENUMBER 14 L2
    ALOAD 1
    INVOKEVIRTUAL me/gicheol/dynamicdispatchdoubledispatch/ReceiverParameter.anything ()V
   L3
    LINENUMBER 15 L3
    RETURN
   L4
    LOCALVARIABLE args [Ljava/lang/String; L0 L4 0
    LOCALVARIABLE receiverParameter Lme/gicheol/dynamicdispatchdoubledispatch/ReceiverParameter; L1 L4 1
    MAXSTACK = 2
    MAXLOCALS = 2
}
```

이것을 바로 `Receiver Parameter` 라고 한다.

처음 들은 용어이고, 처음 알게 된 내용이라 아직 명확한 이해는 가지 않지만,

anything 메서드에 Receiver Parameter로 this가 들어가 있는 것으로 이해하면 될 것 같다.

[Chapter 8. Classes](https://docs.oracle.com/javase/specs/jls/se9/html/jls-8.html#jls-8.4.1)

다시 본론으로 돌아가 service 변수가 호출하는 run 메서드가 

MyService1 인 것을 런타임 시 알 수 있는 것은,

객체를 생성하는 생성자인 `new MyService1();` 부분에서 

MyService1의 Receiver Parameter인 this가 있는 것을 확인하여,

여러개의 run 메서드 중에서, MyService1의 run 메서드가 정확하게 실행되는 것이다.

이렇게 객체를 생성하면서 어떤 메서드를 사용할지 확인하기 때문에,

컴파일 시에 알 수 없고, 런타임 시에 동적으로 결정되는 것이다.

이것이 `Dynamic Dispatch`이다.

---

## Double Dispatch

`Double Dispatch`는 Dynamic Dispatch가 두번 발생하는 것을 말한다.

```java
public class Dispatch {

    interface Post {
        void postOn(SNS sns);
    }

    static class Text implements Post {

        @Override
        public void postOn(SNS sns) {
            System.out.println("text -> " + sns.getClass().getSimpleName());
        }

    }

    static class Picture implements Post {

        @Override
        public void postOn(SNS sns) {
						System.out.println("picture -> " + sns.getClass().getSimpleName());
        }

    }

    interface SNS {}

    static class Facebook implements SNS {}

    static class Twitter implements SNS {}

    public static void main(String[] args) {
        List<Post> posts = Arrays.asList(new Text(), new Picture());
        List<SNS> sns = Arrays.asList(new Facebook(), new Twitter());

        posts.forEach(p -> sns.forEach(p::postOn));
    }

}
```

위 코드는 SNS를 구현한 코드이다.

마찬가지로 postOn 메서드를 실행할 때 컴파일 시점은 어떤 메서드를 사용할 지 알지 못하고,

런타임 시점에 결정되는 `Dynamic Dispatch`이다.

---

### instanceof를 사용하기

위 예제의 경우 메인 메서드에서 호출하는 Post 오브젝트가 두개, SNS 오브젝트가 두개이다.

경우에 따라 총 4가지의 경우의 수를 모두 다른 로직을 구현해야 할 수 있다.

`postOn` 메서드의 경우 `Dynamic Dispatch`에 의해 어떤 메서드를 사용할 지 쉽게 결정 될 수 있지만,

그 내부적으로 `SNS`라는 인터페이스를 파라미터로 받기 때문에

컴파일 시점에는 어떤 구현체를 사용하는지 알 수 없다.

이때 가장 쉽게 구현체를 알아낼 수 있는 방법은 `instanceof`이다.

```java
public class Dispatch {

    interface Post {
        void postOn(SNS sns);
    }

    static class Text implements Post {

        @Override
        public void postOn(SNS sns) {
            if (sns instanceof Facebook) {
                System.out.println("text - facebook");
            }
            if (sns instanceof Twitter) {
                System.out.println("text - twitter");
            }
        }

    }

    static class Picture implements Post {

        @Override
        public void postOn(SNS sns) {
            if (sns instanceof Facebook) {
                System.out.println("picture - facebook");
            }
            if (sns instanceof Twitter) {
                System.out.println("picture - twitter");
            }
            
        }

    }

    interface SNS {}

    static class Facebook implements SNS {}

    static class Twitter implements SNS {}

    public static void main(String[] args) {
        List<Post> posts = Arrays.asList(new Text(), new Picture());
        List<SNS> sns = Arrays.asList(new Facebook(), new Twitter());

        posts.forEach(p -> sns.forEach(p::postOn));

    }

}
```

---

## instanceof

instanceof는 객체의 타입을 확인하는 키워드인데, 형변환이 가능한지 true와 false로 리턴한다.

```java
public class MyInstanceOf {

    static class Parent {}

    static class Child extends Parent {}

    public static void main(String[] args) {
        Parent parent = new Parent();
        Child child = new Child();

        System.out.println(parent instanceof Parent);
        System.out.println(child instanceof Child);
        System.out.println(parent instanceof Child);
        System.out.println(child instanceof Parent);
    }

}
```

자바는 상위클래스가 하위클래스로 형변환하지 못하기 때문에 

`parent instanceof Child` 는 false를 리턴한다.

---

### instanceof의 문제점

instanceof는 객체지향적이지 못하다는 치명적인 단점이 있다.

1. 다형성을 통해 외부에서 해당 객체의 정보를 알 수 없는 `캡슐화`는 객체지향의 특징인데,
    
    instanceof를 사용하면 외부에서도 해당 객체의 정보를 알 수 있어 캡슐화가 보장되지 않는다.
    

1. postOn 메서드에서의 행위는 사실 각각의 구현체인 Facebook, Twitter에서 정의하는 것이 옳아보인다.  
    
    이로인해 본인 외의 여러개의 구현클래스의 책임까지 가져야 하기 때문에,  
    
    객체지향원칙(SOLID) 의 `S` 인 `Single Responsibility Principle` 을 위배한다.
    
    💡 ***Single Responseibility Principle : 클래스는 하나의 기능 , 하나의 책임만을 가져야 한다.***
    
    

1. 구현체가 늘어날때마다 일일이 instanceof로 분기를 해주어야 하기 때문에, 
    
    객체 확장 시 항상 변화가 필요하다. 이것은 객체지향원칙의 `O` 인 `Open-Closed Principle`을 위배한다.
    
    💡 ***Open-Closed Principle : 객체의 확장에는 열려있고, 변화에는 닫혀있어야 한다.***
    
    

1. instanceof는 컴파일 시점에 모든 구현체를 확인해서 비교하기 때문에 
    
    다형성을 사용하는 것 보다 성능이 떨어진다.
    

1. 사실 개발에 있어 가장 큰 문제점은, 위 `Dispatch` 클래스의 경우 
    
    새로운 구현 클래스가 추가 되더라도, 컴파일 에러가 발생하지 않기 때문에,
    
    실수로 인한 `Logical Error` 가 발생할 가능성이 매우 높다.
    

---

## 구현 클래스를 파라미터로 받기

```java

public class Dispatch {

    interface Post {

        void postOn(Facebook facebook);

        void postOn(Twitter twitter);

    }

    static class Text implements Post {

        @Override
        public void postOn(Facebook facebook) {
            System.out.println("text - facebook");
        }

        @Override
        public void postOn(Twitter twitter) {
            System.out.println("text - twitter");
        }

    }

    static class Picture implements Post {

        @Override
        public void postOn(Facebook facebook) {
            System.out.println("text - facebook");
        }

        @Override
        public void postOn(Twitter twitter) {
            System.out.println("text - twitter");
        }

    }

    interface SNS {}

    static class Facebook implements SNS {}

    static class Twitter implements SNS {}

    public static void main(String[] args) {
        List<Post> posts = Arrays.asList(new Text(), new Picture());
        List<SNS> sns = Arrays.asList(new Facebook(), new Twitter());

        posts.forEach(p -> sns.forEach(p::postOn)); // 컴파일 에러 발생
    }

}
```

Post 인터페이스에 메서드를 정의할 때 파라미터로 구현 클래스를 직접 받는다.

이렇게 한다면 새로운 구현 클래스가 추가되더라도,

메서드를 호출 할 때 해당 구현 클래스를 파라미터로 받는 메서드가 없기 때문에,

개발자가 누락없이 메서드를 추가할 수 있고, 구현 클래스에서는 컴파일 에러가 발생하기 때문에

쉽게 에러를 막을 수 있다.

---

### 구현 클래스를 파라미터로 받기의 문제점

이제는 컴파일시에 에러를 알아차릴 수 있고, 무분별하게 늘어나는 if문이 줄어들지만, 

여전히 구현 클래스가 늘어날 때 마다, 메서드를 추가해줘야 한다.

또한 메인 메서드 마지막 코드를 보면 컴파일 에러 발생이라고 작성했는데,

맨 처음에 공부한듯이 오버로딩은 Static Dispatch를 사용한다.

그렇기 때문에 인터페이스인 SNS 타입을 파라미터로 넘긴다면, 

어떤 메서드를 실행시켜야 할지 모르기 때문에 컴파일 에러가 발생하게 된다.

💡 [***A Simple Technique for Handling Multiple Polymorphism***](https://algoritmos-iii.github.io/assets/bibliografia/simple-technique-for-handling-multiple-polymorphism.pdf)

_다형성이 다수 발생할 때 어떻게 해결할지에 대한 논문._
_Daniel H. H. Ingalls_



---

## Double Dispatch 사용

```java
public class Dispatch {

    interface Post {
        void postOn(SNS sns);
    }

    static class Text implements Post {

        @Override
        public void postOn(SNS sns) {
            sns.post(this);
        }

    }

    static class Picture implements Post {

        @Override
        public void postOn(SNS sns) {
            sns.post(this);
        }

    }

    interface SNS {

        void post(Text text);

        void post(Picture picture);

    }

    static class Facebook implements SNS {

        @Override
        public void post(Text text) {
            System.out.println("text - facebook");
        }

        @Override
        public void post(Picture picture) {
            System.out.println("picture - facebook");
        }

    }

    static class Twitter implements SNS {

        @Override
        public void post(Text text) {
            System.out.println("text - twitter");
        }

        @Override
        public void post(Picture picture) {
            System.out.println("picture - twitter");
        }

    }

    public static void main(String[] args) {
        List<Post> posts = Arrays.asList(new Text(), new Picture());
        List<SNS> sns = Arrays.asList(new Facebook(), new Twitter());

        posts.forEach(p -> sns.forEach(p::postOn));
    }

}
```

이번에는 Dynamic Dispatch가 발생하는 postOn 메서드에서 SNS 인터페이스를 파라미터로 받고,

SNS에서 Dynamic Dispatch가 발생하고자 하는 Text와 Picture를 파라미터로 받는 메서드를 만들었다.

여기서 postOn 메서드는 SNS의 post 메서드를 호출하면서 자기 자신인 this를 인자로 넘긴다.

중요한 것은 postOn 메서드에서 구현 클래스가 아닌,

인터페이스 SNS를 파라미터로 받아서 사용하기 때문에,

post 메서드를 호출하면서 SNS의 구현클래스가 누구인지에 대한 Dynamic Dispatch가 발생하게 된다.

이렇게 두 번의 Dynamic Dispatch가 발생하는 것을 `Double Dispatch`라고 한다.

 Double Dispatch를 사용한다면, 

아래와 같이 만약 구현 클래스가 추가적으로 생성되더라도, Post, SNS의 변화가 없으며,

신규 구현 클래스의 비즈니스 로직 구현과 클라이언트의 호출 부분만 수정되게 된다.

훨씬 더 객체지향적인 코드로 변경된 것을 알 수 있다.

```java
public class Dispatch {

    interface Post {
        void postOn(SNS sns);
    }

    static class Text implements Post {

        @Override
        public void postOn(SNS sns) {
            sns.post(this);
        }

    }

    static class Picture implements Post {

        @Override
        public void postOn(SNS sns) {
            sns.post(this);
        }

    }

    interface SNS {

        void post(Text text);

        void post(Picture picture);

    }

    static class Facebook implements SNS {

        @Override
        public void post(Text text) {
            System.out.println("text - facebook");
        }

        @Override
        public void post(Picture picture) {
            System.out.println("picture - facebook");
        }

    }

    static class Twitter implements SNS {

        @Override
        public void post(Text text) {
            System.out.println("text - twitter");
        }

        @Override
        public void post(Picture picture) {
            System.out.println("picture - twitter");
        }

    }

    static class GooglePlus implements SNS {

        @Override
        public void post(Text text) {
            System.out.println("text - google plus");
        }

        @Override
        public void post(Picture picture) {
            System.out.println("picture - google plus");
        }

    }

    public static void main(String[] args) {
        List<Post> posts = Arrays.asList(new Text(), new Picture());
        List<SNS> sns = Arrays.asList(new Facebook(), new Twitter(), new GooglePlus());

        posts.forEach(p -> sns.forEach(p::postOn));
    }

}
```

---

### Double Dispatch의 단점

만약 Post의 구현클래스가 추가 된다면, SNS에 새로운 메서드를 만들어줘야 한다.

하지만 이런 경우라면, SNS를 인터페이스가 아닌 `추상 클래스`로 만들어, 

추가되는 구현클래스에 대한 내용은 일반 메서드로 정의해 단점을 극복 할 수 있다.

---

## Visitor Pattern

Visitor Pattern은 기존 코드를 수정하지 않고 새로운 기능을 추가하는 방법을 제안하는 패턴이다.

Visitor Pattern을 사용하면 Double Dispatch를 사용할 수 있으며,

Double Dispatch는 Visitor Pattern의 더 근본적인 형태라고 볼 수 있다.

---

## 출처

[토비의 봄 TV 1회 - 재사용성과 다이나믹 디스패치, 더블 디스패치 - YouTube](https://www.notion.so/TV-1-YouTube-2ae9c18e0a63439185151a894a0d458c)

[백기선 - 코딩으로 학습하는 GoF의 디자인 패턴](https://www.notion.so/GoF-4348d878d03d4d67a7b31d413b5ad928)