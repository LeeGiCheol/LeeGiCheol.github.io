---
title : "WhiteShip Live Study 11주차. Enum"
categories :
- whiteship-live-study
tag :
- whiteship-live-study
toc : true
---

## WhiteShip Live Study 11주차. Enum

---

## 목표
자바의 열거형에 대해 학습하세요.

## 학습할 것 (필수)
- Enum 정의하는 방법
- Enum이 제공하는 메소드 (values()와 valueOf())
- java.lang.Enum
- EnumSet

---

### Enum이란

Enum은 JDK 1.5 버전부터 추가된 기능이다.  
**열거형**이라고 불리고, **서로 연관된 상수들의 집합**이다.  

Enum을 사용함으로써 얻는 이점은 다음과 같다.
- Enum 키워드를 사용함으로써 열거임을 분명하게 나타낸다.  
- 코드가 단순해지며, 가독성이 좋아진다.  
- 값을 비교하는 경우 비교 전에 타입을 먼저 비교하므로 값이 같더라도 타입이 다르면 컴파일 에러가 발생한다.  

---

### Enum 정의하는 방법

일반적으로 상수를 선언할 때 final 키워드를 사용한다.  
그러나 상수가 많아지면 코드가 불필요하게 길어지게 된다.  

```java
public class Card {

    static final int CLOVER = 0;
    static final int HEART = 1;
    static final int DIAMOND = 2;
    static final int SPACE = 3;

    static final int TWO = 0;
    static final int THREE = 1;
    static final int FOUR = 2;

}
```

이때 Enum을 사용함으로써 상수를 간단하게 선언할 수 있다.  

```java
public class Card {

    enum Kind { CLOVER, HEART, DIAMOND, SPADE }
    enum Value { TWO, THREE, FOUR }

}
```

위에서 장점 이야기를 했는데 그 중  
**값을 비교하는 경우 비교 전에 타입을 먼저 비교하므로 값이 같더라도 타입이 다르면 컴파일 에러가 발생한다.** 를 살펴보자

![error](/assets/images/whiteship-live-study/2021-01-24/type.png)

Card.Kind.CLOVER의 값은 0이고, Card.Value.TWO의 값도 0 이다.  
조건식 자체는 true가 되겠지만 둘은 타입이 다르기 때문에 false가 되고 컴파일 에러가 발생한다.  

enum 키워드는 클래스에도 붙일 수 있다.  

```java
public enum Card {
    CLOVER, 
    HEART, 
    DIAMOND, 
    SPADE;
}
```

---

### Enum이 제공하는 메소드


| 이름                                           | 설명                                                                   |
| --------------------------------------------- | --------------------------------------------------------------------- |
| name()                                        | 열거형 상수의 이름을 문자열로 리턴한다.                                        |
| ordinal()                                     | 열거형 상수의 정의된 순서를 리턴한다. (0부터 시작함)                             |
| compareTo(E o)                                | 열거형 상수의 순서를 비교해서 순번차이를 리턴한다.                                |
| values()                                      | 모든 열거형 상수를 배열로 리턴한다.                                           |
| valueOf(Class<T> enumType, String name)       | 열거형의 name과 일치하는 열거형 상수를 리턴한다.                                                 |

<br>

다음은 아래의 enum을 기준으로한 예시이다.  

```java
public enum Card {
    CLOVER,
    HEART,
    DIAMOND,
    SPADE;
}
```

### name() 


```java
public static void main(String[] args) {
    Card card = Card.CLOVER;
    System.out.println(card.name());
}

// CLOVER
```

### ordinal()
```java
public static void main(String[]args){
    Card card = Card.SPACE;
    System.out.println(card.ordinal());
}

// 3
```

### compareTo(E o)
```java
public static void main(String[]args){
    System.out.println(Card.DIAMOND.compareTo(Card.HEART));
    System.out.println(Card.HEART.compareTo(Card.DIAMOND));
}

/*
     1
    -1
 */
```

### values()
```java
public static void main(String[]args){
    Card[] card = Card.values();

    for (Card kind: card) {
    System.out.println(kind);
    }
}

/*
    CLOVER
    HEART
    DIAMOND
    SPADE     
 */
```

### valuesOf(Class<T> enumType, String name)
```java
public static void main(String[]args){
    System.out.println(Card.valueOf("CLOVER"));
    System.out.println(Enum.valueOf(Card.class, "HEART"));
}

/* 
    CLOVER
    HEART
*/
```

---

### java.lang.Enum

클래스는 기본적으로 Object 클래스를 상속받듯이  
enum은 Enum 클래스를 상속받는다.  

![error](/assets/images/whiteship-live-study/2021-01-24/extendsEnum1.png)  

![error](/assets/images/whiteship-live-study/2021-01-24/extendsEnum2.png)  

아래는 바이트 코드를 보면 상속받은 것을 더욱 정확하게 알 수 있다.

