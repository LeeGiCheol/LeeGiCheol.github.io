---
title : "자바 Method Reference"
categories : 
    - java
tag :
    - MethodReference
toc : true
---

## Method Reference

자바8에 등장한 Method Reference는 Lambda 표현식을 더욱 간결하게 만들어주는 문법이다.

```java
public class MyMethodReference {

    public static void main(String[] args) {
        int number1 = 10;
        int number2 = 100;

        BiFunction<Integer, Integer, Integer> function1 = (Integer i, Integer j) ->  {
            return i + j;
        };        
				
        Integer apply = function.apply(number1, number2);
        
        System.out.println("apply = " + apply);
    }

}
```

Integer 파라미터 두개를 받아서, 두 값을 더한 값을 리턴하는 간단한 람다 문법이다.

이것을 더 줄이려면 어떻게 해야할까?

파라미터의 타입은 추론이 가능하니, 타입을 제거하고, 

리턴하는 코드가 한줄이기 때문에, 중괄호는 제거할 수 있다.

이때 중괄호를 제거하면 BiFunction은 파라미터 두개를 받아 리턴하는 Functional Interface이기 때문에,

리턴하는 것을 추론할 수 있기 때문에, 필수적으로 return 키워드는 제거해야한다.

위 코드는 해당이 되지 않지만, 파라미터가 한개 뿐이라면, 파라미터를 감싸는 괄호 또한 제거 가능하다.

코드를 줄이면 아래와 같다.

```java
public class MyMethodReference {

    public static void main(String[] args) {
        int number1 = 10;
        int number2 = 100;

        BiFunction<Integer, Integer, Integer> function1 = (i, j) -> i + j;
				
        Integer apply = function.apply(number1, number2);
        
        System.out.println("apply = " + apply);
    }

}
```

위에서 말한 것과 같이 자바8 에서는 이보다 더욱 람다식을 줄일 수 있는 방법을 고안해냈는데, 

이것이 Method Reference이다.

```java
public class MyMethodReference {

    public static void main(String[] args) {
        int number1 = 10;
        int number2 = 100;

        BiFunction<Integer, Integer, Integer> function = Integer::sum;
        Integer apply = function.apply(number1, number2);
        
        System.out.println("apply = " + apply);
    }

}
```

메서드 레퍼런스는 `ClassName::MethodName` 으로 표현한다.

메서드를 호출 할 때 괄호를 쓰는것이 일반적이지만, 메서드 레퍼런스는 괄호를 사용하지 않는다.

---

### Method Reference 조건

### Method Signature

메서드 시그니처는 헷갈릴 수 있는 용어이다.

`Overloading`의 조건을 의미하는데,

`Method Name, Method Parameter Types` 이 일치하지 않는다면 오버로딩을 사용할 수 있다. 

(참고로 Method Return Type은 **절대 포함되지 않는다**.)

메서드 레퍼런스와는 관련없는 용어이다.

### Method Type

Return Type, Method Type Parameter, Method Argument Types, Exception

이 네 가지를 묶어 메서드 타입이라고 말하며, 이것이 모두 일치할 때 Method Reference를 사용할 수 있다.

---

메서드 레퍼런스는 문법이 어려운건 아니지만, 

사용하기 위해서는 파라미터와 리턴 타입을 모두 알아야 하기 때문에,

자주 쓰는 메서드가 아니라면 사용하기 어려운 문법이다. 

그렇기 때문에 람다식을 만든 후 메서드 레퍼런스로 변환하거나,

IDE의 힘을 빌려 코드를 작성하는 것도 학습에 좋은 방법이라고 생각한다.

---

## 출처

[토비의 봄 TV 1회 - 재사용성과 다이나믹 디스패치, 더블 디스패치 - YouTube](https://www.notion.so/TV-1-YouTube-2ae9c18e0a63439185151a894a0d458c)