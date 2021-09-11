---
title : "스프링 의존관계. DI, DIP"
categories : 
    - java
tag :
    - spring
toc : true
---

[토비의 스프링 3.1](http://www.yes24.com/Product/Goods/7516911)를 공부한 내용입니다.    

---

#### **의존관계**  
![error](/assets/images/study/toby-spring-3.1/01-spring-di.png)   

위 그림의 점선은 A 클래스가 B 클래스에 의존하는 것을 의미한다.  
이때 B 클래스의 기능이 추가/변경 되었을 때 A 클래스에도 영향이 생긴다.  

---

#### **의존관계 예제**  
과일 가게를 운영한다고 가정하자.  

```java
public class Apple {

    private int price = 1000;

    public int getPrice() {
        return price;
    }

}

public class Store {

    private Apple apple;

    public Store() {
        this.apple = new Apple();
    }

    public void priceDisplay() {
        System.out.println("사과의 가격은 " + this.apple.getPrice() + "원 입니다.");
    }

}
```

이 가게는 사과만을 판매하고 있는데, 어느날은 사과가 다 떨어져  
오렌지 판매를 하게 되었다.  

```java
public class Orange {

    private int price = 2000;

    public int getPrice() {
        return price;
    }

}

public class Store {
    
    private Orange orange;

    public Store() {
        this.orange = new Orange();
    }

    public void priceDisplay() {
        System.out.println("오렌지의 가격은 " + this.orange.getPrice() + "원 입니다.");
    }

}
```

이 가게는 사과만을 판매하고 있었기 때문에 급하게 오렌지 판매 준비를 해야한다.  

사과에 대한 의존도가 높아 판매 품종이 바뀌면 가게도 그에 맞게 변경해야 되는 것이다.  

![error](/assets/images/study/toby-spring-3.1/02-spring-di.png)   

이렇게 수정해보자.  

![error](/assets/images/study/toby-spring-3.1/03-spring-di.png)   


```java
public interface Fruit {

    void priceDisplay();

}

public class Apple implements Fruit {

    private int price = 1000;

    @Override
    public void priceDisplay() {
        System.out.println("사과의 가격은 " + this.price + "원 입니다.");
    }

}

public class Orange implements Fruit {

    private int price = 2000;

    @Override
    public void priceDisplay() {
        System.out.println("오렌지의 가격은 " + this.price + "원 입니다.");
    }
}
```

과일 인터페이스를 만들고, 사과, 오렌지 클래스가 의존하도록 만들었다.  

가게에서는 과일을 의존하면 어떻게 변할까?  

```java
public class Store {

    private Fruit fruit;

    public void setFruit(Fruit fruit) {
        this.fruit = fruit;
    }

    void priceDisplay() {
        this.fruit.priceDisplay();
    }

}
```

너무나도 단순하고 간단해진다.  
고객의 요청에 따라 동적으로 과일의 정보를 알려줄 수 있게 되었다.  

```java
public class Customer {

    public static void main(String[] args) {
        Store store = new Store();

        store.setFruit(new Apple());
        store.priceDisplay();
        
        store.setFruit(new Orange());
        store.priceDisplay();
    }

}
```

이로써 런타임 시점에 의존관계가 드러나지 않으며,  
제 3의 존재에 의해 (위 예제의 경우 클라이언트) 관계를 결정하게 되었다.

이는 SOLID 원칙 중 DIP (Dependency Injection Principle)  
의존성 역전 원칙에 해당한다.  

의존관계를 맺을 때 변하기 쉬운 것에 (클래스 - 사과) 의존하는 것보다  
변하기 쉽지 않는 것에 (인터페이스 - 과일) 의존하는 것 이다.  

---

스프링 DI의 대한 내용 추가 필요함.  

---