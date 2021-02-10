---
title : "WhiteShip Live Study 13주차. IO"
categories :
- whiteship-live-study
tag :
- whiteship-live-study
toc : true
---

## WhiteShip Live Study 13주차. IO

---

## 목표
자바의 Input과 Ontput에 대해 학습하세요.

## 학습할 것 (필수)
- 스트림 (Stream) / 버퍼 (Buffer) / 채널 (Channel) 기반의 I/O
- InputStream과 OutputStream
- Byte와 Character 스트림
- 표준 스트림 (System.in, System.out, System.err)
- 파일 읽고 쓰기

---

### I/O ?

I는 Input, O는 Output의 약자로 입력과 출력을 의미하며,  
입출력은 컴퓨터 내부 또는 외부의 장치와 프로그램 간의 데이터를 주고 받는 것이다.  
가장 간단하게 생각하자면 키보드로는 문자를 입력 하고, 스피커로는 소리를 출력받는다는것으로 예를 들 수 있다.     

---

### 스트림 (Stream) / 버퍼 (Buffer) / 채널 (Channel) 기반의 I/O

#### 스트림
자바에서 입출력을 수행하려면 두 대상을 연결하고 전송할 수 있는 무언가가 필요하다.  
이때 무언가라는 것이 바로 스트림이다.  

#### 채널
사실 채널도 마찬가지이다.  
두 가지의 차이점을 보자면 읽거나, 쓰거나 하나만 가능한 스트림에 비해      
채널은 읽고 쓰고가 모두 가능한 양방향이다.  

IO에서는 스트림, NIO에서는 채널을 사용한다.  
IO와 NIO에 대해서는 마지막에 다루겠다.

#### 버퍼
버퍼는 임시저장공간이다.  
생활 속에서 버퍼를 자주 볼 수 있는데(요새는 많이 줄었지만..)    
'동영상 시청 중 버퍼링 걸린다' 의 버퍼링이 바로 버퍼이다.  

서버와 클라이언트 간의 통신에도 버퍼는 상당히 많이 사용된다.  
TCP 통신의 각 소켓에는 send buffer와 receive buffer가 있다.  
이때 쓰이는 buffer는 패킷의 유실방지 및 성능개선 등의 이유로 사용된다.  

![error](/assets/images/whiteship-live-study/2021-02-07/tcp-buffer.png)

네트워크 스터디가 아니니 가볍게 버퍼가 무엇인지에 초점을 두고 생각해보자.  

화살표는 각 패킷이다. 만약 1~3번을 전송하던 중 2번이 유실됬다.  
3번을 받았지만 2번이 유실됬으므로 (혹은 아직 도착하지 않았으므로) receive buffer에 보관해둔다.  
추후 2번 패킷을 다시 받게 되면 버퍼에 담아둔 3번 패킷까지 한번에 다 받을 수 있다.  

---

### InputStream과 OutputStream

스트림은 데이터를 바이트단위로 전송한다.  
이때 입출력 스트림은 InputStream과 OutputStream을 사용한다.  
두개의 스트림은 추상 클래스이며, 입출력 대상에 따라  
추상메서드를 구현한 클래스들이 존재한다.  


| 입출력 대상               | 스트림                                                                        |
| :--------------------: | -------------------------------------------- |
| 파일                    | FileInputStream <br> FileOutputStream            |
| 메모리 (Byte배열)         | ByteArrayInputStream <br> ByteArrayOutputStream  |
| 프로세스                 | PipedInputStream <br> PipedOutputStream          |
| 오디오장치               | AudioInputStream <br> AudioOutputStream           |

---

### 보조 스트림

보조 스트림은 실제로 데이터를 주고받는 스트림은 아니지만,   
스트림의 기능을 향상시키거나, 새로운 기능을 추가하는 역할을 한다.  
그렇기 때문에 스트림을 먼저 생성한 다음, 이를 이용해서 보조 스트림을 생성한다.  

---

### Byte와 Character 스트림
자바는 한 문자를 의미하는 char가 1바이트가 아닌 2바이트이다.  
이를 처리하는데 어려움이 있었는데 이 점을 보완하기 위해 Character 스트림이 제공된다.  

InputStream은 Reader, OutputStream은 Writer로 바꿔 사용하면 된다.  
단 ByteArrayInputStream의 문자기반 스트림은 CharArrayReader이다.  

| Byte Stream                                         | Character Stream                                                                        |
| :-------------------------------------------------: | ------------------------------------- |
| FileInputStream <br> FileOutputStream               | FileReader <br> FileWriter            |
| ByteArrayInputStream <br> ByteArrayOutputStream     | CharArrayReader <br> CharArrayWriter  |
| PipedInputStream <br> PipedOutputStream             | PipedReader <br> PipedWriter          |
| AudioInputStream <br> AudioOutputStream             | AudioReader <br> AudioWriter          |

두 가지의 차이점은 읽고 쓰기에 사용되는 메서드에 매개변수가  
byte배열과 char배열인 것 말고는 사용법은 거의 동일하다.  

**Byte Stream**  
![error](/assets/images/whiteship-live-study/2021-02-07/byte-stream.png)  

**Character Stream**  
![error](/assets/images/whiteship-live-study/2021-02-07/char-stream.png)  

---

### 표준 스트림 (System.in, System.out, System.err)

