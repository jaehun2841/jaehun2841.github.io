---
title: Item 78. 공유 중인 가변 데이터는 동기화해 사용하라
catalog: true
Categories:
  - Effective-Java
tags:
  - Effective-Java
typora-root-url: effective-java-item78
typora-copy-images-to: effective-java-item78
date: 2019-03-11 16:02:45
subtitle:
header-img:
---

# 서론

싱글 스레드 기반의 프로그램에서는 하나의 스레드가 하나의 객체를 사용하면 되기 때문에 동기화에 대한 걱정을 하지 않아도 되지만,  
멀티 스레드 기반의 환경에서는 여러개의 스레드가 하나의 객체를 공유해서 사용하는 경우가 있다.  
하나의 객체를 공유하며 사용하는 경우 불변 객체에 대해서는 동기화를 걱정할 필요가 없지만,  
스레드가 메서드를 실행하면서 변수의 데이터를 변경하는 경우 다른 스레드에서 동기화되지 않은 데이터를 읽을 수 있다.  
이런 경우에는 프로그래머가 기대한 결과와는 다른 결과를 초래할 수 있기 때문에 주의해야 한다.



# 동기화란?

동기화(Syncronized)란 멀티스레드 환경에서 하나의 메서드나 블록을 한번에 한 스레드씩 수행하도록 보장하는 것을 의미한다.  



## 동기화의 특징

* 한 객체가 일관된 상태를 가지고 생성되었을 때, 이 객체에 접근하는 메서드는 그 객체에 Lock을 건다.  
  (다른 스레드가 메서드를 실행할 때 실행되지 못하도록 Lock을 건다)
* Lock을 건 메서드는 객체의 상태를 확인하거나 필요하면 수정한다.
* 즉 일관된 하나의 상태 -> 다른 일관된 하나의 상태로 변화한다.
* 메서드 실행이 끝나면 Lock을 해제한다.
* 동기화를 제대로 사용하면 어떤 메서드도 이 객체의 상태가 일관되지 않은 순간을 볼 수 없다.
* **동기화 없이는** 한 스레드가 만든 변화를 다른 스레드에서 확인하지 못할 수 있다.  
  (동기화가 없다면, 너도나도 접근하는데 시점에 따라 일관된 상태가 아닐 수도 있기 때문)
* 언어 명세상 long과 double 외의 변수를 읽고 쓰는 동작은 원자적(atomic)이다.  
  (여러 스레드가 하나의 변수에 동기화 없이 접근해도 항상 어떤 스레드가 정상적으로 저장한 값을 온전히 읽어옴을 보장)
* 성능을 높이려면 원자적 데이터를 읽고 쓸 때는 동기화 하지 말아야겠다 **(위험한 발상)**  
  (필드를 읽을 때 항상 **수정이 완전히 반영된 값** 을 얻지만, **한 스레드가 저장한 값이 다른 스레드에도 보이는가?** 는 보장하지 않음)

# Java에서의 가변 데이터 동기화 방법

Java에서는 **synchronized** 키워드를 통해 동기화 처리를 할 수 있다.

* 가변데이터를 수정하거나 읽는 메서드를 동기화
* 가변 객체에 대한 동기화

```java
public synchronized void countup() {
}
```

```java
synchronized(list) {
}
```



# 한 스레드가 저장한 값이 다른 스레드에도 보이는가?

```java
public class StopThread {

    private static boolean stopRequested;

    public static void main(String[] args) throws InterruptedException {

        Thread backgroundThread = new Thread(() -> {
            int i = 0;
            while(!stopRequested) {
                i++;
            }
        });

        backgroundThread.start();

        TimeUnit.SECONDS.sleep(1);
        stopRequested = true;
    }
}
```

* 위의 코드는 대충 보면 1초뒤에 `stopRequested` 필드가 false로 바뀌면서 스레드 내의 while문이 종료될 것 처럼 보인다.
* 하지만 무한루프!!!
* 원인은 동기화에 있다.
* 동기화하지 않으면 메인 스레드에서 수정한 `stopRequested` 필드가 언제 false로 보일지 모른다.  
  (맨 마지막에 stopRequested가 false가 되면서 실제 원잣값은 false가 된다)
* 위의 문제를 해결하기 위해서는 `stopRequested` 필드에 대한 동기화 처리가 필요하다



## 동기화가 빠지는 경우 JVM에서 최적화를 수행할 수도 있다.

```java
//원래 코드
while(!stopRequested) {
    i++;
}

// 최적화한 코드
if(!stopRequested) {
    while(true) {
        i++;
    }
}
```

* JVM의 호이스팅 기법을 통해 최적화 할 수 있다.
* 이 결과 프로그램은 응답 불가 상태가 되어 더 이상 진전이 없다.



# 위의 코드를 동기화 해보자

```java
public class StopThread {

    private static boolean stopRequested;
    
    private static synchronized void requestStop() {
        stopRequested = true;
    }
    
    private static synchronized boolean stopRequested() {
        return stopRequested;
    }

    public static void main(String[] args) throws InterruptedException {

        Thread backgroundThread = new Thread(() -> {
            int i = 0;
            while(!stopRequested()) {
                i++;
            }
        });

        backgroundThread.start();

        TimeUnit.SECONDS.sleep(1);
        stopRequested();
    }
}
```

