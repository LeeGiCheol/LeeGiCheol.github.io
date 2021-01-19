---
title : "WhiteShip Live Study 10주차. 멀티쓰레드"
categories :
- whiteship-live-study
  tag :
- whiteship-live-study
  toc : true
---

## WhiteShip Live Study 10주차. 멀티쓰레드

---

## 목표
자바의 멀티쓰레드 프로그래밍에 대해 학습하세요.

## 학습할 것 (필수)
- Thread 클래스와 Runnable 인터페이스
- Main 쓰레드
- 쓰레드의 상태
- 쓰레드의 우선순위
- 동기화
- 데드락

---

### Process
Thread를 알기전에 Process를 간단하게 먼저 알아보자.  
'실행중인 프로그램'을 Process 라고 부른다.  
프로세스는 프로그램을 수행하는데 필요한 데이터와 메모리와 같은 자원, 그리고 쓰레드로 구성되어 있다.  
모든 프로세스는 하나 이상의 쓰레드가 존재하고, 두개 이상의 쓰레드를 가진 프로세스를 멀티쓰레드 프로세스라고 부른다.  
프로세스는 작업공간이라고 생각할 수 있다.

### Thread
Thread는 무엇일까?  
위에서 프로세스는 프로그램을 수행하는데 필요한 자원을 가진다고 했다.    
이때 이 자원을 이용해서 실제로 작업을 수행하는 것이 쓰레드이다.  
프로세스라는 작업공간에서 작업을 처리하는 일꾼이라고 생각할 수 있다.

아래 코드를 잠깐 확인해 보자

```java
public class AlphabetThread extends Thread {

   @Override
   public void run() {
      for (char i = 'a'; i < 'j'; i++) {
         System.out.println(i);
      }
      System.out.println("alphabetThread end");
   }
}

public class NumberThread extends Thread {

   @Override
   public void run() {
      for (int i = 1; i <= 10; i++) {
         System.out.println(i);
      }
      System.out.println("numberThread end");
   }
}

public class Main {

   public static void main(String[] args) {
      AlphabetThread alphabetThread = new AlphabetThread();
      NumberThread numberThread = new NumberThread();

      alphabetThread.start();
      numberThread.start();
      System.out.println("main end");
   }
}
```
```text
결과
main end
a
b
c
d
1
2
3
4
5
6
7
8
9
10
e
numberThread end
f
g
h
i
alphabetThread end
```

너무나 당연하게도 이렇게 출력되길 원했을 것이다.

```text
a
b
c
...
i   
alphabetThread end  
1
2
3
...
10  
numberThread end  
main end
``` 

근데 순서가 정말 엉망진창으로 나왔다.  
심지어 각 쓰레드의 for문이 끝나고 나오도록 출력한 'xxxThread end'와  
모든 쓰레드가 종료되고 출력할 'main end' 까지도..

이 테스트를 통해 쓰레드는 여러가지 작업을 ```동시에``` 수행할 수 있다는 것을 알 수 있.

자세한건 아래에서 다시 살펴보도록 하자.

### Processor?
프로세서는 하드웨어 관점과 소프트웨어 관점으로 나뉜다.

**하드웨어 관점**
- 프로그램을 수행하는 하드웨어 장치
- ex) CPU

**소프트웨어 관점**
- 데이터 구성방식을 바꾸는 시스템 소프트웨어
- ex) 컴파일러, 워드프로세서

이름이 비슷해서 헷갈리기 쉽다.  
프로세스, 프로세서 엄연히 다른 부분이니 차이를 꼭 기억하자.

---

### Thread 클래스와 Runnable 인터페이스
쓰레드를 구현하는 방법은 두 가지 이다.  
Thread 클래스를 상속받거나, Runnable 인터페이스를 구현하거나

큰 차이는 없지만 일반적으로 Runnable 인터페이스를 구현하는 것을 선호한다.  
스터디를 잘 진행하고 있다면, 이유를 알 수 있을 것이다.  
Thread 클래스를 상속받으면 다른 클래스를 상속 받지 못하는 '제약'이 생기기 때문이다.

Thread 클래스를 상속받은 경우는 위에서 살펴봤으니 Runnable 인터페이스를 구현하는 방법을 알아보겠다.

