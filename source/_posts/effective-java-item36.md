---
title: Item 36. 비트 필드 대신 EnumSet을 사용하라
catalog: true
Categories:
  - Effective-Java
tags:
  - Effective-Java
typora-root-url: effective-java-item36
typora-copy-images-to: effective-java-item36
date: 2019-02-04 16:27:40
subtitle:
header-img:
---



# 서론

 열거한 값들이 주로 단독이 아닌 집합으로 사용 될 경우, 예전에는 각 상수에 서로 다른 2의 거듭제곱 값을 할당한 정수 열거 패턴을 사용해왔다.

```java
public class Text {
    public static final int STYLE_BOLD =          1 << 0; //1
    public static final int STYLE_ITALIC =        1 << 1; //2
    public static final int STYLE_UNDERLINE =     1 << 2; //4
    public static final int STYLE_STRIKETHROUGH = 1 << 3; //8
    
    //매개 변수 styles는 0개 이상의 STYLE_ 상수를 비트별 OR 한 값이다.
    public void applyStyle(int styles) {...}
}
```



다음과 같은 식으로 비트별 OR을 사용해 여러 상수를 하나의 집합으로 모을 수 있으며,   
이렇게 만들어진 집합을 비트 필드(bit field)라고 한다.

```java
text.applyStyles(STYLE_BOLD | STYLE_ITALIC); // 1 | 2 => 3
```



하지만 비트 필드는 열거 상수의 단점을 그대로 지닌다.

* 컴파일 되면 값이 새겨진다 (무슨 의미인지 모름)
* 비트 필드 값이 그대로 출력되면 단순한 정수 열거 상수보다 해석하기 어렵다.  
  (어떤 값이 OR연산되서 나온값인지 알기 어렵다.)
* 최대 몇 비트가 필요한지 API작성 시 미리 예측이 필요하다.



# 비트 필드 대신 EnumSet을 사용하라

* java.util 패키지의 EnumSet 클래스는 열거 타입 상수의 값으로 구성된 집합을 효과적으로 표현해준다.  
* Set 인터페이스를 완벽히 구현하며, 타입 안전하고 다른 Set 구현체와도 함께 사용할 수 있다.  
* EnumSet의 내부는 비트 벡터로 구현되었다.
* 원소가 64개 이하인 경우에는 EnumSet 전체를 long 변수 하나로 표현하여 비트 필드에 비견되는 성능을 보여준다.
* removeAll과 retailAll과 같은 대량 작업은 비트를 효율적으로 처리 할 수 있는 산술 연산을 사용하였다.  
  (RegularEnumSet과 JumboEnumSet 클래스를 보면 비트연산을 많이 한다.)



# 비트필드를 대체하는 EnumSet 예제

```java
public class Text {
    public enum Style { BOLD, ITALIC, UNDERLINE, STRIKETHROUGH }
    
    // 어떤 Set을 넘겨도 되나 EnumSet이 가장 좋다.
    // 이왕이면 Set Interface로 받는게 좋은 습관이다.
    public void applyStyles(Set<Style> styles) {...}
}
```

```java
text.applyStyles(EnumSet.of(Style.BOLD, Style.ITALIC));
```



# 정리

* 열거할 수 있는 타입을 한데 모아 집합 형태로 사용한다고 해도, 비트 필드를 사용할 이유는 없다.
* EnumSet 클래스가 비트 필드 수준의 명료함과 성능을 제공한다.
* EnumSet의 유일한 단점은 불변 객체를 생성할 수 없는 점이다.



# 참고
* Effective Java 3rd Edition - Item 36. 비트 필드 대신 EnumSet을 사용하라
