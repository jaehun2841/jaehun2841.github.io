---
title: Item 35. ordinal 메서드 대신 인스턴스 필드를 사용하라
catalog: true
Categories:
  - Effective-Java
tags:
  - Effective-Java
typora-root-url: effective-java-item35
typora-copy-images-to: effective-java-item35
date: 2019-02-04 16:06:55
subtitle:
header-img:
---



# 서론

대부분의 열거 타입 상수는 자연스럽게 하나의 정숫값에 대응된다.  
모든 열거 타입은 해당 상수가 그 열거 타입에서 몇 번째 위치인지를 반환하는 ordinal이라는 메서드를 제공한다.



# ordinal을 잘못 사용한 예

```java
public enum Ensemble {
    SOLO, DUET, TRIO, QUARTET, QUINTET,
    SEXTET, SEPET, OCTET, NONET, DECTET;
    
    public int numberOfMusicians() { return ordinal() + 1; }
}
```

상수 선언 순서를 바꾸는 순간 numberOfMusicians 메서드의 기능을 상실하게 된다.  
상수 순서를 바꾸는 순간 ordinal 값이 바뀌기 때문에 상수의 의미와 실제 뮤지션 숫자와 일치하지 않는다.

가령, 똑같이 8명인 복4중주(double quartet)은 OCTET이 있기 때문에 추가할 수 도 없다.



# 인스턴스 필드를 사용하라

```java
public enum Ensemble {
    SOLO(1), DUET(2), TRIO(3), QUARTET(4), QUINTET(5),
    SEXTET(6), SEPET(7), OCTET(8), DOUBLE_QUARTET(8),
    NONET(9), DECTET(10), TRIPLE_QUARTET(12);
    
    private final int numberOfMusicians;
    Ensemble(int numberOfMusicians) {
        this.numberOfMusicians = numberOfMusicians;
    }
    public int numberOfMusicians() { return this.numberOfMusicians; }
}
```

이처럼 인스턴스 필드를 두어 값을 초기화 하여 사용하는 방법으로 사용하는 것이 좋다.



# ordinal은 언제 사용하나?

>The ordinal of this enumeration constant  
>(its position in the enum declaration, where the initial constant is assigned an ordinal of zero).  
>Most programmers will have no use for this field.  
>It is designed for use by sophisticated enum-based data structures,  
>such as {@link java.util.EnumSet} and {@link java.util.EnumMap}.

ordinal은 enum타입의 상수 값이다. (상수의 위치값)  
대부분의 프로그래머는 이 값이 필요는 없다.  
이 값은 EnumSet이나 EnumMap 같은 열거 타입 기반의 범용 자료구조에 사용 될 목적으로 만들어졌다.  
**결론: 사용하지 말자**



# 참고

* Effective Java 3rd Edition - Item 35. ordinal 메서드 대신 인스턴스 필드를 사용하라