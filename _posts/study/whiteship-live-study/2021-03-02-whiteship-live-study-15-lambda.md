---
title : "WhiteShip Live Study 15주차. Lambda"
categories :
- whiteship-live-study
tag :
- whiteship-live-study
toc : true
---

## WhiteShip Live Study 15주차. Lambda

---

## 자바의 람다식에 대해 학습하세요.

## 학습할 것 (필수)
- 람다식 사용법
- 함수형 인터페이스
- Variable Capture
- 메소드, 생성자 레퍼런스

---

### 람다식이란? (Lambda Expression)

람다식은 JDK 1.8 부터 추가된 기능이다.  
람다식을 사용함으로써 함수를 간략하면서도 명확하게 표현할 수 있게 해준다.

람다식의 기본적인 생김새는 아래와 같다.

```java
public static void main(String[]args){
    Runnable runnable = () -> System.out.println("Hello Lambda!");        
}
```

이미 쓰레드를 만드는 방법을 공부하고 왔기 때문에  
대략적으로 무슨 행동을 하려는건진 알 것이다.  

그러나 메서드의 이름과 리턴값이 없기 때문에 꽤나 난감할 수 있는 모양새이다.  

위의 람다식을 메서드로 다시 고쳐보자.  

```java
public static void main(String[] args) {
    Runnable runnable = new Runnable() {
        @Override
        public void run() {
            System.out.println("Hello Lambda!");
        }
    };
}
```

@Override 애노테이션을 제외하더라도 5줄의 코드를 단 한줄로 줄일 수 있는  
굉장히 강력한 기능이라는 것은 누구라도 반박할 수 없을 것이다.  

게다가 일반적인 메서드는 클래스에 포함되어야 하기 때문에  
클래스가 필요하고, 객체를 생성해서 호출해야하지만,  
람다식을 사용하면 이와 같은 과정은 필요없어진다.  

자바스크립트와 같은 언어처럼 함수를 변수처럼 사용할 수 도 있는 것이다.  

이처럼 이름과 반환값이 없어지므로 람다식을 익명함수 (anonymous function)이라고 부르기도 한다.  

---

### 람다식 사용법

람다식을 만드는 방법은 일반 메서드에서 이름과 반환값을 지우고,  
매개변수 선언부와 블럭 ('{ }') 사이에 '->' 이러한 화살표를 추가하면 된다.  

```java
@FunctionalInterface
public interface LambdaTest {
    int sum(int a, int b);
}
```

위와 같은 인터페이스가 있을 때 람다식을 통해 사용해 보겠다.    

```java
public static void main(String[]args){
    LambdaTest lambdaTest = (int a, int b) -> a + b;

    LambdaTest lambdaTest2 = (a, b) -> a + b;
        
    LambdaTest lambdaTest3 = (int a, int b) -> {
        System.out.println("괄호를 사용하면 이런 것도 가능하다");
        return a + b;
    };
    
    // 매개변수가 하나인 경우
    LambdaTest lambdaTest4 = a -> a + 10;

    // 이건 불가능하다.
    LambdaTest lambdaTest5 = (int a, int b) -> 
        return a + b;
    
}
```

위처럼 반환값이 있는 메서드의 경우라면  
return 대신 식을 대신해서 사용할 수 있다.  
이때 식의 연산 결과가 자동으로 반환 값이 된다.   

2번의 경우 소괄호 안의 매개변수의 타입이 아예 없다.  
이것은 람다식의 매개변수 타입은 타입 추론이 가능하기 때문인데,  
대부분의 경우 생략 가능하다.  
연산 결과가 자동으로 반환 값이 되는 것도 항상 추론을 할 수 있기 때문이다.  

3번처럼 중괄호를 사용해 여러 문장을 사용할 수 있다.  
매개변수가 하나인 경우 4번처럼 소괄호도 생략 가능하다.  

이때 주의할 점은, 5번과 같이 괄호 안의 문장이 return 문이라면 괄호를 생략할 수 없다.  

---

### 함수형 인터페이스

---

### Variable Capture

---

### 메소드, 생성자 레퍼런스

---