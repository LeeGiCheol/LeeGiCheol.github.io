---
title : "WhiteShip Live Study 7주차. 패키지"
categories :
- whiteship-live-study
tag :
- whiteship-live-study
toc : true
---

# 작성 중
## WhiteShip Live Study 7주차. 패키지

---

### 학습할 것 (필수)
1. package 키워드
2. import 키워드
3. 클래스패스
4. CLASSPATH 환경변수
5. -classpath 옵션
6. 접근지시자

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

IntelliJ에서 패키지를 생성하는 경우이다.  
![error](/assets/images/whiteship-live-study/2020-12-27/package1.png)  
<br>
![error](/assets/images/whiteship-live-study/2020-12-27/package2.png)  
위와 같이 패키지는 한번에 여러개를 생성할 수도 한번에 하나씩 생성할 수도 있다.  

---

### 2. import 키워
