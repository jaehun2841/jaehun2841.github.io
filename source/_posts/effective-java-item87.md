---
title: Item 87. 커스텀 직렬화 형태를 고려해보라
catalog: true
Categories:
  - Effective-Java
tags:
  - Effective-Java
typora-root-url: effective-java-item87
typora-copy-images-to: effective-java-item87
date: 2019-03-17 15:21:53
subtitle:
header-img:
---

# 서론

개발 일정에 쫓기는 상황에서는 API 설계에 노력을 집중하는 편이 낫다.  
다음 릴리스에서 세부적인 기능을 제대로 구현하고 이번 릴리즈는 대충 동작만하게 하면 된다는 뜻이다.  
하지만 클래스가 Serializable을 구현하고 기본 직렬화 형태를 사용한다면 다음 릴리즈때 버리려 한 현재의 구현에 영원히 발이 묶이게 된다.  
(현재의 기본 직렬화 형태를 버릴 수 없게 되기 때문이다.)



# 먼저 고민해보고 괜찮다고 판단될 때만 기본 직렬화 형태를 사용하라

* 기본 직렬화 형태는 유연성,  성능, 정확성, 측면에서 신중히 고민한 후 합당할 때만 사용해야 한다.  
* 직접 설계하더라도 기본 직렬화 형태와 거의 같은 결과가 나올 경우에만 기본 형태를 써야 한다. 
* 기본 직렬화 형태는 그 객체를 루트로 하는 객체 그래프의 물리적 모습을 나름 효율적으로 인코딩한다.
* 객체가 포함한 데이터들과 그 객체에서부터 시작해 접근할 수 있는 모든 객체를 담아내며 객체들이 연결된 위상(topology)까지 기술한다.
* 하지만 이상적인 직렬화 형태는 **물리적인 모습과 독립된 논리적인 모습만을 표현해야 한다.**



# 객체의 물리적 표현과 논리적 내용이 같다면 기본 직렬화 형태라도 무방하다

```java
public class Name implements Serializable {
    
    /**
     * 성. null이 아니어야함
     * @serial
     */
    private final String lastName;
    
    /**
     * 이름. null이 아니어야 함.
     * @serial
     */
    private final String firstName;
    
    /**
     * 중간이름. 중간이름이 없다면 null.
     * @serial
     */
    private final String middleName;
}
```

* 기본 직렬화 형태가 적합하다고 결정했더라고 불변식 보장과 보안을 위해 readObject 메서드를 제공해야 할 때가 많다.
* Name의 3개의 필드는 private임에도 불구하고 문서화 주석이 달려있다. 
* 이 필드들은 결국 클래스의 직렬화 형태에 포함되는 공개 API에 속하며 공개 API는 모두 문서화 해야 하기 때문이다. 
* private 필드의 설명을 API 문서에 포함하라고 자바독에 알려주는 역할은 @serial태그가 한다.
* @serial 태그로 기술한 내용은 API 문서에서 직렬화 형태를 설명하는 특별한 페이지에 기록된다.



# 기본 직렬화 형태에 적합하지 않은 클래스

```java
public final class StringList implements Serializable {
    private int size = 0;
    private Entry head = null;
    
    private static class Entry implements Serializable {
        String data;
        Entry next;
        Entry previous;
    }
}
```

논리적으로는 이클래스는 일련의 문자열을 표현한다.  
물리적으로는 문자열을 이중 연결 리스트로 연결했다. 이 클래스에 기본 직렬화 형태를 사용하면 각 노드의 양방향 연결 정보를 포함해 모든 엔트리(Entry)를 철두철미하게 기록한다.



# 객체의 물리적 표현과 논리적 표현의 차이가 클 때 기본 직렬화 형태를 사용하는 경우 생기는 문제

## 공개 API가 현재의 내부 표현 방식에 영구히 묶인다.

* 앞의 예에서 private 클래스인 StringList.Entry가 공개 API가 되어버린다.
* 다음 릴리스에서 내부 표현 방식을 바꾸더라도 StringList 클래스는 여전히 연결 리스트로 표현된 입력도 처리할 수 있어야 한다.
* 즉 연결 리스트를 더 이상 사용하지 않더라도 관련 코드를 제거할 수 없다.



## 너무 많은 공간을 차지할 수 있다.

