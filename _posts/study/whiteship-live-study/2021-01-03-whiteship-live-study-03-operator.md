---
title : "WhiteShip Live Study 3주차. 연산자"
categories :
- whiteship-live-study
tag :
- whiteship-live-study
toc : true
---

## WhiteShip Live Study 3주차. 연산자

---

## 목표
자바가 제공하는 다양한 연산자를 학습하세요.

## 학습할 것
- 산술 연산자
- 비트 연산자
- 관계 연산자
- 논리 연산자
- instanceof
- assignment(=) operator
- 화살표(->) 연산자
- 3항 연산자
- 연산자 우선 순위
- (optional) Java 13. switch 연산자

---

### 연산자란?

연산자를 보기 전 연산이란 수나 식을 일정한 규칙에 따라 계산하는 것이며,  
연산을 수행하는 기호를 연산자라고 한다.  

---

### 산술 연산자
산술 연산은 사칙 연산, 나머지 연산을 말한다.

```java
public class Main {
   public static void main(String[] args) {
      int number1 = 10;
      int number2 = 6;
      
      System.out.println("number1 + number2 = " + (number1 + number2));
      System.out.println("number1 - number2 = " + (number1 - number2));
      System.out.println("number1 * number2 = " + (number1 * number2));
      System.out.println("number1 / number2 = " + (number1 / number2));     // 나눗셈의 몫
      System.out.println("number1 % number2 = " + (number1 % number2));     // 나눗셈의 나머지
   }
}

// 결과
number1 + number2 = 16
number1 - number2 = 4
number1 * number2 = 60
number1 / number2 = 1
number1 % number2 = 4
```

---

### 비트 연산자

비트연산자는 데이터를 비트 단위로 연산한다.  
기능에 따라 비트 이동 연산자와, 비트 논리 연산자로 나뉘어진다.  

#### 비트 이동 연산자 (<< , >>, , <<<, >>>)

