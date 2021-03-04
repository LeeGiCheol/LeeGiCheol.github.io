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

함수형 인터페이스란 한개의 추상 메서드를 가진 인터페이스를 말한다.  
Single Abstract Method(SAM)라고 부르기도 한다. (코틀린에서 더 많이 쓰이는 듯 하다.)  

위 예제 LambdaTest 인터페이스에 붙은 @FunctionalInterface 애노테이션이 바로 함수형 인터페이스라고 명시해주는 애노테이션이다.  

![error](/assets/images/whiteship-live-study/2021-03-02/@FunctionalInterface-1.png)  

![error](/assets/images/whiteship-live-study/2021-03-02/@FunctionalInterface-2.png)  

위와 같이 추상 메서드가 아예 없거나, 한개보다 많다면 컴파일 에러가 발생한다.   
마치 @Override 처럼 말이다.  
  
<br>  

단 default 메서드의 경우는 예외이다.  

![error](/assets/images/whiteship-live-study/2021-03-02/@FunctionalInterface-3.png)  

인터페이스를 구현할때 추상 메서드는 반드시 구현해야 하지만,  
default 메서드는 필수가 아닌 것을 생각하면 된다.  

---

### Variable Capture

람다식은 지역변수를 참조할 수 있다.  

![error](/assets/images/whiteship-live-study/2021-03-02/variable-capture-1.png)  

![error](/assets/images/whiteship-live-study/2021-03-02/variable-capture-2.png)  

단 사용되는 변수는 final 이거나 final 처럼 쓰여야한다.  
만약 str 이라는 변수의 값이 변한다면 컴파일 에러가 발생한다.

![error](/assets/images/whiteship-live-study/2021-03-02/variable-capture-3.png)

참고로 final 처럼 쓰인다는 것. 이 것의 명칭은 effectively final 이다.  
final 키워드를 붙이진 않았지만, 재할당하지 않고 참조가 변경되지 않는 변수를 effectively final이라 한다.  

#### 왜 final만 가능할까?

위 helloLambda 메서드를 호출한 스레드의 스택에 str 이라는 변수도 만들어진다.

![error](/assets/images/whiteship-live-study/2021-03-02/variable-capture-4.png)  

당연하겠지만 stack() 호출이 끝나면 str 변수도 스택에서 제거된다.  

![error](/assets/images/whiteship-live-study/2021-03-02/variable-capture-5.png)  

리턴된 람다식은 어떤 스레드에서 다시 호출될지 모르고, str 변수는 이미 사라져버렸으며, 다른 스레드의 스택 영역에 있으므로 접근도 못한다.    
이런 문제를 해결하기 위해 생긴 기능이  
변수의 복사본을 만들어 접근하도록 하는 Variable Capture이다.

복사본을 사용하는 것이지만, 람다식이 언제 몇개의 스레드에서 사용될지는 아무도 모르기 때문에  
final이 아니라면 동기화를 할 수 없기 때문에 final 또는 effectively final을 사용해야한다.

#### 의문. 스택??

지역변수는 메서드 호출이 끝나면 스택에서 제거되니까 그렇다 치고    
static 변수나 instance 변수는 어떨까 싶은 의문이 들었다.  

static 변수는 클래스 로딩될때 한번, instance 변수는 Heap 영역에 생성되기 때문이다.  
바로 테스트해보자.  

```text 
static
```

![error](/assets/images/whiteship-live-study/2021-03-02/variable-capture-static.png)   

<br><br>

```text 
instance
```

![error](/assets/images/whiteship-live-study/2021-03-02/variable-capture-instance.png)   

컴파일 에러가 발생하지도 않은 뿐더러 이상없이 동작한다.  

스태틱변수나 인스턴스변수는 동일한 변수를 참조할 수 있기 때문이다.
그러면 컴파일러는 람다가 heap 영역 안의 str 값인 즉 가장 최신의 값을 참조하도록 보장할 수 있다.  

대신 Thread Safe 하지 않기 때문에 멀티스레드 환경에서는 주의해서 사용해야 한다.

---

### java.util.function

대부분 메서드의 생김새는 비슷하다.  
매개변수가 없거나, 한개 또는 두개.  
리턴값이 있거나, 없거나.  

그렇기 때문에 java.util.function 패키지에  
자주 쓰이는 메서드 형식을 함수형 인터페이스로 미리 정의해두었다.  
함수형 인터페이스에 정의된 메서드 이름도 통일되고, 재사용성이나 유지보수도 좋다.  
그렇기에 가능하면 새로운 함수형 인터페이스를 만드는 것보단  
이 패키지의 인터페이스를 사용하는 것이 좋다.  

