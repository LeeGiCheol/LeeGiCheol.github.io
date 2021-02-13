---
title : "WhiteShip Live Study 2주차. 자바 데이터 타입, 변수 그리고 배열"
categories :
- whiteship-live-study
tag :
- whiteship-live-study
toc : true
---

## WhiteShip Live Study 2주차. 자바 데이터 타입, 변수 그리고 배열

---

## 목표
자바의 프리미티브 타입, 변수 그리고 배열을 사용하는 방법을 익힙니다.

## 학습할 것
- 프리미티브 타입 종류와 값의 범위 그리고 기본 값
- 프리미티브 타입과 레퍼런스 타입
- 리터럴
- 변수 선언 및 초기화하는 방법
- 변수의 스코프와 라이프타임
- 타입 변환, 캐스팅 그리고 타입 프로모션
- 1차 및 2차 배열 선언하기
- 타입 추론, var

---

### 프리미티브 타입 종류와 값의 범위 그리고 기본 값

프리미티브 타입은 한글로 기본형이라고 한다.  
총 8개의 타입이 있으며, 논리형, 문자형, 정수형, 실수형으로 구분된다.  

|        | 1 byte  | 2 byte | 4 byte   | 8 byte      |
| :---:  | :-----: | :----: | :------: | :---------: |
| 논리형   | boolean |       |           |             |
| 문자형   |         | char  |           |             |
| 정수형   | byte    | short | **int**   | long        |
| 실수형   |         |       | float     | **double**  |

정수형 중엔 int가 기본이며, 실수형 중엔 double이 기본이다.  


| 자료형    | 기본 값  | 범위                                                     | bit | byte |
| :----:  | :-----: | ------------------------------------------------------- | --- | ---- |
| boolean | false   | false, true                                             | 8   | 1    |
| char    | \u0000  | '\u0000' ~ '\uffff'                                     | 16  | 2    |
| byte    | 0       | -128 ~ 127                                              | 8   | 1    |
| short   | 0       |  -32,768 ~ 32,767                                        | 16  | 2    |
| int     | 0       | -2,147,483,648 ~ 2,147,483,647                          | 32  | 4    |
| long    | 0L      | -9,223,372,036,854,775,808 ~ 9,223,372,036,854,775,807  | 64  | 8    |
| float   | 0.0F    | 1.4E-45 ~ 3.4E38                                        | 32  | 4    |
| double  | 0.0     | 4.0E=324 ~ 1.8E3-9                                      | 64  | 8    |

float와 double는 양의 범위만 적은 것이다.  
다 외우기엔 너무 많고, 기억하기 쉽지 않다.  
정수형의 경우 -2^n-1 ~ 2^n-1 -1 (n은 bit수)를 기억하면 된다.  
예를들어 int는 32비트이니 -2^31 ~ 2^31 -1 의 범위를 가진다.  

---

### 프리미티브 타입과 레퍼런스 타입

자료형은 크게 프리미티브 타입과 (기본형) 레퍼런스 타입(참조형)으로 나뉜다.  
프리미티브 타입은 실제 값을 저장하는 반면,  
레퍼런스 타입은 값이 저장되어있는 주소를 값으로 가진다.  
자바는 C언어와는 달리 참조형 변수 간의 연산을 할 수 없으므로  
실제 연산에 사용되는 것은 모두 기본형 변수이다.  

---

### 리터럴

리터럴을 공부하기 전 상수를 먼저 알아보자.  
상수란 **변하지 않는 값**이다.  
그러나 프로그래밍에서는 **변하지 않는 저장공간**이라는 의미이다.  

만약 final int MAX_VALUE = 100 이러한 상수가 있다.  
원래 100 이 상수가 되어야 하지만  
프로그래밍에서는 MAX_INT가 상수이다.  

그렇다면 리터럴은 무엇일까?  
바로 100이 리터럴이다.  

단지 상수의 의미를 저장공간으로 정의했기 때문에  
일반적으로 생각하는 상수를 구분하기 위해 리터럴이라고 부른다.  
(대입 연산자 기준 우측의 항을 리터럴이라고 부른다고 생각하면 된다.)  

타입별로 접미사가 필요한 경우도 있다.  
long 타입은 "l"또는 "L"을 붙이고 float 타입은 "f" 또는 "F"를, double 타입은 "d" 또는 "D"를 붙인다.  
단 double은 실수형의 기본형이기 때문에 생략 가능하다.  

정수형 리터럴은 아무래도 큰 숫자는 읽기 힘들다.  
그렇기 때문에 JDK 1.7부터 정수형 리터럴 중간에 구분자를 사용할 수 있게 되었다.  
ex) long bigNumber = 1_000_000_000L;    // long bigNumber = 1000000000L;

---

### 변수 선언 및 초기화하는 방법
```java
public static void main(String[]args){
    // 변수 x를 선언
    int x;       
    
    // 변수 x를 초기화
    x = 10;
    
    // 변수 y를 선언 하면서 초기화
    int y = 100;
}
```

---

### 변수의 스코프와 라이프타임