|               | 설명                                                                        |
| :-----------: | -------------------------------------------------------------------------- |
| x << y        | 정수 x의 각 비트를 y만큼 왼쪽으로 이동시킨다. (빈자리는 0)                             |
| x >> y        | 정수 x의 각 비트를 y만큼 오른쪽으로 이동시킨다. (빈자리는 정수 x의 최상위 부호 비트와 같은 값  |
| x <<< y       | 정수 x의 각 비트를 y만큼 왼쪽으로 이동시킨다. (빈자리는 0)                             |
| x >>> y       | 정수 x의 각 비트를 y만큼 오른쪽으로 이동시킨다. (빈자리는 0)                           |

무슨말인지 차근차근 확인해보자.  

#### 10진수 - 2진수

우선 10진수와 2진수를 알아야한다.  
10011001 이란 숫자가 10진수로 무엇일까?  
다음은 간단하게 계산하는 방법이다.  

![error](/assets/images/whiteship-live-study/2021-01-03-operator/binaryTodecimal.png)  

아래의 빨간색 수를 1이면 더하고, 0이면 더하지 않는다.  
즉 128 + 16 + 8 + 1 을 하면 10011001의 10진수 숫자인 153이 나온다.  

![error](/assets/images/whiteship-live-study/2021-01-03-operator/decimalTobinary.png)  

반대로 10진수를 2진수를 변경하기 위해선 2로 나누고 각각의 나머지를 아래에서부터 연결하면 2진수가 된다.  

**10 << 2 이라면?**    
우선 10을 2진수로 변경을 하고 2비트 왼쪽으로 이동시킨다.  
```text
00000000 00000000 00000000 00001010 에서         (10)  
00000000 00000000 00000000 00101000 으로 변한다.  (40)  
```

**10 >> 2 이라면?**  

```text
00000000 00000000 00000000 00001010 에서         (10)  
00000000 00000000 00000000 00000010 으로 변한다.   (2)  
```

빈자리는 정수 x의 최상위 부호 비트와 같은 값 이라는 의미를 알아보자.  
첫번째 자리 숫자는 0은 양수를, 1은 음수를 표현한다.  

```text
00000000 00000000 00000000 00000111 이런 수를 2비트 이동 시킨다면,  
00000000 00000000 00000000 00000001 으로 변한다. 
 
11111111 11111111 11111111 11111000 이런 수를 2비트 이동 시킨다면,  
11111111 11111111 11111111 11111110 으로 변한다.
```  
오른쪽으로 밀기 때문에 맨 앞자리는 이동할 비트 수 만큼 사라지게 되는데,  
이때 사라진 값들은 맨 최상위 부호 비트를 따라가면 된다.  

**10 <<< 2, 10 >>> 2**  
10 << 2, 10 >> 2와 같지만 빈자리는 항상 0이 된다.  

간단한 방식이지만 숫자가 많이 나오고 쓸일이 많지 않아 항상 헷갈린다..  
왼쪽에서 화살표가 바라보는 방향으로 오른쪽 비트 수 만큼 이동하면 된다.  

---

#### 비트 논리 연산자 (&, |, ^, ~)

|        | 설명                                               |
| :----: | ------------------------------------------------- |
| &      | 두 비트가 모두 1인 경우에만 결과 1 (AND)                 |
| &#124; | 두 비트중 하나만 1인 경우에만 결과 1 (OR)                 |
| ^      | 두 비트가 하나는 1이고, 하나는 0인 경우에만 결과 1 (XOR)    |
| ~      | 비트 반전 (NOT)                                     |

예시를 살펴보자.  

```
10101
  &
10001
  =
10001
```

```
10101
  |
10001
  =
10101
```

```
10101
  ^
10001
  =
00100
```

```
10101
  ~
01010
```

---

### 관계 연산자

관계 연산자는 수학에서 나오는 **부호**를 생각하면 된다.  
관계 연산자의 결과는 boolean이다.  (true 혹은 false)  

|        | 설명                                               |
| :----: | ------------------------------------------------ |
| <      | 왼쪽 항의 값이 작으면 true, 아니면 false                |
| \>      | 왼쪽 항의 값이 크면 true, 아니면 false                 |
| <=     | 왼쪽 항의 값이 작거나 같으면 true, 아니면 false          |
| >=     | 왼쪽 항의 값이 크거나 같으면 true, 아니면 false          |
| ==     | 두개의 항의 값이 같으면 true, 아니면 false              |
| !=     | 두개의 항의 값이 다르면 true, 아니면 false              |

---

### 논리 연산자

논리 연산자도 마찬가지로 반환값이 boolean이다.  

|              | 설명                                               |
| :----------: | ------------------------------------------------ |
| &&           | 두 항이 모두 true면 true, 아니면 false                |
| &#124;&#124; | 두 항 중 하나의 항만 true면 true, 아니면 false         |
| !            | true는 false로, false는 true로 결과 반전             |

---

### instanceof

instanceof는 왼쪽 항의 참조변수 값이 오른쪽 참조변수의 값이면 true, 아니면 false를 반환한다.  

---

### assignment(=) operator

대입연산자라고 부른다.  
기호는 ' = ' 을 사용하는데 일반적으로 수학에서 말하는 equals와는 다르다.  
만약 int value = 10; 이라는 코드가 있다면,  
int 타입의 value라는 변수에 10을 **대입한다.** 라는 뜻이다.  

```
int a = 10;
int b = 5;
int c = 13;

a = b+c
```

수학적으로 생각하면 이상해 보이지만 프로그래밍에서는 당연하게 a에 b+c가 대입되어 18이 될 것 이다.  

---

### 화살표(->) 연산자

화살표 연산자는 한개의 추상메서드만을 가진  
함수형 인터페이스를 보다 짧고, 보기 좋게 만들 수 있다.      

Runnable 인터페이스로 예시를 들어보겠다.  

```java
Runnable r = new Runnable() {
    @Override
    public void run() {
        System.out.print("일반적인 방법");
    }
};
```

Runnable 인터페이스를 구현한다면 위 처럼 할 수 있지만,  
아래와 같이 화살표 연산자를 사용해서 더욱 코드를 줄일 수 있다.  

```java
Runnable runnable = () -> System.out.println("화살표 연산자를 쓰는 경우");
```

---

### 3항 연산자

if else로 표현할 것을 3항 연산자를 사용함으로써 깔끔하게 코드를 작성할 수 있다.  

```java
int max(int value1, int value2) {
    return value1 > value2 ? value1 : value2;
}
```

보다시피 굉장히 단순하다.  
?를 기준으로 왼쪽은 조건, 오른쪽은 결과이다.  
만약 조건이 true라면 value1을, false라면 value2를 반환한다.  

---

### 연산자 우선 순위

|                    | 설명                                               |
| :----------------: | ------------------------------------------------ |
| -x + 3             | 단항 연산자가 이항 연산자보다 우선순위가 높다. x의 부호를 바꾼 뒤 덧셈을 수행한다.         |
| x + 3 * y          | 덧셈 뺄셈보다 곱셈 나눗셈의 우선순위가 높다. x + (3 * y) 가 된다.                   |
| x + 3 > y - 2      | 비교 연산자보다 산술 연산자가 먼저 수행된다. (x + 3) > (y - 2) 가 된다.             |
| x > 3 && x < 5     | 논리 연산자보다 비교 연산자가 먼저 수행된다. (x > 3) && (x < 5) 가 된다.            |
| result = x + y * 3 | 대입 연산자는 우선순위가 가장 낮다. x + (y * 3) 가 result에 대입된다.              |

---

### Java 13. switch 연산자

기존 switch문이 case와 break로 인해 장황하고,  
break를 빼먹고 작성하면 다음 분기로 넘어가며, 리턴값이 없다는 단점 때문에 생긴 기능이다.  

JDK12 에서 switch문에서 화살표 연산자를 사용할 수 있게 되었고,  
JDK13 에서 yield 예약어를 사용할 수 있게 되었다.  

```java
// 기존 switch문
String result = "java";

switch(result){
    case "java":
        System.out.println("Hello Java");
        break;
    case "C++":
        System.out.println("Hello C++");
        break;
    default:
        System.out.println("etc");
}

System.out.println("result = " + result);

// result = Hello Java
```

```java
// JDK 12 Switch Arrow
public static void main(String[]args){
    String result = switchArraw("java");
    System.out.println(result);
}

public static String switchArrow(String str) {
    return switch(result){
        case "java" -> "Hello Java";
        case "C++" -> "Hello C++";
        default -> "etc";
    };
}

// Hello Java
```

```java
// JDK 13 Switch yield
public static void main(String[]args){
    String result = switchYield("java");
    System.out.println(result);
}
 public static String switchYield(String str) {
    String resultStr switch (result) {
        case "java":
            yield "Hello Java";
        case "C++":
            yield "Hello C++";
        default:
            yield "etc";
    };
    
    return resultStr;
}

// Hello Java
```

JDK13 부터는 yield를 사용함으로써 변수에 값을 대입할 수 있다.  
기존의 switch문 처럼 break를 사용할 필요가 없어졌기 때문에  
실수가 줄어들고, 코드가 줄어들어 가독성이 좋아졌다.    

JDK12 부터 리턴을 할 수 있어졌으니 조금 더 다양하게 활용할 수 있을 것 같다.  