* 앞 예의 직렬화 형태는 연결 리스트의 모든 엔트리와 연결 정보까지 기록했지만, 엔트리와 연결 정보는 내부 구현에 해당하니 직렬화 형태에 포함할 가치가 없다.
* 이처럼 직렬화 형태가 너무 커져서 디스크에 저장하거나 네트워크로 전송하는 속도가 느려진다.



## 시간이 너무 많이 걸릴 수 있다.

* 직렬화 로직은 객체 그래프의 위상에 관한 정보가 없으니 그래프를 직접 순회해볼 수밖에 없다.
* 앞의 예제는 간단히 다음 참조를 따라가 보는 정도로 충분하다.



## 스택 오버플로를 일으킬 수 있다.

* 기본 직렬화 과정은 객체 그래프를 재귀 순회하는데 중간정도 크기의 객체 그래프에서도 스택 오버플로 에러가 날 수 있다.
* 그때그때 다른 시점에서 스택 오버플로가 날 수 있고, 어떤 플랫폼에서는 에러가 나지 않을 수도 있다.



# 합리적인 커스텀 직렬화 형태를 갖춘 StringList

```java
public final class StringList implements Serializable {
    private transient int size = 0;
    private transient Entry head = null;
    
    // 이제는 직렬화되지 않는다.
    private static class Entry {
        String data;
        Entry next;
        Entry previous;
    }
    
    // 지정한 문자열을 이 리스트에 추가한다.
    public final void add(String s) {...}
    
    /**
     * 이 {@code StringList} 인스턴스를 직렬화한다.
     * 
     * @serialData 이 리스트의 크기(포함된 문자열의 개수)를 기록한 후
     * ({@code int}), 이어서 모든 원소를(각각은 {@code String})
     * 순서대로 기록한다.
     */
    private void writeObject(ObjectOutputStream s) throws IOException {
     	//기본 직렬화를 수행한다.
        s.defaultWriteObject();
        s.writeInt(size);
        
        // 커스텀 역직렬화를 수행한다.
        // 모든 원소를 올바른 순서로 기록한다.
        for (Entry e = head; e != null; e = e.next)
            s.writeObject(e.data);
    }
    
    private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
        //기본 역직렬화를 수행한다.
        s.defaultReadObject();
        int numElements = s.readInt();
        
        // 커스텀 역직렬화 부분
        // 모든 원소를 읽어 이 리스트에 삽입한다.
        for(int i = 0; i < numElements; i++) {
            add((String) s.readObject());
        }
    }
}
```

* StringList의 필드 모두가 transient더라도 writeObject와 readObject는 각각 먼저 defaultWriteObject와 defaultReadObject를 호출한다.
* 클래스의 인스턴스가 모두 transient더라도 defaultWriteObject와 defaultReadObject를 호출해줘야 한다.  
  (향후 릴리즈에서 transient가 아닌 필드가 추가되더라도 상호 호환되기 때문이다)
* 신버전 인스턴스를 직렬화한 후 구버전으로 역직렬화하면 새로 추가된 필드들은 무시될 것이다.
* 구버전 readObject 메서드에서 defaultReadObject를 호출하지 않는다면 역직렬화할 때 StreamCorruptedException이 발생할 것이다.
* writeObject는 private 메서드임에도 문서화 주석이 달려 있다.  
  이 private 메서드는 직렬화 형태에 포함되는 공개 API에 속하며 공개 API는 모두 문서화 해야한다.
* 메서드에 달린 @serialData 태그는 자바독 유틸리티에게 이 내용을 직렬화 형태 페이지에 추가하도록 요청한다.

* 개선한 StringList는 원래버전의 절반정도의 공간을 차지하며 수행속도 또한 두 배 이상 빠르다.
* 개선한 StringList는 스택 오버플로 에러가 발생하지 않는다. (크기의 제한이 사라짐)
*  객체를 직렬화한 후 역직렬화하면 원래 객체를 그 불변식까지 포함해 제대로 복원해낸다는 점에서 정확하다 할 수 있다.
* 하지만 불변식이 세부 구현에 따라 달라지는 객체에서는 이 정확성마저 깨질 수 있다.



# 객체의 불변식이 깨지는 경우에는 직렬화를 주의해야한다.