Thread 클래스를 상속받은 경우와 Runnable 인터페이스를 상속받은 경우의  
인스턴스 생성방법은 약간의 차이가 있다.

```java
public class RunnableThread implements Runnable {

   @Override
   public void run() {
      for (int i = 0; i < 10; i++) {
         System.out.println(Thread.currentThread().getName());
      }
   }

}


public class Main {

   public static void main(String[] args) {
      Runnable runnable = new RunnableThread();
      Thread runnableThread1 = new Thread(runnable);

      // 혹은

      Thread runnableThread2 = new Thread(new RunnableThread());

      runnableThread1.start();
      runnableThread2.start();
   }

}
```

그런데 Thread를 상속받던, Runnable을 구현하던 분명히 run 메서드를 구현했는데,  
정작 메인 메서드는 start 메서드를 호출한다.

무슨 이유일까?

---

### start()
start 메서드를 호출하면 새로운 쓰레드가 작업을 실행 할 호출스택을 생성한다.  
그 이후 run 메서드를 호출해서 생성된 호출 스택에 run 메서드를 올라가게 한다.  
그림으로 표현하자면 아래와 같다.

![error](/assets/images/whiteship-live-study/2021-01-19/stack.png)

다음은 start 메서드의 코드이다. 내부적으로 어떤일이 일어나는지 확인해보자.

![error](/assets/images/whiteship-live-study/2021-01-19/start().png)

빨간색 네모칸의 start0 메서드를 보자.  
아쉽게도 이 메서드는 native 코드로 직접 볼 수는 없었다.

그러나 이 메서드에서 run 메서드를 호출한다는 주석은 확인 할 수 있었다.

```text
If this thread was constructed using a separate Runnable run object, then that Runnable object's run method is called; otherwise, this method does nothing and returns.
Subclasses of Thread should override this method.
See Also:
start(), stop(), Thread(ThreadGroup, Runnable, String)
```

반대로 start 메서드가 아닌 run 메서드를 호출한다는 의미는  
단순히 클래스에 선언된 메서드를 호출하겠다는 의미이다.

---

### main Thread

![error](/assets/images/whiteship-live-study/2021-01-19/stack.png)

이 그림을 다시 한번 살펴보자.

실행중인 프로그램인 프로세스는 하나 이상의 쓰레드를 가진다고 했었다.  
우리는 쓰레드를 만든 적이 없지만 프로그램은 잘 실행됬었다.  
바로 main 메서드도 하나의 쓰레드이기 때문이다. 이것을 main 쓰레드라고 부른다.

알파벳과 숫자를 나열하는 예제에서 alphabetThread와 numberThread가 끝나지 않았는데도 제일 먼저 main end 가 출력 됐었다. (순서는 항상 랜덤하다.)  
프로그램은 실행 중인 쓰레드가 하나도 없을 때 종료되는데  
main 메서드가 모든 작업을 수행하였더라도,  
다른 쓰레드가 아직 작업중이기 때문에 프로그램은 종료되지 않았던 것이다.

---

### MultiThread
멀티쓰레드는 여러 쓰레드를 동시에 실행시키는 것을 의미한다.  
자원을 효율적으로 사용할 수 있으며, CPU의 사용량을 향상시킬 수 있다.

게임은 동시다발적으로 일어나는 일이 굉장히 많다.  
예를들어 몬스터를 공격하는 동안 몬스터는 아무일도 하지못한다면 굉장히 재미없는 게임이 될 것이다.  
이럴때 멀티쓰레드를 사용해 유저와 몬스터의 쓰레드가 다르다면  
내가 공격하는 동안 몬스터도 나를 향해 공격할 수 있을 것이다.

그러나 쓰레드간의 작업 전환에는 (context switching)  
다음에 실행해야할 위치 등 정보를 저장하고 읽어 오는 시간이 소요된다.  
그렇기 때문에 싱글 코어에서 단순히 CPU만을 사용하는 계산작업 같은 경우  
멀티쓰레드보다 오히려 싱글쓰레드가 효율적이다.

---

### Thread의 상태

