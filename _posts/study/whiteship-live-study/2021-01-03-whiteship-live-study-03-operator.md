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