표준 입출력은 콘솔로의 데이터 입력과 출력을 의미한다.  
자바는 표준 입출력을 위해 System.in, System.out, System.err 세 가지 입출력 스트림을 제공한다.  
이 세 가지 스트림은 자바 애플리케이션의 실행과 동시에 사용할 수 있게  
자동적으로 생성이 되므로 별도로 스트림을 생성하지 않고도 사용 할 수 있다.  

```java
public static void main(String[] args) {
    Scanner scanner = new Scanner(System.in);
    String input = scanner.next();

    System.out.println(input);
}
```

아마 가장 손쉽게 찾아볼 수 있는 예시일 것 이다.
Scanner의 인자값으로 System.in을 넣는다는 것에서 눈치를 챘겠지만,  
Scanner를 통해 콘솔에 데이터를 입력할 수 있다.  
또한 입력한 값을 System.out.println()을 사용함으로써  
콘솔에 데이터를 출력할 수 있다.  

---

### 파일 읽고 쓰기

**파일 읽기**  

![error](/assets/images/whiteship-live-study/2021-02-07/file-read.png)  

FileRead.txt 파일을 미리 만들어두었다.  
FileReader를 통해 읽어 올 것이고,  
문자열로 받아올 것이니 BufferedReader를 사용했다.  

```java
 public static void main(String[] args) {
    try {
        BufferedReader bufferedReader = new BufferedReader(new FileReader(new File("/Users/Desktop/IO-Exam/FileRead.txt")));
        
        String message = "";

        while ((message = bufferedReader.readLine()) != null) {
            System.out.println(message);
        }
        
        bufferedReader.close();

    } catch (IOException e) {
        e.printStackTrace();
    }
}

// 파일 읽고쓰기 테스트입니다.
```

BufferedReader 는 아래의 세 가지 객체를 한줄로 만든 것이다.  

```java
File file = new File("/Users/Desktop/FileRead.txt");
FileReader fileReader = new FileReader(file);
BufferedReader bufferedReader = new BufferedReader(fileReader);
```

**파일 쓰기**

다음은 파일 쓰기이다.  
File 객체를 사용해서 새로운 txt 파일을 만들 것 이다.  
FileWriter를 통해 읽어 올 것이고,  
BufferedWriter를 사용했다.  

```java
public static void main(String[] args){
    File file = new File("/Users/Desktop/IO-Exam/FileWrite.txt");

    try{
        FileWriter fileWriter = new FileWriter(file);
        BufferedWriter bufferedWriter = new BufferedWriter(fileWriter);

        if(file.isFile() && file.canWrite()){
            bufferedWriter.write("파일 쓰기 테스트입니다.");
            bufferedWriter.newLine();
            bufferedWriter.write("두 번째 라인");
        
            bufferedWriter.close();
        }
    }catch(IOException e){
        e.printStackTrace();
    }
}
```

![error](/assets/images/whiteship-live-study/2021-02-07/file-write.png)  


물론 BufferedWriter 또한 아래와 같이 한줄로 줄일 수 있다.  

```java
BufferedWriter bufferedWriter = new BufferedWriter(new FileWriter(new File("/Users/ekkkk1/Desktop/IO-Exam/FileWrite.txt")));
```

---

### IO / NIO

제일 처음에 IO는 컴퓨터 내부 또는 외부의 장치와 프로그램 간의 데이터를 주고 받는 것이라고 했었다.  
그럼 NIO는 무엇일까?  

Blocking을 사용하는 IO와는 다르게 NIO에는 Non-Blocking이라는 것이 추가되었다.  
원래는 New Input/Output의 약자였는데 2002년에 도입되었으니 new 라는 키워드를 붙이긴 조금 애매해졌다.    
그렇기때문에 대부분 사람들은 NIO를 논블로킹 입출력으로 받아들인다.   

---

### Blocking Non-Blocking

#### Blocking

![error](/assets/images/whiteship-live-study/2021-02-07/blocking.png)

블로킹은 위와 같이 소켓마다 새로운 Thread를 할당해야한다.  
이러한 방식의 문제점은 크게 세 가지가 있다.  

첫째 여러 스레드가 입출력 데이터가 들어오기를 기다리며 무한정 대기 상태에 빠질 수 있다.  
이는 리소스 낭비로 이어질 가능성이 높다.  
둘째 각 스레드가 스택 메모리를 할당해야 하는데, 스택의 기본크기는 64KB ~ 1MB 까지 차지할 수 있다.  
셋째 JVM은 물리적으로 아주 많은 수의 스레드를 지원할 수 있지만, 컨텍스트 전환에 따른 오버헤드로 심각한 문제가 생길 수 있다.  

10만 이상의 동시 연결을 지원해야 하는 경우에는 처음부터 배제하는 것이 바람직하다.  

#### Non-Blocking

![error](/assets/images/whiteship-live-study/2021-02-07/non-blocking.png)  

이 그림 한장으로도 앞서 봤던 Blocking의 문제점이 해결된 것을 알 수 있을 것이다.  
Selector 클래스는 논블로킹 입출력의 핵심이다.  
이를 통해 한 스레드로 여러 동시 연결을 처리할 수 있는데,  
이러한 방식은 블로킹 입출력에 비해 훨씬 개선된 리소스 관리 효율을 보여준다.  

Blocking에 비해 Non-Blocking은 구현이 어려운 편이다.  
Netty나 Mina와 같은 프레임워크를 공부하는것도 좋은 방법이다.  

---

# 참고
[남궁성 - 자바의 정석](http://www.yes24.com/Product/Goods/24259565)  
[Norman Maurer , Marvin Allen Wolfthal - 네티션 인 액션](http://www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9791158390327&orderClick=&Kc=)

---