| 함수형 인터페이스       | 메서드                    | 설명                                                      |
| ------------------- | ----------------------- | -------------------------------------------------------- |
| java.lang.Runnable  | void run()              | 매개변수 리턴값 없음.                                         |
| Supplier<T>         | T get()                 | 매개변수는 없고 리턴값은 있음.                                  |
| Consumer<T>         | void accept(T t)        | 매개변수는 있고 리턴값은 없음.                                  |
| Function<T, R>      | R apply(T, t)           | 일반적인 함수. 하나의 매개변수를 받아서 결과 리턴                   |
| Predicate<T>        | boolean test(T t)       | 조건식을 표현하는데 사용한다. 매개변수는 하나. 리턴 타입은 boolean    |
| BiConsumer<T, U>    | void accept(T t, U u)   | 두 개의 매개변수가 있고, 리턴값이 없음                           |
| BiPredicate<T, U>   | boolean test(T t, U u)  | 조건식을 표현하는데 사용한다. 매개변수는 둘. 리턴 타입은 boolean      |
| BiFunction<T, U, R> | R apply(T t, U u)       | 두 개의 매개변수를 받아서 결과 리턴                              |

맨 위 4개의 함수형 인터페이스는 매개변수와 반환값의 유무에 따라 정의되어있고,  
Predicate는 조건식을 함수로 표현하는데 사용되며 반환값이 boolean인 것을 제외하면 Function과 동일하다.  

```java
public static void main(String[]args){
    Predicate<LocalDateTime> now = currentTime -> LocalDateTime.now().isAfter(currentTime);
    System.out.println(now.test(LocalDateTime.now().minusMinutes(10)));
}
```

입력한 시간이 현재 시간보다 이전 시간이라면 true를 반환하는 메서드이다.  
이런식으로 Predicate를 활용할 수 있다.  

맨 아래 3개는 접두사로 Bi가 붙는다.  
이는 매개변수가 2개인 함수형 인터페이스이다.  

메서드는 두 개의 값을 리턴할 수 없으므로 BiSupplier는 존재하지 않는다.  

만약 매개변수가 두개보다 많은 경우라면 직접 함수형 인터페이스를 만들어야 한다.  

```java
// 매개변수가 3개인 경우
@FunctionalInterface
public interface TriFunction<T, U, V, R> {
    R apply(T t, U u, V v);
}

// 매개변수가 4개인 경우
@FunctionalInterface
public interface QuadFunction <T, U, V, W, R> {
    R apply(T t, U u, V v, W w);
}
```

---

### 메소드, 생성자 레퍼런스

#### 메서드 레퍼런스

```java
public int parseInt(String str) {
    return Integer.parseInt(str);
}
```

위와 같은 메서드가 있다.  
메서드가 굉장히 간단하고 하는일이 크게 없다.  
이런 경우 메서드 레퍼런스라는 방법으로 메서드를 간략하게 표현할 수 있다.  

먼저 람다식으로 표현하면 아래와 같다.  

```java
public static void main(String[]args){
    Function<String, Integer> function = (String s) -> Integer.parseInt(s);        
}
```

메서드 레퍼런스를 사용하면 아래와 같이 코드가 간결해진다.  

```java
public static void main(String[]args){
    Function<String, Integer> function = Integer::parseInt;        
}
```

즉 하나의 메서드만을 호출하는 람다식은  
클래스이름::메서드 이름 또는 참조변수::메서드 이름으로 변환할 수 있다.  

#### 생성자 레퍼런스

생성자를 호출하는 람다식을 메서드 레퍼런스로 변환할 수 있다.  

```java
Supplier<ConstructorReferenceClass> s = () -> new ConstructorReferenceClass();  // 람다
```

```java
Supplier<ConstructorReferenceClass> s = ConstuctorReference::new;               // 메서드 레퍼런스
```

만약 생성자에 매개변수가 있다면 이렇게 사용할 수 있다.  

```java
Function<Integer, ConstructorReferenceClass> s = (i) -> new ConstructorReferenceClass(i);
Function<Integer, ConstructorReferenceClass> s = ConstructorReferenceClass::new;
```

매개변수가 두개라면 BiFunction을 사용한다.    

```java
BiFunction<Integer, Integer,ConstructorReferenceClass> s = ConstructorReferenceClass::new;
```

메서드 레퍼런스를 사용함으로써 람다식을 마치 static한 변수처럼 다룰 수 있다.

---

# 참고
[정아마추어님 블로그](https://jeong-pro.tistory.com/211)  
[남궁성 - 자바의 정석](http://www.yes24.com/Product/Goods/24259565)

---