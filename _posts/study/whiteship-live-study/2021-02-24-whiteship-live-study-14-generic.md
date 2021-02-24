---
title : "WhiteShip Live Study 14주차. Generic"
categories :
- whiteship-live-study
tag :
- whiteship-live-study
toc : true
---

## WhiteShip Live Study 14주차. Generic

---

## 자바의 제네릭에 대해 학습하세요.

## 학습할 것 (필수)
- 제네릭 사용법
- 제네릭 주요 개념 (바운디드 타입, 와일드 카드)
- 제네릭 메소드 만들기
- Erasure

---

### 제네릭이란?

제네릭은 JDK 1.5부터 추가된 기능이다.  
제네릭은 타입을 Generalize(일반화)한다는 것을 의미하며    
데이터 타입을 컴파일 시에 미리 지정하는 방법이다.  

제네릭을 사용하지 않으면 여러 타입을 사용하는 클래스나 메서드에서는  
보통 Object 타입을 인자값이나 리턴값으로 사용한다.  

그런데 이러한 경우 Object 타입의 객체를 알맞는 타입에 맞게 형변환을 해줘야하는데,  
이때 에러가 생길 가능성이 높다.  

이럴때 제네릭을 사용하고 생기는 장점은 다음과 같다.  
1. 형변환이나 타입체크를 해주지 않아도 된다.  
2. 객체의 Type Safety

---

### 제네릭 사용법

```java
public static void main(String[]args){
    ArrayList<String> genericList = new ArrayList<String>();
    // ArrayList<String> genericList = new ArrayList<>();   // JDK 1.7부터 가능한 방법
}
```

위와 같은 방법으로 Generic을 사용할 수 있다.  
꺽쇠 기호 '< >' 를 사용해 그 안에 사용할 타입을 정의하면 된다.  
위의 경우 String 타입만 사용하는 ArrayList가 되는 것이다.  

생성자 뒤로 오는 꺽쇠의 타입은 JDK 1.7부터 생략할 수 있다.  


![error](/assets/images/whiteship-live-study/2021-02-24/generic-arraylist.png)  

위 코드는 제네릭을 사용한 ArrayList와 사용하지 않은 ArrayList이다.  
제네릭을 사용하지 않은 nonGenericList는 어떤 타입이던 리스트에 담을 수 있다.  
그러나 꺼내는 과정에서 형변환을 필수적으로 해줘야한다.  
타입이 어떤것인지에 대해 확신이 없을 가능성이 높기 때문에 타입 체크도 필요하다.  

그러나 제네릭을 사용한 genericList는 이미 리스트안에 String 밖에 사용하지 못한다.  
int 값을 담으려하자 컴파일 에러가 발생한다.  

값을 꺼낼때도 형변환이나 타입체크가 필요없다.  
int로 값을 꺼내려하자 컴파일 에러가 발생한다.  

이처럼 더 직관적이고, 코드를 간략화 시킬 수 있다.  

또한 nonGenericList는 밑줄이 쳐져있는 걸 보고  
인텔리제이는 제네릭을 사용하는걸 권장한다고 생각했다.  

---

### 제네릭 명명규칙

꼭 지켜야하는것은 아니지만 제네릭은 명명규칙이 있다.  
가독성을 위해 지켜주는 것이 좋다.  

| 타입             | 의미                    |
| :-------------: | ---------------------- |
| T               | Type                   |
| E               | Element                |
| N               | Number                 |
| K               | Key                    |
| V               | Value                  |
| S, U, V ...     | 2nd, 3rd, 4th types    |

---

### 제네릭 주요 개념 (바운디드 타입, 와일드 카드)

#### 바운디드 타입

바운디드 타입이란 제네릭으로 사용될 타입 파라미터의 값을 제한할 수 있는 방법이다.  
만약 정수든 실수든 숫자값이라면 모두 사용하고 싶다면 이렇게 할 수 있다.  

```java
public class GenericClass<T extends Number> { }
```

먼저 Number를 상속받은 타입을 사용하도록 위와 같은 클래스를 만들어주었다.  

![error](/assets/images/whiteship-live-study/2021-02-24/number-extends.png)

GenericClass 객체를 만들면서 위와 같이 Number를 상속받은 타입을 사용하도록 한다.  
만약 Number를 상속받은 객체가 아니라면 컴파일 에러가 발생한다.  

#### 와일드 카드

제네릭을 사용하면 타입이 제한된다.  
ArrayList<String> stringLisat = new ArrayList<>();  
이러한 ArrayList는 String만 받을 수 있는 것 처럼 말이다.  

이때 어떤 타입도 받을 수 있도록 만들기 위해선 와일드 카드를 사용해야 한다.  

와일드 카드는 '?' 기호와 extends, super를 사용해 만들 수 있다.  

```text
<? super T> : 와일드 카드의 상한 제한. T와 그 자식들만 가능
<? extends T> : 와일드 카드의 하한 제한. T와 그 조상들만 가능
<?> : 제한 없이 모든 타입 가능. <? extends Object> 와 동일하다.
```

```java
public class Person {}

public class Gicheol extends Person {}
```

Person을 상속받은 Gicheol 클래스가 있다.  

![error](/assets/images/whiteship-live-study/2021-02-24/super-extends.png)  

?super Person을 사용한 경우 Gicheol을 사용 못하지만,  
?extends Person을 사용한 경우 Gicheol을 사용해서 다형성을 쓸 수 있다.    

---

### 제네릭 메소드 만들기

제네릭 메서드란 메서드 선언부에 제네릭 타입이 선언된 메서드를 말한다.  

```java
public static <T> void sort(List<T> list, Comparator<? super T> c) {
    list.sort(c);
}
```

대표적인 제네릭메서드 중 하나인 Collections 클래스의 sort 메서드이다.  

주의해야 할 점은 메서드의 정의된 타입인 T와 매개변수 T는 이름은 동일하지만,  
전혀 별개의 것이라는 것을 주의해야한다.  

이러한 메서드느 아래와 같이 사용 할 수 있다.  

![error](/assets/images/whiteship-live-study/2021-02-24/collections-sort.png)  

메서드 앞에 타입을 적어주는게 맞지만  
대부분의 경우 컴파일러가 대입된 타입을 추정할 수 있기 때문에 생략할 수 있다.  

---

### Erasure

Erasure는 삭제(소거)라는 의미인데,  
제네릭을 사용하면 그 타입은 컴파일 후 지워지게 된다.  

즉 컴파일시에는 타입의 제약이 있지만,  
런타임시에는 타입의 제약은 사라진다는 의미이다.  

```java
public static void main(String[] args) {
    List<String> erasureTest = new ArrayList<>();
}
```
<br>

![error](/assets/images/whiteship-live-study/2021-02-24/erasure.png)  

보면 erasureTest 객체는 List일뿐 String 이라는 정보는 사라졌다.  
단지 바로 아래 주석으로 남아있을 뿐이다.  

<br>

다음은 add와 get 메서드의 바이트 코드이다.  

```java
public static void main(String[]args){
    List<String> byteCodeTest = new ArrayList<>();
    byteCodeTest.add("byteCode?");
    byteCodeTest.get(0);
}
```

![error](/assets/images/whiteship-live-study/2021-02-24/add-get-erasure.png)  

List에 값을 넣고 가져올 때는 Object로 변환을 시킨다는 것을 알 수 있었다.  

이러한 Type Erasure가 있기 때문에  
Generic이 없는 JDK 1.5 아래 버전에서 Generic을 사용하더라도    
컴파일은 못하겠지만 컴파일된 class파일은 동작한다.  

---