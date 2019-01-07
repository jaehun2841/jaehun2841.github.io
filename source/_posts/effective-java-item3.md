---
title: Item 3. private 생성자나 열거 타입으로 싱글턴임을 보증하라
catalog: true
Categories:
  - Effective-Java
tags:
  - Effective-Java
typora-root-url: effective-java-item3
typora-copy-images-to: effective-java-item3
date: 2019-01-07 19:34:19
subtitle:
header-img:
---

# 싱글턴이란?
싱글턴 객체란 Application내에서 단 1개의 인스턴스만 생성할 수 있는 클래스를 의미한다.  
시스템에서 유일성을 보장하기 위해 1개만 만들어야 하는 경우도 있고, 시스템 구조 상 1개 이상이 되면 개념적으로 맞지 않는 상황도 있기 때문이다.

# 싱글턴을 만드는 방법
## private 생성자 + static 객체
private 생성자를 통해 내부에서만 객체를 생성 할 수 있도록 하고,
public static final 키워드를 이용해 static 변수로 1개의 인스턴스만 제공하는 방식이다.

```java
public class Elvis {
  public static final Elvis INSTANCE = new Elvis();
  private Elvis() {}
}
```

사용자가 Elvis 객체를 클라이언트에서 사용하기 위해서는  
외부로 노출 된 생성자가 없기 때문에 Elvis.INSTANCE의 형태로 사용해야만 한다.  
최초로 INSTANCE 변수가 초기화 될 때 private 생성자를 통해 단 한번 인스턴스가 생성되게 된다.
> 하지만, Java의 Reflection기능을 이용하여, AccessibleObject.setAccessible(true)를 이용하면 private 생성자를 호출 할 수 있다.  
(이런 부분은 논외로 한다.)  여차하면 두번째 호출 부터 Exception을 발생 시켜 싱글턴을 보장할 수 있도록 방어로직을 심을 수 있다.


## 정적 팩터리 메서드
위의 private 생성자 + static 객체에서 조금 진화된 형태가 클래스에서 정적 팩터리 메서드를 제공하는 case이다.

```java
public class Elvis {
  private static final Elvis INSTANCE = new Elvis();
  private Elvis() {}
  public static Elvis getInstance() {
    return INSTANCE;
  }
}
```
1. 지금은 싱글턴 객체를 리턴하는 정적 메서드이지만, 향후 필요에 따라 변경될 수 있는 확장성을 가지고 있다.
특정 파라미터나, 특정 스레드에는 다른 인스턴스를 리턴한다던지에 대해 확장과 변경에 열려 있는 장점이 있다.

2. 원한다면 정적 팩터리를 제네릭 싱글턴 팩터리로 만들 수 있다.
3. 정적 팩터리 메서드의 참조를 공급자(Supplier)로 만들 수 있다.


```java
public class Elvis {
  private static final Elvis INSTANCE = new Elvis();
  private Elvis() {}
  public static Elvis getInstance() {
    return INSTANCE;
  }

  public static Supplier<Elvis> get() {
    return () -> INSTANCE;
  }
}
```

## 열거 타입(Enum)을 이용한 싱글턴 객체 생성
```java
public enum Elvis {
  INSTANCE;
}
```
1번째 예시와 비슷하지만 가장 안전하고 좋은 방법이다.
복잡한 직렬화 상황이나, 리플렉션 공격에도 안전하다.  
단, 만들려는 싱글턴이 Enum이외의 클래스를 상속해야 한다면 이 방법은 사용할 수 없다.  
(열거 타입이 다른 인터페이스를 구현하도록 하는 건 가능.)


# 싱글턴 객체의 직렬화
싱글턴 클래스를 직렬화 하기 위해서는 Serializable을 구현한다고 선언하는 것 만으로는 부족하다.  
모든 인스턴스 필드에 transient 예약어를 붙여 직렬화를 막은 다음 readResolve 메서드를 제공해야 한다.  
이렇게 하지 않으면 역직렬화 시점에 새로운 인스턴스가 생성 된다.

```java

public class Elvis implements Serializable {
  private static final Elvis INSTANCE = new Elvis();
  private Elvis() {}
  public static Elvis getInstance() {
    return INSTANCE;
  }

  //역직렬화가 되어 새로운 인스턴스가 생성되더라도, 
  //클래스간 공유 변수인 static 변수를 이용하면 싱글턴을 보장 할 수 있다.
  // 새로운 인스턴스는 GC에 의해 UnReachable 형태로 판별되어 제거된다.
  public Object readResolve() {
    return INSTANCE;
  }
}
```

# 참고
* Effective Java 3rd Edition - Item 3. private 생성자나 열거 타입으로 싱글턴임을 보증하라