쓰레드는 생성된 후부터 종료될 때까지 여러 상태를 가질 수 있다.  
그 상태는 다음과 같다.

| 상태           | 설명                                                                    |
| ------------- | --------------------------------------------------------------------- |
| NEW           | 쓰레드가 생성되고 아직 start()가 호출되지 않은 상태                              |
| RUNNABLE      | 실행 중 또는 실행 가능한 상태                                                |
| BLOCKED       | 동기화블럭에 의해서 일시정지된 상태 (lock이 풀릴 때까지 기다리는 상태)                |
| WAITING       | 쓰레드의 작업이 종료되지는 않았지만, 실행가능하지 않은 (unrunnable) 일시정지상태를 의미 |
| TIMED_WAITING | 일시정지시간이 지정된 경우를 의미                                             |
| TERMINATED    | 쓰레드의 작업이 종료된 상태                                                  |  

<br>

![error](/assets/images/whiteship-live-study/2021-01-19/state.png)

1) 쓰레드를 생성하고 start()를 호출하면 바로 실행되는 것이 아니라 실행 대기열에 저장되어 자신의 차례가 될 떄까지 기다려야 한다.  
   실행대기열은 큐 (queue) 와 같은 구조로 먼저 실행대기열에 들어온 쓰레드가 먼저 실행된다.
2) 실행대기상태에 있다가 자신의 차례가 되면 실행상태가 된다.
3) 주어진 실행시간이 다 되거나 yield() 를 만나면 다시 실행대기상태가 되고 다음 차례의 쓰레드가 실행상태가 된다.
4) 실행 중에 suspend(), sleep(), wait(), join(), I/O block에 의해 일시정지상태가 될 수 있다.  
   I/O block은 입출력작업에서 발생하는 지연상태를 말한다. 사용자의 입력을 기다리는 경우를 예로 들 수 있는데,  
   이런 경우 일시정지 상태에 있다가 사용자가 입력을 마치면 다시 실행대기상태가 된다.
5) 지정된 일시정지시간이 다되거나 (time-out), notify(), resume(), interrupt()가 호출되면  
   일시정지상태를 벗어나 다시 실행대기열에 저장되어 자신의 차례를 기다리게 된다.
6) 실행을 모두 마치거나 stop()이 호출되면 쓰레드는 소멸된다.

그림실력이 상당하다...ㅎ

---

### Thread의 우선순위

쓰레드 클래스는 priority (우선순위) 라는 변수/상수를 가지고 있다.

```java
    void setPriority(int newPriority);
    int getPriority();
    
    public static final int MIN_PRIORITY = 1;   // 최소 우선순위
    public static final int NORM_PRIORITY = 5;  // 보통 우선순위
    public static final int MAX_PRIORITY = 10;  // 최대 우선순위
```

쓰레드의 우선순위의 범위는 1~10이며 숫자가 높을수록 우선순위가 높다.  
main 쓰레드는 우선순위가 5이므로, main 메서드에서 생성하는 쓰레드의 우선순위는 자동적으로 5가 된다.
 
---

### Thread Group
쓰레드 그룹은 서로 관련된 쓰레드를 그룹으로 만들 수 있는 기능이다.  
쓰레드를 그룹에 넣을 수 있으며, 쓰레드 그룹을 다른 쓰레드 그룹에 넣는 것도 가능하다.

그러나 쓰레드 그룹이 도입된 이유는 보안상의 이유로,  
자신이 속한 그룹이나, 하위 그룹은 변경 할 수 있지만, 다른 쓰레드 그룹의 쓰레드는 변경할 수 없다.

main 쓰레드는 main 쓰레드 그룹에 속한다.  
그렇기에 쓰레드 그룹을 지정하지 않은 쓰레드는 모두 main 쓰레드 그룹에 속하게 된다.

```java
public class AlphabetThread extends Thread {

   public AlphabetThread(ThreadGroup grp1, String alphabetThread) {
      super(grp1, alphabetThread);
   }

}



public class Main {

   public static void main(String[] args) {
      ThreadGroup group = new ThreadGroup("Thread Group");

      AlphabetThread alphabetThread = new AlphabetThread(group, "alphabetThread");
      System.out.println(alphabetThread.getThreadGroup().getName());
   }

}

// 결과
Thread Group
```

