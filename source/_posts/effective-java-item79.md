---
title: Item 79. 과도한 동기화는 피하라
catalog: true
Categories:
  - Effective-Java
tags:
  - Effective-Java
typora-root-url: effective-java-item79
typora-copy-images-to: effective-java-item79
date: 2019-03-12 00:28:29
subtitle:
header-img:
---

# 서론

과도한 동기화는 성능을 떨어뜨리고, 교착상태에 빠뜨리고, 심지어 예측할 수 없는 동작을 낳기도 한다.  
**응답 불가와 안전 실패를 피하려면 동기화 메서드나 동기화 블록 안에서는 제어를 절대로 클라이언트에 양도하면 안 된다.**  

* 동기화(synchronized) 된 코드 블럭 안에서는 재정의 가능한 메서드를 호출해선 안된다.
* 클라이언트가 넘겨준 함수객체를 호출해서도 안된다.
* 이런 메서드는 동기화도니 클래스 관점에서 외계인 메서드(alien method)라고 칭한다.  
  (무슨일을 할지 모르니, 이 메서드가 예외를 발생시키거나, 교착상태를 만들거나, 데이터를 훼손시킬 수 있다.)



# 외계인 메서드

```java
public class ObservableSet<E> extends ForwardingSet<E> {

    public ObservableSet(Set<E> set) {
        super(set);
    }

    private final List<SetObserver<E>> observers = new ArrayList<>();

    public void addObserver(SetObserver<E> observer) {
        synchronized (observers) {
            observers.add(observer);
        }
    }
    
    public boolean removeObserver(SetObserver<E> observer) {
        synchronized (observers) {
            return observers.remove(observer);
        }
    }
    
    private void notifyElementAdded(E element) {
        synchronized (observers) {
            for(SetObserver<E> observer : observers) {
                observer.added(this, element);
            }
        }
    }
    
    @Override
    public boolean add(E element) {
        boolean added = super.add(element);
        if(added) {
            notifyElementAdded(element);
        }
        return added;
    }

    @Override
    public boolean addAll(Collection<? extends E> c) {
        boolean result = false;
        for (E element : c) {
            result |= add(element); //notifyElementAdded를 호출
        }
        return result;
    }
}
```

* 관찰자들은 addObserver와 removeObserver 메서드를 호출해 구독을 신청하거나 해지한다.
* 두 경우 다음 콜백 인터페이스의 인스턴스를 메서드에 전달

```java
@FunctionalInterface
public interface SetObserver<E> {
    //ObservableSet에 원소가 더해지면 호출된다.
    void added(ObservableSet<E> set, E element);
}
```

* 이 인터페이스는 구조적으로 BiConsumer<ObservableSet<E>,  E>와 똑같다.
* 커스텀 함수형 인터페이스를 정의한 이유는 이름이 더 직관적이고 다중 콜백을 지원하도록 확장할 수 있기 때문이다.



# 외계인 메서드 - 예제1

```java
public static void main(String[] args) {
    ObservableSet<Integer> set = new ObservableSet<>(new HashSet<>());
    set.addObserver(new SetObserver<>() {
        public void added(ObservableSet<Integer> s, Integer e) {
            System.out.println(e);
            if(e == 23) {
                s.removeObserver(this);
            }
        }
    });
    
    for(int i = 0; i < 100; i++) {
        set.add(i);
    }
}
```

* s.removeObserver에 this를 넘겨야하는데 람다에서는 방법이 없으므로 익명클래스의 형태로 사용
* 이 프로그램은 0~23까지 출력한 후 Observer 자신을 구독해제 한 후 아무런 로그도 뜨지 않고 종료할 것 같다.
* 하지만 그렇지 않다. 프로그램은 0~23까지 출력한 후 `ConcurrentModificationException` 을 던진다.
* 왜냐하면 Observer의 added 메서드 호출이 일어난 시점이 notifyElementAdded가 Observer들의 리스트를 순회하는 도중이기 때문이다.
* added 메서드 -> ObservableSet.removeObserver를 호출 -> observers.remove 호출
* 리스트에서 원소를 제거하려는데 마침 지금은 이 원소를 순회하는 중 **(허용되지 않은 동작)**
* notifyElementAdded 메서드에서 수행하는 순회는 동기화 블록 안에 있으므로 동시 수정이 일어나지 않지만  
  정작 자신이 콜백을 거쳐 되돌아와 수정하는 것을 막진 못한다.



# 외계인 메서드 - 예제 2

```java
set.addObserver(new SetObserver<>() {
    public void added(ObservableSet<Integer> s, Integer e) {
        System.out.println(e);
        if(e == 23) {
            ExecutorService exec = Executors.newSingleThreadExecutor();
            try {
                exec.submit(() -> s.removeObserver(this)).get(); //여기서 lock이 걸려서 못들어감
               													 //하지만 메인 스레드는 너의 작업을 기다리고 있어
            } catch(ExecutionException | InterruptedException ex) {
                throw new AssertionError(ex);
            } finally {
                exec.shutdown();
            }
        }
    }
});
```

