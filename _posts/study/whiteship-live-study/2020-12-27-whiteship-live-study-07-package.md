---
title : "WhiteShip Live Study 7주차. 패키지"
categories :
- whiteship-live-study
tag :
- whiteship-live-study
toc : true
---

## WhiteShip Live Study 7주차. 패키지

---

### 학습할 것 (필수)
1. package 키워드
2. import 키워드
3. 클래스패스
4. CLASSPATH 환경변수
5. -classpath 옵션
6. 접근제어자

---

### 1. package 키워드
- 패키지란 
  - 클래스 혹은 인터페이스의 모임을 뜻한다.
  - 클래스의 이름을 중복으로 사용했을 때 충돌을 막아준다.
  - 폴더를 사용해 파일 정리를 하듯이 패키지를 나누어 클래스를 기능별로 나눌 수 있다.

  
- 패키지 키워드
```java
package whiteship.live.study;
```

패키지는 항상 맨 첫번째 줄에 한번만 선언할 수 있다.  
선언을 하지 않는다면 이름이 없는 default package에 속하며 default package에 클래스를 정의한 것이다.  

관습적으로 패키지명은 모두 소문자를 사용한다.  

다음은 IntelliJ에서 패키지를 생성하는 경우이다.  
![error](/assets/images/whiteship-live-study/2020-12-27/package1.png)  
<br>
![error](/assets/images/whiteship-live-study/2020-12-27/package2.png)  
위와 같이 패키지는 한번에 여러개를 생성할 수도 한번에 하나씩 생성할 수도 있다.  

---

### 2. import 키워드
다른 패키지의 클래스를 사용하려면 사용할 때 마다 소스코드에 package명을 사용해야한다.  
그러나 이러한 방법은 너무나도 불편하다.  
이때 import 키워드를 선언함으로써 클래스 이름만으로도 클래스를 사용할 수 있다.

```java
import java.util.ArrayList;

pubic class Main {
  public static void main(String[] args) { 
      //java.util.ArrayList arrayList = new java.util.ArrayList();  // 이렇게 사용할 것을
      ArrayList arrayList = new ArrayList();                        // 이렇게 사용하게 해준다.
  }
}
```

아래와 같이 와일드카드도 사용 가능하다.  


```java
import java.util.*;

pubic class Main {
  public static void main(String[] args) { 
      ArrayList arrayList = new ArrayList();
      Scanner scan = new Scanner(System.in);
  }
}
```


**단** java.lang 패키지는 자바 프로그래밍에 있어서  
가장 기본적인 클래스가 모인 패키지임으로, import문을 사용하지 않아도 사용할 수 있다.  
대표적으로 String 클래스나, System 클래스 등이 있다.  

---

### 클래스패스

클래스패스는 .class 파일이 포함된 디렉토리와 파일을 콜론(:)으로 구분한 목록이다.  

소스코드를 작성하고 컴파일을 하면 바이트코드로 변환되어 ***.class 파일이 생긴다.  
이 클래스 파일을 실행하기위해서는 우선 클래스 파일을 찾을 수 있어야 한다.  
이때 클래스패스에 지정된 경로를 통해 클래스 파일을 찾을 수 있다.  

---

### CLASSPATH 환경변수

사실 맥의 경우 환경변수는 배포환경이 아니라면 IDE에서 이미 잡아주기 때문에 설정하지 않아도 큰 문제는 없다.  
맥에서 환경변수 설정하는 방법  

```shell
$ vi ~/.bash_profile


export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk-11.0.9.jdk/Contents/Home   # 본인의 자바 경로를 작성한다.
export PATH=${PATH}:$JAVA_HOME/bin
```

```shell
$ source ~/.bash_profile
$ echo $PATH
```

![error](/assets/images/whiteship-live-study/2020-12-27/classpath6.png)  
<br>
나의 경우 파이썬의 환경변수가 먼저 잡혀있었기 때문에 빨간색으로 표시한 부분만 자바의 환경변수이다.

---

### -classpath 옵션

한가지 예시를 보겠다.  

![error](/assets/images/whiteship-live-study/2020-12-27/classpath1.png)  
<br>
![error](/assets/images/whiteship-live-study/2020-12-27/classpath2.png)  
<br>
아래를 보면 클래스를 두개 생성하고, 컴파일후 실행까지 했지만 전혀 이상이 없다.  
하지만 MoveClass.class를 move라는 폴더로 옮기면 어떻게 될까?

![error](/assets/images/whiteship-live-study/2020-12-27/classpath3.png)  
<br>
![error](/assets/images/whiteship-live-study/2020-12-27/classpath4.png)  
<br>
위와같이 **NoClassDefFoundError**가 발생하게 된다.  
여기서 클래스패스를 설정해주면 다른 폴더에 있는 클래스 파일도 실행 할 수 있게 된다.

![error](/assets/images/whiteship-live-study/2020-12-27/classpath5.png)  
<br>
아까와 달라진건 **<span style="color: rgb(255, 204, 102)">java -classpath ".:move" ClasspathTest</span>** 라는 명령어 뿐이다.  
java ClasspathTest 명령어에 **<span style="color: rgb(255, 204, 102)">-classpath ".:move"</span>** 가 붙었을 뿐인데 실행은 성공했다.  

**.:move**의 의미는 **.** 이 폴더에서 찾아보고, 만약 존재하지 않는다면 **:move** move 폴더에서 찾아라. 라는 의미이다.  
:은 구분자 역할이며 윈도우에서는 :(콜론) 대신 ;(세미콜론)을 사용한다고 한다.  

참고사항으로 **-classpath** 옵션은 줄여서 **-cp** 로도 사용 가능하다.  

---

### 접근제어자

접근제어자는 변수, 메서드, 생성자, 클래스에 사용해 외부에서 접근하지 못하도록 제한하는 역할을 한다.  
그 이유는 클래스의 내부에 선언된 데이터를 **보호**하기 위함이다.  
데이터가 유효한 값을 유지하도록, 또는 비밀번호와 같은 데이터를 외부에서 함부로 변경하지 못하게 외부의 접근 제한이 필요하다.  

이런 경우 해당 변수를 private이나 protected로 제한을 하고,  
반드시 따라야하는것은 아니지만 일반적으로  
get으로 시작하는 변수의 값을 읽는 메서드와  
set으로 시작하는 변수의 값을 변경할 수 있는 메서드를 public으로 생성해 사용한다.  
이것을 **<span style="color: rgb(255, 204, 102)">getter, setter</span>** 라고 부른다.

이것은 객체지향언어의 특징 중 하나인 캡슐화에 해당된다.   

```text
접근제어자가 사용될 수 있는 곳 - 클래스, 멤버변수, 메서드, 생성자

public          접근 제한이 없다.
protected       같은 패키지 내에서 접근 가능하며, 다른 패키지의 자식 클래스에서 접근 가능하다.
default         같은 패키지 내에서만 접근 가능하다.
private         같은 클래스 내에서만 접근 가능하다.
```

![error](/assets/images/whiteship-live-study/2020-12-27/accessModifier.png)  

default의 경우 접근제어자가 default임을 알리기 위해 default를 붙이지 않는다.  
만약 접근제어자가 붙어있어야 하는 곳에 붙어있지 않다면  
접근제어자가 default임을 뜻한다.

---