쓰레드 그룹을 구현할 때는 그룹으로 만들 쓰레드의 생성자를 만들어 주면 된다.

---

### 동기화
동기화는 synchronization 이라고 부른다.  
싱글 쓰레드 프로세스의 경우 하나의 쓰레드만이 작업을 하기 때문에 문제가 없지만,  
멀티 쓰레드 프로세스의 경우라면 여러 쓰레드가 같은 자원을 공유해서 작업하기 때문에  
서로 작업에 영향이 미칠 수 있다.

만약 A 쓰레드와 B 쓰레드가 있다고 가정하자.
1. A 쓰레드가 작업을 시작한다.
2. 도중에 B 쓰레드에게 작업의 제어권이 넘어간다.
3. B 쓰레드는 A 쓰레드가 하려던 작업과는 다른 작업을 진행했다.
4. 제어권이 다시 A 쓰레드에게 돌아왔다.

A 쓰레드가 원래 의도했던 작업을 정상적으로 수행할 수 있을까?  
원래 의도와는 다른 결과를 얻게 될 것이다.

이를 방지하기 위해 lock이 필요하다.

공유데이터를 사용하는 영역을 지정하고, 이 공유 데이터가 가진 lock을 획득한 하나의 쓰레드만이 작업을 수행한다.  
그리고 작업을 마치면 lock을 반납해야한다.

이렇게 한 쓰레드가 진행중인 작업을 다른 쓰레드가 방해하지 못하도록 막는 것을 **synchronization** 이라고 한다.

```java
public class Main {

   public static void main(String[] args) {
      Runnable accountThread = new AccountThread();

      new Thread(accountThread).start();
      new Thread(accountThread).start();
   }

}


public class Account {
   private int balance = 1000;

   public int getBalance() {
      return balance;
   }

   public void withdraw(int money) {
      if (balance >= money) {
         try {
            Thread.sleep(1000);
         } catch(InterruptedException e) {
            e.printStackTrace();
         }
         balance -= money;
      }
   }
}


public class AccountThread extends Thread {

   Account account = new Account();

   @Override
   public void run() {
      while (account.getBalance() > 0) {
         int money = (int) (Math.random() * 3 + 1) * 100;
         account.withdraw(money);

         System.out.println("현재 잔고 : " + account.getBalance());
      }

   }
}
```

```text
현재 잔고 : 700
현재 잔고 : 700
현재 잔고 : 100
현재 잔고 : 100
현재 잔고 : 400
현재 잔고 : 100
현재 잔고 : 100
현재 잔고 : 100
현재 잔고 : 100
현재 잔고 : -100
현재 잔고 : 0 
```

분명히 잔고가 출금하려는 금액보다 큰 경우에만 출금 할 수 있도록 구현되어있는데,  
중간에 현재잔고 -100이 보인다.

이 이유는 한 쓰레드가 if문을 통과하고, 출금하려는 순간에 다른 쓰레드가 끼어들어서 먼저 출금을 했기 때문이다.  
이것이 실제 상황이라면 굉장히 큰 이슈가 될 것이다...

그럼 synchronized 키워드를 사용해보자.


```java
public class Main {

   public static void main(String[] args) {
      Runnable accountThread = new AccountThread();

      new Thread(accountThread).start();
      new Thread(accountThread).start();
   }

}


public class Account {
   private int balance = 1000;

   public int getBalance() {
      return balance;
   }

   public synchronized void withdraw(int money) {
      if (balance >= money) {
         try {
            Thread.sleep(1000);
         } catch(InterruptedException e) {
            e.printStackTrace();
         }
         balance -= money;
      }
   }
}


public class AccountThread extends Thread {

   Account account = new Account();

   @Override
   public void run() {
      while (account.getBalance() > 0) {
         int money = (int) (Math.random() * 3 + 1) * 100;
         account.withdraw(money);

         System.out.println("현재 잔고 : " + account.getBalance());
      }

   }
}
```

```text
현재 잔고 : 700
현재 잔고 : 500
현재 잔고 : 200
현재 잔고 : 100
현재 잔고 : 100
현재 잔고 : 100
현재 잔고 : 0
현재 잔고 : 0
```