```java
public final enum com/example/demo/Card extends java/lang/Enum {

  // compiled from: Card.java

  // access flags 0x4019
  public final static enum Lcom/example/demo/Card; CLOVER

  // access flags 0x4019
  public final static enum Lcom/example/demo/Card; HEART

  // access flags 0x4019
  public final static enum Lcom/example/demo/Card; DIAMOND

  // access flags 0x4019
  public final static enum Lcom/example/demo/Card; SPADE

  // access flags 0x101A
  private final static synthetic [Lcom/example/demo/Card; $VALUES

  // access flags 0x9
  public static values()[Lcom/example/demo/Card;
   L0
    LINENUMBER 3 L0
    GETSTATIC com/example/demo/Card.$VALUES : [Lcom/example/demo/Card;
    INVOKEVIRTUAL [Lcom/example/demo/Card;.clone ()Ljava/lang/Object;
    CHECKCAST [Lcom/example/demo/Card;
    ARETURN
    MAXSTACK = 1
    MAXLOCALS = 0

  // access flags 0x9
  public static valueOf(Ljava/lang/String;)Lcom/example/demo/Card;
    // parameter mandated  name
   L0
    LINENUMBER 3 L0
    LDC Lcom/example/demo/Card;.class
    ALOAD 0
    INVOKESTATIC java/lang/Enum.valueOf (Ljava/lang/Class;Ljava/lang/String;)Ljava/lang/Enum;
    CHECKCAST com/example/demo/Card
    ARETURN
   L1
    LOCALVARIABLE name Ljava/lang/String; L0 L1 0
    MAXSTACK = 2
    MAXLOCALS = 1

  // access flags 0x2
  // signature ()V
  // declaration: void <init>()
  private <init>(Ljava/lang/String;I)V
    // parameter synthetic  $enum$name
    // parameter synthetic  $enum$ordinal
   L0
    LINENUMBER 3 L0
    ALOAD 0
    ALOAD 1
    ILOAD 2
    INVOKESPECIAL java/lang/Enum.<init> (Ljava/lang/String;I)V
    RETURN
   L1
    LOCALVARIABLE this Lcom/example/demo/Card; L0 L1 0
    MAXSTACK = 3
    MAXLOCALS = 3

  // access flags 0x8
  static <clinit>()V
   L0
    LINENUMBER 6 L0
    NEW com/example/demo/Card
    DUP
    LDC "CLOVER"
    ICONST_0
    INVOKESPECIAL com/example/demo/Card.<init> (Ljava/lang/String;I)V
    PUTSTATIC com/example/demo/Card.CLOVER : Lcom/example/demo/Card;
   L1
    LINENUMBER 7 L1
    NEW com/example/demo/Card
    DUP
    LDC "HEART"
    ICONST_1
    INVOKESPECIAL com/example/demo/Card.<init> (Ljava/lang/String;I)V
    PUTSTATIC com/example/demo/Card.HEART : Lcom/example/demo/Card;
   L2
    LINENUMBER 8 L2
    NEW com/example/demo/Card
    DUP
    LDC "DIAMOND"
    ICONST_2
    INVOKESPECIAL com/example/demo/Card.<init> (Ljava/lang/String;I)V
    PUTSTATIC com/example/demo/Card.DIAMOND : Lcom/example/demo/Card;
   L3
    LINENUMBER 9 L3
    NEW com/example/demo/Card
    DUP
    LDC "SPADE"
    ICONST_3
    INVOKESPECIAL com/example/demo/Card.<init> (Ljava/lang/String;I)V
    PUTSTATIC com/example/demo/Card.SPADE : Lcom/example/demo/Card;
   L4
    LINENUMBER 3 L4
    ICONST_4
    ANEWARRAY com/example/demo/Card
    DUP
    ICONST_0
    GETSTATIC com/example/demo/Card.CLOVER : Lcom/example/demo/Card;
    AASTORE
    DUP
    ICONST_1
    GETSTATIC com/example/demo/Card.HEART : Lcom/example/demo/Card;
    AASTORE
    DUP
    ICONST_2
    GETSTATIC com/example/demo/Card.DIAMOND : Lcom/example/demo/Card;
    AASTORE
    DUP
    ICONST_3
    GETSTATIC com/example/demo/Card.SPADE : Lcom/example/demo/Card;
    AASTORE
    PUTSTATIC com/example/demo/Card.$VALUES : [Lcom/example/demo/Card;
    RETURN
    MAXSTACK = 4
    MAXLOCALS = 0
}
```

---

### EnumSet

EnumSet은 Set 인터페이스의 구현체로 Set 형태로 사용할 수 있다.    


```java
public static void main(String[] args) {
    EnumSet<Card> cardEnumSet = EnumSet.allOf(Card.class);

    Iterator<Card> iterator = cardEnumSet.iterator();

    while (iterator.hasNext()) {
        System.out.println(iterator.next());
    }
}
```
