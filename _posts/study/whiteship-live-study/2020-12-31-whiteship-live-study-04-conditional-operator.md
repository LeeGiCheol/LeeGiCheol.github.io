---
title : "WhiteShip Live Study 4주차. 제어문"
categories :
- whiteship-live-study
tag :
- whiteship-live-study
toc : true
---

## WhiteShip Live Study 4주차. 제어문

---

## 목표
자바가 제공하는 제어문을 학습하세요.

## 학습할 것 (필수)
- 선택문
- 반복문

## 과제 (옵션)
- 과제 0. JUnit 5 학습하세요.  
- 과제 1. live-study 대시 보드를 만드는 코드를 작성하세요.
- 과제 2. LinkedList를 구현하세요.
- 과제 3. Stack을 구현하세요.
- 과제 4. 앞서 만든 ListNode를 사용해서 Stack을 구현하세요.
- 과제 5. Queue를 구현하세요.

[4주차 과제](https://github.com/LeeGiCheol/whiteship-live-study/tree/master/4week/src/main/java/com/whiteship/homework)

[4주차 과제 테스트코드](https://github.com/LeeGiCheol/whiteship-live-study/tree/master/4week/src/test/java/com/whiteship/homework)


---

## 제어문
코드의 실행흐름은 위에서 아래로 순차적으로 진행이 된다.  
그러나 때로는 조건에 따라 특정 문장을 반복하거나 건너뛰어야 할 필요성이 있다.  
이렇게 프로그램의 흐름을 바꿔주는 역할을 하는것이 **제어문**이다.

제어문은 조건문과 반복문이 있다.  

---

## 조건문

### if문
if문은 가장 기본적인 조건문이다.  
조건식이 true이면 블럭(괄호) 안의 문장을 수행하라는 의미이다.  
수행할 문장이 한 문장이라면 블럭은 생략할 수 있지만, 가독성이 떨어질 수 있으니 웬만하면 붙이도록 하자.   

```java
if (조건식) {
    System.out.println("if문에 들어왔다!");   // 조건식이 참이면 if문에 들어왔다! 를 출력한다.     
}
```

예를들어 이런식으로 사용할 수 있다.  

```java
if (age >= 20) {
    System.out.println("성인입니다.");    
}
```

여담으로 블럭 위치에 대한 스타일도 존재한다...  

```java
if (true)
{
    System.out.println("BSD");      // BSD 방식의 블럭
}

if (true) {
    System.out.println("K&R");      // K&R 방식의 블럭
}
```

### if-else문
else 즉 조건식의 결과가 false일 때 블럭안의 문장을 수행하라는 의미이다.  

```java
if (조건식) {
    System.out.println("조건식이 true이다!");
} else {
    System.out.println("조건식이 false이다!");
}
```

성인인지 아닌지 판별해야하는 경우 if문을 두번 사용하는 것보다 if-else문을 사용하는 것이 효율적이다.  
if문이 여러개면 항상 여러개의 조건식을 계산해야하지만 if-else문은 한번만 계산하면 되기 때문이다.  

```java
if (age >= 20) {
    System.out.println("성인입니다.");
}
if (age <= 19) {
    System.out.println("성인이 아닙니다.");
}

// 위의 코드를 if-else문으로 수정한다.

if (age >= 20) {
  System.out.println("성인입니다.");
} else {
  System.out.println("성인이 아닙니다.");
}
```

### if-else if문
처리 할 경우의 수가 세가지 이상인 경우에 사용할 수 있다.  

```java
if (score >= 90) {
    grade = "A";
} else if (score >= 80) {
    grade = "B";
} else if (score >= 60) {
    grade = "C";
} else {
    grade = "D"
}
```

첫번째 조건식부터 순서대로 계산을 해서 true인 조건식을 만나면 해당 블럭을 수행하고 if-else if문을 벗어난다.  
만약 모든 조건식이 false라면 else 블럭이 수행된다.  
이때 else 블럭은 필수 조건이 아니며, 모든 조건식이 false일때 else 블럭이 존재하지 않는다면 if-else if문의 어떤 블럭도 실행되지 않는다.  

### 중첩 if문
if문 내부에 if문을 포함시킬 수 있다.

```java
if (score >= 90) {
    if (score >= 95) {
        grade = "A+"
    } else if (score < 95) {
        grade = "A-";    
    } else {
        grade = "A";
    }
} else if (score >= 80) {
    if (score >= 85) {
        grade = "B+"
    } else {
        grade = "B";
    }
} else if (score >= 60) {
  grade = "C";
} else {
  grade = "D"
}
```

---

## 선택문

### switch문
switch문 혹은 switch-case문이라고 부른다.  
if문은 조건식 결과가 true와 false만 있기 때문에  
분기가 많아지면 복잡해지고, 느려진다.  
분기가 많다면 switch문을 사용해서 개선해보도록 하자.  

```java
int value = 1;

switch(value){
  case 1:
    System.out.println("1");
    break;
  case 2:
    System.out.println("2");
    break;
  case 3 :
    System.out.println("3");
    break;
  default :
      System.out.println("그 외의 숫자");
}
```

switch문의 블럭안의 조건식을 계산해 일치하는 case문으로 이동한다.  
문장을 수행하고, break문을 만나면 빠져나간다.  
만약 case1에서 break문이 없다면 case2로 넘어가서 2를 출력하고 break문을 만나 빠져나가게 된다.  
break문을 만나야만 혹은 switch문을 모두 수행해야 빠져나갈 수 있기 때문이다.  
예상못한 결과값을 얻을 수 있으니 주의해야 한다.  
default문은 if-else문의 else 블럭과 같은 역할을 한다.  

### switch문 제약조건
- switch문의 조건식은 정수 혹은 문자열이어야 한다.
- case문의 값은 정수, 상수, 문자열은 가능하지만 실수, 변수는 불가하다.
- case문의 값은 반드시 중복이 되지 않아야한다.

---

## 반복문
반복문은 특정 구간이 반복적으로 수행해야 될 때 사용된다.  
반복문의 종류는 for문, for-each문, while문, do-while문이 있다.  
for문과 while문은 유사한 구조를 가지고 있으며, 어느 경우에도 서로 변환을 할 수 있다.  
보통 반복 횟수를 알고 있다면 for문을, 모르는 경우에는 while문을 사용한다.  

### for문

for문의 구조는 아래와 같다.

```java
for (초기화; 조건식; 증감식) {   
    // 조건이 true인 동안 수행할 동작
}
```

아래의 for문은 블록안의 문장을 10번 반복한다.

![error](/assets/images/whiteship-live-study/2020-12-31/for_loop.png)  

우선 변수 i에 1을 저장한다.  
그리고 반복마다 i가 1씩증가하다가 10을 넘어서면 조건식이 false가 되어 반복을 마치게된다.

초기화 조건식 증감식은 상황에 따라 생략할 수 있다.  

```java
for (;;) { }
```

이런 경우 조건식은 true로 무한 반복문이 된다.  

---

### for-each문
for each라는 키워드가 있는건 아니고, for문의 조건식 부분이 조금 다르다.  

```java
String[] languages = {"Java", "C++", "Python"};

for (String langueage : languages) {
    System.out.println(langueage);
}
```

for문과 비교해 굉장히 간결한 것을 알 수 있다.  

languages는 반복을 할 객체이고, 이 객체에서 한개씩 순차적으로 langueage에 대입이 되어 for문을 수행한다.      
이때 객체는 배열이나 List 형태여야 한다.  

---

### while문

for문 보다 간단한 구조로 조건식과 블럭만 가진다.  
while문은 조건식이 true인 동안 반복된다.  

```java
while (조건식) {
    
}
```

또한 while문과 for문은 어느 경우에도 서로 변환 가능하다고 했는데,  
for문에서 살펴봤던 예제를 while문으로 변경하면 이렇게 된다.

```java
int i = 1;  // 초기화

while (i <= 10) {   // 조건식
    System.out.println(i);
    i++;    // 증감식
}
```

for문의 초기화, 조건식, 증감식을 분리해놓은것이 보인다.  
위와 같은 경우엔 while문 보다는 for문이 더 간결하고 알아보기 쉽다.  
그러나 초기화나 증감식이 필요하지 않다면 while문이 더 적합할 것이다.

---

### do-while문
while문의 변형으로 기본 구조는 while문과 같지만 조건식과 블럭의 순서를 바꿔놓은 것이다.  
그렇기때문에 블럭을 먼저 수행하고 조건을 계산한다.  

```java
do {
    // 조건식이 true이면 실행된다. 처음 한번은 무조건 실행된다.
} while (조건식);
```

---

### break문
switch문에서 잠깐 살펴본 break문이다.  
반복문에서도 사용할 수 있는데, break문을 사용하면 가장 가까운 반복문을 벗어날 수 있다.  

예를들어 for문과 while문 둘다 무한루프를 돌 수 있다.  
의도한것이라면 상관없지만, 의도한 것이 아니라면 큰 문제가 될 수 있다.  
StackOverFlowError를 만나게 될 것 이다...  

이럴 때 if문과 break문을 적절히 사용하면 된다.  

```java
int i = 1;

while (true) {
    if (i > 10) {
        break;    
    }
  
    System.out.println("무한 루프!");
    i++;
}
```

---

### continue문
continue문은 반복문 내에서만 사용 가능하다.  
반복문 전체를 벗어나는 break문과는 다르게  
반복문 안에서 continue문을 만나면 해당 반복문의 끝으로 이동해 다음 반복을 한다.  

만약 1부터 10까지 반복하는 for문에서 짝수만을 출력하고 싶다면 아래처럼 할 수 있다.  

```java
for (int i = 1; i <= 10; i++) {
    if (i % 2 != 0) {
        continue;
    }
    
    System.out.println(i);
}
```

---

## JUnit 5
[백기선 - 더 자바, 애플리케이션을 테스트하는 다양한 방법](https://www.inflearn.com/course/the-java-application-test) 을 많이 참고했다.   
테스트코드 작성이 어렵게만 느껴졌었는데,  
강의를 보면서, 또 4주차 과제를 통해 이것저것 체험하면서 나름 테스트코드 작성에 익숙해지지 않았나 싶다.   

<br>

JUnit은 테스트 코드를 작성할때 사용하는 도구이다.   
테스트 작성을 하는 자바 개발자 중 90% 이상이 사용한다고 한다.  

### Junit 5의 특징
- 기존 JUnit 4는 단일 jar였지만, JUnit 5는 JUnit Platform, JUnit Jupiter, JUnit Vintage 모듈로 구성되어있다. 
  - Platform : 테스트를 싱행해주는 런처 제공. TestEngine API 제공
  - Jupiter : TestEngine API 구현체로 JUnit 5 제공
  - Vintage : JUnit 4와 3를 지원하는 TestEngine 구현체
- 자바 8 이상 버전을 사용해야 한다.
- 테스트 클래스, 테스트 메서드에 public를 선언하지 않아도 된다.  

### JUnit 5 의존성 

만약 스프링 부트를 사용한다면 기본적으로 JUnit 5 의존성이 추가된다.  

스프링 부트를 사용하지 않는다면 pom.xml에 아래와 같은 의존성을 추가해주면 된다. (maven 기준)  

```xml
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter-engine</artifactId>
    <version>5.7.0</version>
    <scope>test</scope>
</dependency>
```

### JUnit 5 기본 사용법

- @BeforeAll, @BeforeEach
  - 테스트 케이스 전에 실행되는 메서드이다. @BeforeAll 애노테이션이 붙은 메서드는 static 이어야한다.
- @AfterAll, @AfterEach
  - 테스트 케이스 이후에 실행되는 메서드이다.
- @DisplayNameGeneration, @DisplayName
 
  - @DisplayNameGeneration
    ```java
    @DisplayNameGeneration(DisplayNameGenerator.ReplaceUnderscores.class)
    class DisplayNameTest() {

      @Test
      void display_name_generation_replace_underscores() { }
    }
    // 클래스내의 모든 테스트 메서드의 언더스코어(_)와 괄호를 지워준다.
    ```

  - @DisplayName
     ```java
     @Test
     @DisplayName("테스트 코드 입니다.")
     void displayName() { }
     
     @Test
     @DisplayName("😋")
     void emoji() { }
     
     // @DisplayName("여기 적힌 이름으로 실행된다.")
     ```
  
    ![error](/assets/images/whiteship-live-study/2020-12-31/displayName.png)  
    <br>

- Assertion
  - assertEquals(exoectedm actual)
    - 실제값이 기대한 값과 같은지 확인
  - assertNotNull(actual)
    - 값이 null이 아닌지 확인
  - assertTrue(boolean)
    - 다음 조건이 true인지 확인
  - assertAll(executables...)
    - 모든 확인 구문 확인
  - assertThrows(expectedType, executable)
    - 예외 발생 확인
  - assertTimeout(duration, executable)
    - 특정 시간 안에 실행 완료되는지 확인
