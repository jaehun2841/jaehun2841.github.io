---
title: Item 7. 다 쓴 객체는 참조를 해제하라
catalog: true
Categories:
  - Effective-Java
tags:
  - Effective-Java
typora-root-url: effective-java-item7
typora-copy-images-to: effective-java-item7
date: 2019-01-05 20:21:26
subtitle:
header-img:
---

# 서론
Java의 장점 중 하나는 가비지 컬렉션을 지원하는 언어라는 점이다.  
C, C++ 처럼 개발자가 메모리를 직접 할당하고 해제하는 방식이 아니기 때문에 Java에서는 메모리 관리가 굉장히 편하다는 장점이 있다.

하지만 `아예 신경을 안써도 되는 것은 아니다.`  
가비지 컬렉션을 통해 소멸 대상이 되는 객체가 되기 위해서는 어떠한 reference 변수에서 가르키지 않아야 한다.  
다 쓴 객체에 대한 참조를 해제하지 않으면 가비지 컬렉션의 대상이 되지 않아 계속 메모리가 할당 되는 **메모리 누수** 현상이 발생 된다.

가비지 컬렉션을 지원하는 언어에서는 메모리 누수를 찾기가 까다롭다  
객체 참조(reference)를 하나 살려두면, 가비지 컬렉터는 그 객체 뿐만 아니라 그 객체 내에서 참조하고 있는 객체까지 회수 할 수 없다.

이로 인해 잠재적으로 성능에 악영향을 줄 수 있다.

# 가비지 컬렉션의 소멸 대상이 되기 위해서는...
## 직접 할당 해제
* 이 방법은 굉장히 단순하다.  
말 그대로 객체 참조 변수를 null로 초기화 한다.  
그렇게 되면 실제 heap 메모리에 존재하는 객체는 어떠한 참조(reference)도 가지지 않기 때문에
  `가비지 컬렉션의 소멸 대상`이 된다.
* 하지만 바람직한 방법은 아니다. 반드시 필요한 경우에만 사용 할 수 있도록 하는 것이 좋다.  
(소스코드가 드러워짐 -_-)
* 클래스 내에서 메모리를 관리 하는 객체(Stack 같은...)라면 이 방법을 통해 다 쓴 객체는 할당을 해제 하는 것이 옳다.

## Scope를 통한 자동 할당 해제
* 보통은 변수 선언`(대게 지역변수)`과 동시에 초기화를 사용한다.  
그 변수에 대한 scope가 종료되는 순간 reference가 해제되어 가비지 컬렉션의 대상이 된다.
* try~catch와 같은 구문에서는 catch 구문에서 try내에서 사용하는 변수를 참조하지 못하므로
  try~catch 변수 초기화를 하기 어렵다.  
  그렇게 때문에 finally 구문에서 변수에 대한 참조를 해제한다.

# 메모리 누수를 일으키는 주범
* 첫번째는 위에서 설명한 class내에서 instance에 대한 참조(reference)를 관리하는 객체이다.
* 두번째는 Map과 같은 캐시
* 세번째는 리스너(Listener) 혹은 콜백(Callback)


Map과 같은 캐시에 객체참조를 넣어두고 사용이 끝났는데도 초기화를 안시켜주는 경우 메모리 누수가 발생한다.  
엔트리가 살아있는 동안만 캐시를 사용하려면 `WeakHashMap`을 사용하자.  
WeakHashMap을 이해하려면 Java의 Reference를 좀 알아야 한다.

## Java Reference 
Java에는 4가지의 Reference가 있다.
* Strong Reference
  * 우리가 흔히 사용하는 reference
  * String str = new String("abc"); 와 같은 형태
  * Strong Reference는 GC의 대상이 되지 않는다. 
  * Strong Reference관계의 객체가 GC가 되기 위해선 null로 초기화해  
  객체에 대한 Reachability상태를 UnReachable 상태로 만들어 줘야 한다.

* Soft Reference
  * 객체의 Reachability가 Strongly Reachable 객체가 아닌 객체 중 Soft Reference만 있는 상태
  * SoftReference<Class> ref = new SoftReference<>(new String("abc"));와 같은 형태로 사용
  * Soft Reference는 대게 GC대상이 아니다가 `out of memory에러`가 나기 직전까지 가면  
  Soft Reference 관계에 있는 객체들은 GC대상이 된다.