그저 withdraw 메서드에 synchronized 키워드를 붙였을뿐인데  
현재 잔고가 음수가 되는 경우는 없다.

---

### 데드락

데드락은 한국말로 교착상태라고 한다.  
한정된 자원을 여러 곳에서 사용하려고 할때 발생된다.

```java
public class Main {
    public static Object Lock1 = new Object();
    public static Object Lock2 = new Object();

    public static void main(String args[]) {
        ThreadDemo1 T1 = new ThreadDemo1();
        ThreadDemo2 T2 = new ThreadDemo2();
        T1.start();
        T2.start();
    }

    private static class ThreadDemo1 extends Thread {
        public void run() {
            synchronized (Lock1) {

                System.out.println("Thread 1: Holding lock 1...");
                try { 
                    Thread.sleep(10); 
                }
                catch (InterruptedException e) {}
               
                System.out.println("Thread 1: Waiting for lock 2...");
                
                synchronized (Lock2) {
                    System.out.println("Thread 1: Holding lock 1 & 2...");
                }
            }
        }
    }
    
    private static class ThreadDemo2 extends Thread {
        public void run() {
            synchronized (Lock2) {
                System.out.println("Thread 2: Holding lock 2...");
                try { 
                    Thread.sleep(10); 
                }
                catch (InterruptedException e) {}
                
                System.out.println("Thread 2: Waiting for lock 1...");
                
                synchronized (Lock1) {
                    System.out.println("Thread 2: Holding lock 1 & 2...");
                }
            }
        }
    }
}
```

위 예제는 두 쓰레드가 서로에게 lock이 걸려 경쟁하는 상태가 된다.  
때문에 이 예제는 무한로딩에 빠지며, lock을 풀기 전까지는 대기상태에 머무른다.

<br>

다음은 데이터베이스에서 발생한 경우이다.  
자바 스터디이니 가볍게 살펴보기만 하자.  
(MySQL WorkBench를 두개 켜서 테스트를 했는데, 편의상 왼쪽, 오른쪽 워크벤치로 지칭하겠다.)

![error](/assets/images/whiteship-live-study/2021-01-19/deadlock01.png)

트랜잭션을 걸고 INSERT를 두 군데에서 하는 상황이다.
첫번째 인자값인 1은 게시물 번호로 PK이다.  
COMMIT을 하지 않고, 오른쪽 워크벤치에서도 INSERT를 해보겠다.

![error](/assets/images/whiteship-live-study/2021-01-19/deadlock02.png)

위와 같이 끊임없는 무한 로딩이 발생한다. (빨간색 원)   
이때 왼쪽 워크벤치에서 COMMIT을 하면 어떻게 될까?

![error](/assets/images/whiteship-live-study/2021-01-19/deadlock03.png)

![error](/assets/images/whiteship-live-study/2021-01-19/deadlock04.png)

오른쪽 워크벤치에서 무한로딩이 풀리고 X로 변한다.  
에러 메시지를 보면 Duplicate entry 즉 중복값이 있다는 것이다.  
당연히 PK가 중복됬기 때문에 발생한 것 이다.

INSERT된 값도 '첫 번째 게시물'이 된다.  
만약 COMMIT이 아닌 ROLLBACK을 했다면, 왼쪽 워크벤치에서의 값은 지워지고 오른쪽 워크벤치에 값인 '두 번째 게시물'이 추가 될 것이다.

![error](/assets/images/whiteship-live-study/2021-01-19/deadlock05.png)

이번엔 DELETE를 시도한다.  
오른쪽 워크벤치에서는 아직 COMMIT이 안됐기 때문에 또다시 무한 로딩이 걸린다.

![error](/assets/images/whiteship-live-study/2021-01-19/deadlock06.png)

오른쪽 워크벤치에서 COMMIT을 하면 데드락이 해결된다.

---

# 참고
[남궁성 - 자바의 정석](http://www.yes24.com/Product/Goods/24259565)  
[자바 데드락 예제 - https://www.tutorialspoint.com/java/java_thread_deadlock.html](https://www.tutorialspoint.com/java/java_thread_deadlock.html)

---