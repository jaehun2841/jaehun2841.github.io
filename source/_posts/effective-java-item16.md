---
title: Item 16. Public 클래스에서는 Public 필드가 아닌 접근자 메서드를 사용하라
catalog: true
Categories:
  - Effective-Java
tags:
  - Effective-Java
typora-root-url: effective-java-item16
typora-copy-images-to: effective-java-item16
date: 2019-01-20 17:53:54
subtitle:
header-img:
---

# 퇴보한 클래스
```java
class Point {
  public double x;
  public double y;
}
```
이런 클래스는 데이터 필드에 직접 접근하여 수정이 가능하다. 따라서 캡슐화의 이점을 제공하지 못한다. 
* API를 수정하지 않고는 내부 표현을 바꿀 수 없다.
* 불변식을 보장할 수 없다.
* 외부에서 필드에 접근할 때 부수 작업을 수행 할 수도 없다.
  (예를들면, x값 조회 시, Comma case로 리턴하는 식의?)

# 흔하게 만드는 캡슐레이션
```java
class Point {
  private double x;
  private double y;

  public Point (double x, double y) {
    this.x = x;
    this.y = y;
  }

  public double getX() { return x; }
  public double getY() { return y; }

  public void setX(double x) { this.x = x; }
  public void setY(double y) { this.y = y; }
}
```
* public 클래스라면 반드시 이정도의 접근자와 수정자를 만드는 정도는 해야된다.  
* 클래스 내부 표현 방식을 언제든지 바꿀 수 있는 유연성을 제공해야 한다.
* package-private 클래스 혹은 private 중첩 클래스라면 데이터 필드를 노출한다해도 문제가 없다.
  * 같은 패키지안에서 특정이유 때문에 사용하던가, 톱레벨 클래스에서만 접근 할 것이기 때문이다.

# 요약
* public 클래스는 절대 가변 필드를 노출해선 안된다.
* 불변 필드라면 노출해도 덜위험 하지만 그래도 하지 말아라.
* package-private 클래스, 톱 레벨 클래스에서는 노출하는 편이 나을 수도 있다. (글쎄..?)

# 참고
* Effective Java 3rd Edition - Item 16. Public 클래스에서는 Public 필드가 아닌 접근자 메서드를 사용하라