* 이 프로그램을 실행하면 에러는 나진 않지만 교착상태(Dead-lock)에 빠진다.
* 백그라운드 스레드가 s.removeObserver를 호출하면 Observer를 잠그려 시도하지만 락을 얻을 수 없다.  
  (메인스레드가 이미 락을 쥐고 있기 때문 - removeObserver는 synchronized 키워드가 달려있어서 실행 시 락이 걸린다.)
* 그와 동시에 메인 스레드는 백그라운드 스레드가 Observer를 제거하기만을 기다리는 중이다.



# 교착상태 해결방법

자바 언어의 락은 재진입(reentrant)을 허용하므로 교착상태에 빠지지는 않는다.  
재진입 가능 락은 객체 지향 멀티스레드 프로그램을 쉽게 구현 할 수 있도록 해준다.   
하지만 응답 불가(교착상태)가 될 상황을 안전 실패(데이터 훼손)으로 변모시킬 수도 있다.  

이런 경우 외계인 메서드 호출을 동기화 블록 바깥으로 옮기면 된다.

```java
private void notifyElementAdded(E element) {
    List<SetObserver<E>> snapshot = null;
    synchronized(observers) {
        snapshot = new ArrayList<>(observers);
    }
    for (SetObserver<E> observer : snapshot) {
        observer.added(this, element); //외계인 메서드를 동기화 블록 바깥으로 옮겼다.
    }
}
```



# CopyOnWriteArrayList

외계인 메서드 호출을 동기화 블록 바깥으로 옮기는 것 보다 더 나은 방법은 java.util.concurrent 패키지의 CopyOnWriteArrayList를 사용하는 것이 좋다.  
내부를 변경하는 작업은 항상 깨끗한 복사본을 만들어 수행한다.  
내부의 배열은 절대 수정되지 않아 락이 없어 빠르다.  
다른 용도로 쓰인다면 매번 복사해서 느리겠지만, 수정할 일은 드물고 순회만 빈번히 일어나는 Observer 리스트 용으로는 딱이다.

```java
private final List<SetObserser<E>> observers = new CopyOnWriteArrayList<>();

public void addObserver(SetObserver<E> observer) {
    observers.add(observer);
}

public boolean removeObserver(SetObserver<E> observer) {
    return observers.remove(observer);
}

public void notifyElementAdded(E element) {
    for (SetObserver<E> observer : observers) {
         observers.added(this, element);
    }
}
```



# 동기화의 성능

자바의 동기화 비용은 빠르게 낮아져 왔지만, 과도한 동기화를 피하는일은 오히려 과거 어느 때보다 중요하다.  
멀티코어가 일반화된 오늘날 과도한 동기화가 초래하는 진짜 비용은 락을 얻는데 드는 CPU 시간이 아니다.  
서로 스레드끼리 경쟁하는 Race Condition에 낭비가 발생한다.  

* 병렬로 실행할 기회를 잃는다.
* 모든 코어가 메모리를 일관되게 보기위한 지연시간이 진짜 비용
* 가상머신의 코드최적화를 제한하는 점도 숨은 비용



## 가변 클래스를 작성하는 경우 동기화에 대해 고려할 점

1. 동기화를 전혀 하지 말고 가변 클래스를 동시에 사용해야하는 클래스가 외부에서 동기화하자
2. 동기화를 내부에서 수행해 스레드 안전한 클래스로 만들자.  
   (단 클라이언트가 외부에서 객체 전체에 락을 거는 것보다 동시성을 월등히 개선할 수 있을 때만 두번째 방법을 쓴다)



# 정리

* 기본 규칙은 동기화 영역에서 가능한 한 일을 적게하는 것이다.  
  (락을 얻고 공유 데이터를 검사하고, 필요하면 수정하고, 락을 놓는다.)
* 오래 걸리는 작업이라면 동기화 영역 밖으로 옮기는 방법을 찾아보자.
* 여러 스레드가 호출할 가능성이 있는 메서드가 정적 필드를 수정한다면 그 필드를 사용하기 전에 반드시 동기화 해야 한다.
* 교착상태와 데이터 훼손을 피하려면 동기화 영역 안에서 외계인 메서드를 절대 호출하지 말자
* 동기화 영역 안에서 작업은 최소한으로 줄이자
* 가변 클래스를 설계할 때는 스스로 동기화해야 할지 고민하자
* 지금은 과도한 동기화를 피하는게 제일 중요하다
* 합당한 이유가 있을때만 내부에서 동기화하고 동기화 여부를 문서에 남기자.  
  (웬만하면 외부에서 동기화를 하자)



# 참고

* Effective Java 3rd Edition - Item 79. 과도한 동기화는 피하라