* 위와 같이 **stopRequested** 필드의 읽기/쓰기에 대한 동기화처리를 하면 위의 프로그램이 1초뒤에 종료된다.
* 읽기/쓰기 메서드 모두 동기화 처리를 하였음에 주목하자
* 쓰기 메서드만 동기화처리를 하고 읽기 메서드에는 동기화처리를 하지 않으면 동작을 보장할 수 없다.



# volatile 키워드

volatile 키워드의 의미는 `volatile 변수를 읽어 들일 때 CPU 캐시가 아니라 컴퓨터의 메인 메모리로 부터 읽어들인다.`   
즉 read 할 때도 CPU 캐시가 아닌 메인메모리에서 read하고, write할 때도 메인 메모리에 write를 수행 

![java-volatile](./java-volatile.png)



long, double을 제외한 기본타입은 **volitile** 키워드를 사용하면 동기화를 생략해도 된다.

```java
public class StopThread {

    private static volatile boolean stopRequested;

    public static void main(String[] args) throws InterruptedException {

        Thread backgroundThread = new Thread(() -> {
            int i = 0;
            while(!stopRequested) {
                i++;
            }
        });

        backgroundThread.start();

        TimeUnit.SECONDS.sleep(1);
        stopRequested = true;
    }
}
```

* volitile 예약어는 배타적 수행(한 블록을 한 스레드가 실행) 하는것과는 관계없다
* volatile 변수는 CPU캐시에서 값을 읽는게 아닌 메인 메모리에서 읽기 때문에 항상 최근에 기록된 값을 읽는다.  
  (그렇기 때문에 위의 프로그램이 1초 뒤에 종료됨)



# volatile 사용 시 주의할 점

```java
private static volatile int nextSerialNumber = 0;

public static int generateSerialNumber() {
    return nextSerialNumber++;
}
```

* 이 메서드는 호출 될 때 마다 1씩 증가하여 스레드에서 고유한 값을 반환할 의도로 만들어 졌다.
* 겉보기에는 int이기 때문에 원자적으로 접근할 수 있을 것 같다
* Volatile 키워드가 쓰여져있기 때문에 최신 값을 읽을 수 있을 것 같지만 제대로 된 고유한 값이 나오지 않는다.



## 원인은 nextSerialNumber++

원인은 nextSerialNumber++에 있었다.  
실제 이 코드는 1줄이지만 풀어쓰면 nextSerialNumber = nextSerialNumber + 1; 와 같은 형태이다.  
결국 nextSerialNumber 값을 한번 읽어와 +1 한다음에 다시 nextSerialNumber 변수에 저장하는 형태이다.  
만약 두번째 스레드가 nextSerialNumber + 1 연산이 이루어지는 시점을 비집고 들어온다면 1이 두번 리턴되는 형국이다.  
이런 오류를 **안전 실패(safety failure)** 이라고 한다.



## 문제 해결은 synchronized

```java
private static volatile int nextSerialNumber = 0;

public static synchronized int generateSerialNumber() {
    return nextSerialNumber++;
}
```

위와 같이 generateSerialNumber 메서드에 synchronized만 붙여주면 문제는 해결된다.  
동시에 호출해도 배타적으로 실행 (한번에 한 스레드만 실행) 되기 때문이다.  
만약 위 처럼 generateSerialNumber 메서드에 synchronized를 붙였다면 nextSerialNumber 변수에는 volatile을 제거해야 한다.  
만약 메서드를 더 견고하게 하려면 int 대신 long을 사용하는게 더 많은 수를 사용할 수 있다.



# long, double을 사용할 때는 더욱 더 주의하자

* Java.util.concurrent.atomic 패키지의 AtomicLong, AtomicDouble을 사용하는 것이 좋다.  
* 이 패키지는 락 없이도(lock-free) 스레드 안전한 프로그래밍을 지원하는 클래스들이 담겨 있다.
* volatile은 동기화 속성 중 통신에 대해서만 보장
* Java.util.concurrent.atomic 패키지는 원자성(배타적 실행) 까지 지원한다.

```java
private static final AtomicLong nextSerialNum = new AtomicLong();

public static long generateSerialNumber() {
    return nextSerialNum.getAndIncrement();
}
```



# 정리

* 동기화에 대한 문제를 피하는 가장 좋은 방법은 가변 데이터를 공유하지 않는 것이다.
* 가변데이터는 단일 스레드에서만 사용하는 것이 좋다
* 가변데이터를 단일 스레드에서만 사용한다면 문서에 남겨 유지보수 정책에서도 지켜지는것이 중요하다
* 멀티 스레드 환경에서 한 스레드가 데이터를 수정한 후에 다른 스레드에 공유할 때는 해당 객체에서 공유하는 부분만 동기화해도 된다.
* 클래스 초기화 과정에서 객체를 정적필드, volatile필드, final 필드, 혹은 보통의 락을 통해 접근하는 필드에 저장해도 된다.
* 여러 스레드가 가변 데이터를 공유한다면 그 데이터를 읽고 쓰는 메서드 모두에 반드시 synchronized 키워드를 붙인다.
* 배타적 실행 (한번에 한스레드) 동작이 필요없고, 스레드 간 최신데이터만 읽는 거로도 충분하면 가변 변수에 volatile 키워드만으로도 동기화가 가능하다



# 참고

* Effective Java 3rd Edition - Item 78. 공유 중인 가변 데이터는 동기화해 사용하라