변수는 클래스 변수, 인스턴스 변수, 지역변수 모두 세 종류가 있다.  
변수의 종류를 결정짓는 중요한 요소는 변수의 선언 위치이다.  

| 변수의 종류   | 선언위치                                             | 생성시기                |
| :--------: | :------------------------------------------------: | --------------------- | 
| 클래스 변수   | 클래스 영역                                           | 클래스가 메모리에 올라갈 때  | 
| 인스턴스 변수 |  클래스 영역                                          | 인스턴스가 생성되었을 때    | 
| 지역변수     | 클래스 영역 이외의 영역 <br> (메서드, 생성자, 초기화 블럭 내부) | 변수 선언문이 수행되었을 때 |

---

### 타입 변환, 캐스팅 그리고 타입 프로모션

자바의 연산은 동일한 타입에서만 가능하다.  
하지만 개발을 하다보면 종종 다른 타입간의 연산이 필요한 경우가 생긴다.  
이러한 경우에 타입 변환을 해줄 수 있다.  

타입 변환은 두 가지 종류이다.  
캐스팅 (강제 형변환), 타입 프로모션 (자동 형변환)

타입 프로모션은 작은 메모리 크기의 데이터 타입을 큰 메모리 타입으로 변환 시키는 것이.  
이 경우 데이터의 손실이나 변형이 생기지 않음으로 자동으로 형변환이 된다.  

```java
public static void main(String[]args){
    byte a = 10;
    int b = a;
    System.out.println(b); // 10
}
```

캐스팅은 더 큰 메모리 데이터 타입을 작은 메모리 타입으로 변환 시키는 것이다.  
타입 프로모션과는 다르게 데이터의 손실이나 변형이 생길 수 있다.  
그렇기 때문에 타입 프로모션처럼 형변환을 하려고 하면 컴파일 에러가 발생한다.  

아래는 바이트의 범위 내에서 형변환을 시도한 것이다. (바이트의 범위 -128 ~ 127)
```java
public static void main(String[]args){
    int a = 10;
    byte b = (byte)a;
    System.out.println(b);  // 10       
}
```

아래는 바이트의 범위 외에서 형변환을 시도한 것이다.
```java
public static void main(String[]args){
    int a = 128;
    byte b = (byte)a;
    System.out.println(b);  // -128       
}
```

바이트의 범위는 127까지인데 128을 변환하려고 했으니 오버플로우가 생겼다.  
오버플로우는 범위를 넘쳤을 때 발생하는데, 뫼비우스의 띠 같은 모양을 생각하면 된다.  
-128, -127 ... , 126, 127, -128, -127 ...   

바이트는 127 다음 수가 없으니 제일 첫번째 수인 -128로 변환된 것 이다.  

---

### 1차 및 2차 배열 선언하기

**배열이란?**  
같은 타입의 여러 변수를 하나의 묶음으로 다루는 것을 배열이라고 한다.  

배열을 선언하기 위해선 [] 기호를 사용하고,    
공간의 크기가 정해져있기 때문에 new 연산자로 배열의 크기를 지정해줘야한다.   

```java
public static void main(String[]args){
    // 배열 선언 
    int[] arr1;
    int arr2[];
    
    // 크기가 10인 배열을 생성
    arr1 = new int[10];
    
    // 선언과 생성을 동시에
    int[] arr3 = new int[5];

    // 선언과 생성과 초기화를 동시에
    int[] arr4 = new int[5] {1, 2, 3, 4, 5};
    
    // arr4의 경우 아래와 같이 new int 생략 가능
    int[] arr5 = {1, 2, 3, 4, 5};
}
```

2차원 배열을 선언하는 방법은 1차원 배열과 동일하다.  
1차원 배열보다 [] 가 하나 더 붙었을 뿐이다.  

```java
public static void main(String[]args){
    // 4행 3열의 2차원 배열 생성
    int[][] arr1 = new int[4][3];
    
    // 생성과 초기화를 동시에
    int[][] arr2 = new int[][]{ {1, 2, 3}, {4, 5, 6} };
    
    // 마찬가지로 new int는 생략 가능하다.
    int[][] arr3 = { {1, 2, 3}, {4, 5, 6} };
    
}
```

---

### 타입 추론, var
타입 추론이란 컴파일러가 값을 보고 어떤 데이터 타입인지 유추하는것을 말한다.  
JDK9 까지는 Generic이나 Lambda만 지원했지만,  
JDK10 부터는 var 라는 키워드가 추가되었다.  

지역변수로만 사용가능하며, 선언과 동시에 초기화를 해야한다.  

![error](/assets/images/whiteship-live-study/2021-02-13-variable/var.png)  

간단하게 테스트해보았다.  
ArrayList에 String 타입의 값을 넣고,  
var를 사용해서 꺼내보았다.  

실행하기 전 인텔리제이가 먼저 String 타입이라는 것을 눈으로 보여줬고,  
실제 동작도 정상적으로 수행했다.

---

# 참고
[남궁성 - 자바의 정석](http://www.yes24.com/Product/Goods/24259565)  

---