* Weak Reference
  * 객체의 Reachability가 Strongly Reachable 객체가 아닌 객체 중 Soft Reference가 없고 Weak Reference만 있는 상태
  * WeakReference<Class> ref = new WeakReference<Class>(new String("abc")); 와 같은 형태로 사용
  * WeakReference는 GC가 발생 할 때마다 대상이 된다.

* Phantomly Reference
  * 객체의 Reachability가 Strongly Reachable 객체가 아닌 객체 중 Soft Reference와 Weak Referencerk 모두가 해당되지 않는 객체
  * finalize 되었지만 메모리가 아직 회수 되지 않은 객체
  * 아직 잘 이해가... 안됨

## WeakHashMap 
Weak Reference를 이용한 HashMap  
아래의 예제코드를 보자
```java
public class ClassWeakHashMap {
    public static class Referred {
        protected void finalize() {
            System.out.println("Good bye cruel world");
        }
    }

    /**
    * GC를 발생 시켜 메모리를 회수하는 코드
    * System.gc()가 잘 동작할지는 모르겠다.
    */
    public static void collect() throws InterruptedException {
        System.out.println("Suggesting collection");
        System.gc();
        System.out.println("Sleeping");
        Thread.sleep(5000);
    }

    public static void main(String args[]) throws InterruptedException {
        System.out.println("Creating weak references");

        // This is now a weak reference. 
        // The object will be collected only if no strong references. 
        Referred strong = new Referred(); //Strong Reference로 하나 추가

        //Weak Reference를 이용한 WeakHashMap에 엔트리를 추가하여
        //Weak Reference 추가
        Map<Referred, String> metadata = new WeakHashMap<Referred, String>();
        metadata.put(strong, "WeakHashMap's make my world go around");

        // Attempt to claim a suggested reference. 
        ClassWeakHashMap.collect();
        //여기서는 gc가 발생해도 GC대상이 아니게 된다.
        //strong이라는 변수를 통해 Strong Reference를 가지므로 GC 대상이 아니다.
        System.out.println("Still has metadata entry? " + (metadata.size() == 1));
        System.out.println("Removing reference");

        // The object may be collected. 
        //Strong Reference를 끊었다.
        strong = null;

        //여기서는 Weak Reference만 남아 있기 때문에 GC대상이 된다.
        ClassWeakHashMap.collect();
        System.out.println("Still has metadata entry? " + (metadata.size() == 1));
        System.out.println("Done");
    }
}
```

Weak Reference를 가지고 있으면 GC가 발생되기 전까지 객체에 접근이 가능하기 때문에 메모리 누수의 입장으로 볼 때 유리한 것 같다.  
한번 캐싱하고 사용하고 버리는 대상에 좋은 방법이다.

하지만 static한 Map을 사용하는 경우에는 비추이다.  
언제 GC가 일어날지 모를 뿐더러.. 갑자기 데이터가 사라져 자칫 하면 장애가 발생 할 수 있으니,
특별한 경우에만 WeakHashMap을 사용해야 한다.

## 리스너 혹은 콜백
리스너와 콜백은 root set에 대한 직접 참조가 아닌 객체에서 참조를 가지고 있다.  
그렇기 때문에 리스너와 콜백을 사용하는 객체가 unreachable 상태가 되지 않는 이상 메모리에서 GC대상이 되지 않는다.  
이 경우 weak reference를 이용하면 리스너와 콜백을 사용하고, GC 작동 시에 메모리 해제를 시킬 수 있어, 메모리 누수에 도움이 된다.



# 추가적으로..
메모리 누수는 겉으로 잘 드러나지 않아 수년 간 잠복하는 사례가 있다고 한다.  
이런 누수는 철저한 코드리뷰나 힙 프로파일링 도구를 통해 디버깅을 동원해야 발견할 수 있으므로,  
평소에 코드를 작성할 때 메모리 누수에 대한 부분을 신경을 써주는 것이 중요하다.

# 참고
* Effective Java 3rd Edition - Item 7. 다 쓴 객체는 참조를 해제하라
* https://d2.naver.com/helloworld/329631
* https://tourspace.tistory.com/42