해시 테이블을 예로 생각해보면 이해할 수 있다.  해시 테이블은 물리적으로는 key-value 엔트리를 담은 해시 버킷을 차례로 나열한 형태다.  
어떤 엔트리를 어떤 버킷에 담을지는 key에서 구한 hashcode가 결정하는데 **그 계산 방식은 구현에 따라 달라질 수 있다.**  
혹은 계산할 때마다 달라지기도 한다.  
따라서 해시테이블을 직렬화한 후 역직렬화하면 불변식이 심각하게 훼손된 객체들이 생겨날 수 있는 것이다.



# 객체의 논리적 상태와 무관한 필드라고 확신하면 transient 한정자를 생략하라

* 기본 직렬화를 수용하든 하지 않든 defaultWriteObject 메서드를 호출하면 transient로 선언하지 않은 모든 인스턴스 필드가 직렬화된다.
* 따라서 transient로 선언해되 되는 인스턴스 필드에는 모두 transient를 붙여야 한다.
* JVM을 실행할 때마다 값이 달라지는 필드도 transient를 붙여야 한다.
* 커스텀 직렬화 형태를 사용한다면 앞서의 StringList 처럼 대부분의(혹은 모든) 인스턴스 필드를 transient로 선언해야 한다.



# 동기화 메커니즘을 직렬화에도 적용해야 한다.

모든 메서드를 synchronized로 선언하여 스레드 안전하게 만든 객체에서 기본 직렬화를 사용하려면  
writeObject도 다음 코드 처럼 synchronized로 선언해야 한다.

```java
private synchronized void writeObject(ObjectOutputStream s) throws IOException {
    s.defaultWriteObject();
}
```

writeObject 메서드 안에서 동기화 하고 싶다면 클래스의 다른 부분에서 사용하는 락 순서를 똑같이 따라야 한다.  
그렇지 않으면 교착상태 (resource-ordering deadlock)에 빠질 수 있다.



# 직렬 버전 UID를 명시적으로 부여하자

어떤 직렬화 형태를 사용하든 직렬 가능 클래스에 모두 직렬 버전 UID를 명시적으로 부여하자.  
이렇게 하면 직렬 버전 UID가 일으키는 잠재적인 호환성 문제가 사라진다.  
성능도 조금 빨라지는데 직렬 버전 UID를 명시하지 않으면 런타임에 이 값을 생성하느라 복잡한 연산을 수행하기 때문이다.

```java
private static final long serialVersionUID = -232923283928929;
```

위와 같은 형태로 사용하면 된다.  
Intellij 에서는 alt+insert 단축키를 누르면 serialVersionUID를 자동으로 생성해 주는 메뉴가 있다.  
기존 클래스를 구버전으로 직렬화된 인스턴스와 호환성을 유지한 채 사용하고 싶다면  
기존 클래스를 구버전에서 사용한 자동 생성된 값을 그대로 사용해야 한다.  

## 주의할 점

**직렬버전 UID는 클래스의 명세가 변경되면 자동 생성된 값이 바뀌기 때문에 이부분도 주의해야 한다.**  
구버전과 호환이 되지 않아 역직렬화가 되지 않는다.  
기존 버전의 직렬화된 인스턴스를 역직렬화할 때 InvalidClassException이 던져질 것이다.  
구버전으로 직렬화된 인스턴스들과 호환성을 끊으려는 경우를 제외하고는 직렬 버전 UID를 절대 수정하면 안된다.



# 정리

* 클래스를 직렬화하기로 했다면 어떤 직렬화 형태를 사용할지 심사숙고 해야한다.
* 자바의 기본 직렬화 형태는 객체를 직렬화한 결과가 해당 객체의 논리적 표현에 부합할 때만 사용하고 그렇지 않으면 객체를 적절히 설명하는 커스텀 직렬화 형태를 고안해야 한다.
* 직렬화 형태도 공개 메서드를 설계할 때에 준하는 시간을 들여 설계 해야 한다.
* 한번 공개된 메서드는 향후 릴리즈에서 제거할 수 없듯이 직렬화 형태에 포함된 필드도 마음대로 제거할 수 없다.
* 직렬화 호환성을 유지하기 위해 영원히 지원해야 한다.
* 잘못된 직렬화 형태를 선택하면 해당 클래스의 복잡성과 성능에 영구히 부정적인 영향을 남긴다.



# 참고

* Effective Java 3rd Edition - Item 87. 커스텀 직렬화 형태를 고려해보라