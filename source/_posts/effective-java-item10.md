---
title: Item 10. equals는 일반 규약을 지켜 재정의하라
catalog: true
Categories:
  - Effective-Java
tags:
  - Effective-Java
typora-root-url: effective-java-item10
typora-copy-images-to: effective-java-item10
date: 2019-01-10 21:14:20
subtitle:
header-img:
---

# 서론
equals 메서드는 Object 클래스에 구현된 메서드로 객체 내의 정보들에 대한 동등성(equality) 비교를 목적으로 하는 메서드이다.  
equals 메서드를 잘못 작성하게 되면 의도하지 않는 결과들이 초래되므로 웬만하면 변경하지 않는 것이 좋다.

# equals를 재정의 하지 않아도 되는 경우
* 각 인스턴스가 본질적으로 고유 한 경우 - 값이 아닌 동작을 표현하는 클래스의 경우 (Thread가 좋은 예이다.)
* 인스턴스의 논리적 동치성 (Logical Equality)를 검사할 일이 없는 경우 - `java.utils.regex.Pattern`의 equals는 내부의 정규표현식이 같은지를 검사하는 메서드이다.
* 상위 클래스에서 재정의한 equals가 하위 클래스에서도 적용 되는 경우 - Set, Map, List의 경우 Abstract(Type)의 equals를 쓴다.
* 클래스가 private이거나, package-private여서 equals를 호출할 일이 없는 경우
* 싱글턴을 보장하는 클래스(인스턴스 통제 클래스, Enum (열거타입)) 인 경우 - 객체 간 동등성, 동일성이 보장된다.

# equals를 재정의 하는 경우 지켜야 할 규약
equals를 재정의 해야 하는 경우는 객체 동일성(식별성(Object Identity))를 확인해야 하는 경우가 아니라  
논리적 동치성(동등성(Logical equality))를 비교 하도록 재정의 되지 않았을 경우이다.

## 반사성(reflexivity)
**null이 아닌 모든 참조 값 x에 대해 x.equals(x)를 만족해야한다.**  
단순히 말하면 객체는 자기 자신과 비교했을 때 같아야 한다는 뜻이다.  
이 조건을 만족하지 않는 예를 찾기가 더 어렵다.  
만약 x.equals(x)가 성립하지 않는 객체라면, 컬렉션에서 contain 메서드를 사용하는 경우 방금 넣은 객체도 찾을 수 없을 것이다.

## 대칭성 (symmetry)
**null이 아닌 모든 참조 값 x, y에 대해 x.equals(y)가 true이면, y.equals(x)가 true를 만족해야 한다.**  

예시 코드를 보면
```java
public final class CaseInsensitiveString {
  private final String s;

  public CaseInsensitiveString(String s) {
    this.s = Objects.requireNonNull(s);
  }

  @Override
  public boolean equals(Object o) {
    if(o instanceof CaseInsensitiveString) {
      return s.equalsIgnoreCase(((CaseInsensitiveString) o).s);
    }

    if(o instanceof String) { //한 방향으로만 작동!!
      return s.equalsIgnoreCase((String) o);
    }
    return false;
  }
}
```

위의 클래스를 기반으로 
```java
CaseInsensitiveString caseInsensitiveString = new CaseInsensitiveString("Test");
String test = "test";
System.out.println(caseInsensitiveString.equals(test)); //true
System.out.println(test.equals(caseInsensitiveString)); //false
```
위의 코드를 실행하게 되면, x.equals(y)가 true일 때, y.equals(x)가 false이므로 `대칭성`이 깨지는 코드가 된다.  
String 클래스에서는 CaseInsensitiveString의 존재를 모르기 때문에 false가 날 수 밖에 없는 상황이다.

```java
List<CaseInsensitiveString> list = new ArrayList<>();
list.add(new CaseInsensitiveString("Test"));
System.out.println(list.contain("test")); //false or true
```
위의 예제의 경우는 JDK버전에 따라 다를 수 있다. x.equals(y)로 비교할 수 도 있고, y.equals(x)로 비교될 수 있기 때문이다.

위의 내용을 수정한다면, String과의 비교는 포기해야 한다.  
같은 CaseInsensitiveString 타입인 경우에만 비교하도록 한다.
```java
@Override
public boolean equals(Object o) {
  return o instanceof CaseInsensitiveString && ((CaseInsensitiveString) o).s.equalsIgnoreCase(s);
}
```

## 추이성 (transitivity)
**null이 아닌 모든 참조 값 x, y, z에 대해 x.equals(y)가 true이고, y.equals(z)가 true이면 x.equals(z)도 true가 되야 한다는 조건이다.**

Point클래스와 ColorPoint 클래스를 가지고 예시를 들어보겠다.  
(ColorPoint는 Point클래스를 확장(extends)한 클래스이다.)
```java
ColorPoint a = new ColorPoint(1, 2, Color.RED);
Point b = new Point(1, 2);
ColorPoint c = new ColorPoint(1, 2, Color.BLUE);
```
위와 같은 인스턴스 a, b, c가 있다.
이 때, a.equals(b)와 b.equals(c) 일 때, a.equals(c)
가 되는 과정을 살펴 보자

```java
class Point {
  private final int x;
  private final int y;

  public Point(int x, int y) {
    this.x = x;
    this.y = y;
  }

  @Override
  public boolean equals(Object o) {
    if(!(o instanceof Point)) return false;
    Point p = (Point) o;
    return this.x == p.x && this.y == p.y;
  }
}
```

### 대칭성이 위배되는 case
```java
class ColorPoint extends Point {
  
  private final Color color;

  @Override
  public boolean equals(Object o) {
    if(!(o instanceof ColorPoint)) return false;
    
    return super.equals(o) && this.color == ((ColorPoint) o).color;
  }
}
```

위 처럼 ColorPoint 클래스의 equals 메서드를 재정의 했다면... 
```java
ColorPoint a = new ColorPoint(1, 2, Color.RED);
Point b = new Point(1, 2);

System.out.println(a.equals(b)); //false
System.out.println(b.equals(a)); //true
```
1. `a.equals(b)`를 보면 a는 ColorPoint이기 떄문에 ColorPoint 클래스에서 재정의 된 equals 메서드를 타게 된다.  
이렇게 되면 첫번째 if 조건에서 걸리게 된다. b는 Point이지만 ColorPoint는 아니기 떄문이다.  
따라서 `a.equals(b)`는 `false`가 된다.

2. `b.equals(a)`를 보면 b는 Point클래스이기 떄문에 Point클래스의 equals메서드를 타게 된다.  
이렇게 되면 ColorPoint는 Point클래스를 상속하고 있기 때문에 첫번째 if조건을 통과하게 되고,  
int x, int y값을 기준으로만 비교하기 떄문에 값이 참이 된다.  
따라서 `b.equals(a)`는 `true`가 된다.

위의 예시는 equals 정의 규약 중 하나인 대칭성을 위반하고 있으므로 다시 작성해야 한다.

### 추이성이 위반되는 case
```java
class ColorPoint extends Point {
  
  private final Color color;

  @Override
  public boolean equals(Object o) {
    if(!(o instanceof Point)) return false;

    //o가 일반 Point이면 색상을 무시햐고 x,y정보만 비교한다.
    if(!(o instanceof ColorPoint)) return o.equals(this);
    
    //o가 ColorPoint이면 색상까지 비교한다.
    return super.equals(o) && this.color == ((ColorPoint) o).color;
  }
}
```
위 처럼 ColorPoint 클래스의 equals메서드를 다시 재정의 했다면..  
```java
ColorPoint a = new ColorPoint(1, 2, Color.RED);
Point b = new Point(1, 2);
ColorPoint c = new ColorPoint(1, 2, Color.BLUE);

System.out.println(a.equals(b)); //true
System.out.println(b.equals(c)); //true
System.out.println(a.equals(c)); //false
```
1. `a.equals(b)`를 보면 a는 ColorPoint클래스의 인스턴스이므로 재정의된 equals 메서드를 실행하게 된다.  
b가 Point인 경우이기 때문에 2번쨰 if 절을 타게 되어 Point의 x, y정보만 비교하게 되므로 결과는 `true`이다.
2. `b.equals(c)`를 보면 b는 Point클래스의 인스턴스이므로 Point클래스의 equals 메서드를 실행하게 된다.  
Point의 x, y정보만 비교하게 되므로 결과는 `true`이다.
3. `a.equals(c)`를 보면 a는 ColorPoint클래스의 인스턴스이므로 재정의된 equals 메서드를 실행하게 된다.  
하지만, c가 ColorPoint클래스의 인스턴스이므로 x, y값 뿐만아니라 color까지 비교하게되어 결과는 `false`이다.

이렇게 a.equals(b)는 `true`를 만족하고 b.equals(c)는 `true`를 만족하지만 a.equals(c)는 `false`가 되므로  
위의 코드는 equals 정의 규약 중 `추이성`을 위반하는 코드가 된다.

### 무한 재귀 (Infinite Recursion)이 발생하는 case
```java
class SmellPoint extends Point {
  
  private final Smell smell;

  @Override
  public boolean equals(Object o) {
    if(!(o instanceof Point)) return false;

    //o가 일반 Point이면 색상을 무시햐고 x,y정보만 비교한다.
    if(!(o instanceof SmellPoint)) return o.equals(this);
    
    //o가 ColorPoint이면 색상까지 비교한다.
    return super.equals(o) && this.smell == ((SmellPoint) o).smell;
  }
}
```
위 처럼 Point를 상속받는 SmellPoint라는 클래스도 추가되었다고 가정해보자

```java
Point cp = new ColorPoint(1, 2, Color.RED);
Point sp = new SmellPoint(1, 2, Smell.SWEET);

System.out.println(cp.equals(sp));  //?
```
위와 같이 ColorPoint와 SmellPoint에 대해 equals비교를 한다고 하면 어떻게 될까??  
이렇게 되면 두번째 if절에서 무한재귀(Infinite Recursion)이 발생하게 된다.  

무슨 의미인지 자세히 설명하면, cp는 ColorPoint 클래스의 인스턴스이기 때문에  
`cp.equals(sp)` 코드는 ColorPoint 클래스의 재정의 된 equals 메서드를 타게 된다.  
이렇게 되면 두번째 if에서 걸리게 된다. 왜냐하면 o는 **SmellPoint** 타입이기 때문에 `!(o instanceof ColorPoint)` 이 조건식이 true가 된다.  
그렇기 때문에 o.equals(this)가 실행되게 되면 결과적으로 o는 SmellPoint 클래스의 인스턴스이기 때문에  
SmellPoint 클래스의 재정의된 equals메서드를 타게 된다.

다시 SmellPoint클래스의 입장에서 보게되면 두번째 if에서 걸리게 된다.  
여기서 o는 ColorPoint타입이기 때문에 `!(o instanceof ColorPoint)` 이 조건식이 true가 된다.  
그렇기 때문에 o.equals(this)가 실행되게 되면 결과적으로 다시 ColorPoint 클래스의 재정의된 equals메서드를 타게 된다.

이런 작업이 계속 재귀적으로 호출 되면서, 결국은 StackOverflowError를 내며 프로그램이 죽는 참사를 맞이 하게 될 것이다.

### 리스코프 치환 원칙 (SOLID)
SOLID원칙 중 3번째인 리스코프 치횐 원칙이란?
>자식 클래스는 최소한 자신의 부모 클래스에서 가능한 행위는 수행할 수 있어야 한다.

쉽게 말해서, 자식클래스에서 부모클래스의 기능을 수행하지 못하는 건 리스코프 치환원칙에 위배된다고 볼 수 있다.

위에서 equals 재정의에 실패해서 다시 또 변경하였다.
```java
class Point {
  
  private final int x;
  private final int y;

  private static final Set<Point> unitCircle = Set.of(new Point(0, -1),
   new Point(0, 1),
   new Point(-1, 0),
   new Point(1, 0)
  );

  public static boolean onUnitCircle(Point p) {
    return unitCircle.contains(p);
  }

  @Override
  public boolean equals(Object o) {
    if(o == null || o.getClass() != this.getClass()) {
      return false;
    }

    Point p = (Point) o;
    return this.x == p.x && this.y = p.y;
  }
}
```

이번에는 주어진 점이 반지름이 1인 원안에 포함되는지에 대한 여부를 판단하는 로직이다.

```java 
ColorPoint cp = new ColorPoint(1, 0, Color.RED);
System.out.println(Point.onUnitCircle(cp)); //false
```

ColorPoint는 Point를 상속한 클래스이다.  
실제 oint.onUnitCircle 메서드의 contains 메서드에서 Set내의 element들에 대한 equals 비교가 일어나게 된다.  
하지만 equals 메서드 첫번째 if문에서 걸리게 된다.  
ColorPoint 객체가 파라미터로 전달되어 null은 아니지만,  
두번째 조건식인 o.getClass()에서 `ColorPoint.class`가 도출되고 this.getClass()에서는 `Point.class`가 도출되게 된다.  

위 조건식 때문에 ColorPoint 클래스는 Point클래스로서 equals메서드 내에서 활용되지 못했기 때문에  
위 코드는 리스코프 치환원칙에 위배되는 코드로 전락해 버렸다.

```java
@Override
public boolean equals(Object o) {
  if(o == null || !(o instanceof Point)) {
    return false;
  }

  Point p = (Point) o;
  return this.x == p.x && this.y = p.y;
}
```
차라리 이런식으로 instanceof를 사용했다면 좋았을 것이다.

## 상속 대신 컴포지션(Composition)을 사용하라
구체클래스의 하위클래스에서 값을 추가할 방법은 없지만, 우회방법이 있다.  
상속 대신에 Point 변수를 갖도록 Composition(구성)을 이용하는 방법이다.  

```java
public ColorPoint {
  private Point point;
  private Color color;

  public ColorPoint(int x, int y, Color color) {
    this.point = new Point(x, y);
    this.color = Objects.requireNonNull(color);
  }

  public Point asPoint() {
    return this.point;
  }

  @Override
  public boolean equals(Object o) {
    if(!(o instanceof ColorPoint)) {
      return false;
    }
    ColorPoint cp = (ColorPoint) o;
    return this.point.equals(cp) && this.color.equals(cp.color);
  }
}
```

이와 같이 컴포지션을 이용하면 상속에 때문에 발생하는 대칭성, 추이성, 리스코프 치환원칙에 위배되지 않는 코드를 작성할 수 있다.

## 일관성 (consistency)
**null이 아닌 모든 참조 값 x, y에 대해, x.equals(y)를 반복해서 호출하면 항상 true를 반환하거나 항상 false를 반환한다.**  
두 객체가 같다면 수정되지 않는 한 영원히 같아야 함을 의미한다.  
가변객체는 비교시점에 따라 서로 달라질 수 있지만 불변객체라면 한번 다르면 끝까지 달라야 한다. (그래서 불변객체로 만드는 것이 나을때가 많다)

하지만, 클래스가 불변이든 가변이든 equals의 판단에 신뢰할 수 없는 자원이 끼어들어서는 안된다.  
`java.net.URL` 클래스는 URL과 매핑된 host의 IP주소를 이용해 비교한다.  
당장 우리회사 서버만 봐도 L4스위치가 적용되있기 때문에 로드밸런싱이 되어 같은 도메인주소라도 그떄그떄 나오는 IP정보가 다르다.

```java
URL url1 = new URL("www.site-name.co.kr");
URL url2 = new URL("www.site-name.co.kr");

System.out.println(url1.equals(url2)); //?
```
실제 url1이 10.0.0.1 이라는 IP가 나왔다면  
url2의 경우 10.0.0.1이 나올수도 있고, 같은 대역에 있는 10.0.0.2가 나올 수도 있다.  
그렇기 떄문에 equals의 결과가 가변적이기 떄문에 다른 자원의 개입으로 인해 equals의 일관성이 깨지게 된다.  

그렇게 때문에 항상 equals를 사용할 때는 메모리에 존재하는 객체만을 사용한 결정적(deterministic) 계산만 수행해야 한다.

## not null
**null이 아닌 모든 참조 값 x에 대해, x.equals(null)은 false다**
기본적으로 x.equals(null)이 true가 되는 일은 생각하기 어렵다.  

```java
@Override 
public boolean equals(Object o) {
  if(o == null) return false; //불필요
  return this.x == o.x;
}
```
하지만 equals를 재정의 할 때 이러한 코드를 작성하기 쉽다.

```java
@Override
public boolean equals(Object o) {
  if(!(o instanceof MyClass)) return false; //묵시적 null검사
  MyClass clazz = (MyClass) o;
  return this.x == clazz.x;
}
```
이런식으로 instanceof 키워드를 통해 묵시적 null 체크를 하는 방법이 좋다.

# 요약 정리
## equals 구현 절차
* `==` 연산자를 사용해 입력된 파라미터와 자기자신이 같은 객체인지 검사한다. (동일성 검사 - Object Identity)
  * 성능 향상을 위한 코드
  * equals가 복잡할 때 같은 참조를 가진 객체에 대한 비교를 안하기 위함
* `instanceof` 연산자로 파라미터의 타입이 올바른지 체크
  * 묵시적 null체크 용도로도 사용
  * equals중에서는 같은 interface를 구현한 클래스끼리도 비교하는 경우가 있다.
* 입력을 올바른 타입으로 형변환한다.
  * Object타입의 파라미터를 비교하고자 하는 타입으로 형변환한다.
  * 앞서 `instanceof` 연산을 수행했기 때문에 100% 성공한다.
* 파라미터 Object 객체와 자기자신의 대응되는 핵심필드들이 모두 일치하는지 확인한다.
  * 하나라도 다르면 false를 리턴
  * 만약 interface기반의 비교가 필요하다면 필드정보를 가져오는 메서드가 interface에 정의되어있어야하고,  
  구현체 클래스에서는 메서드를 재정의 해야한다.
* float, double을 제외한 기본타입은 `==`을 통해 비교
* 참조(reference) 타입은 equals를 통해 비교
* float, double은 Float.compare(float, float)와 Double.compare(double, double)로 비교한다.
  * Float.Nan, -0.0f등을 비교하기 위함이다.
  * 이 메서드들은 float -> Float, double -> Double로 변환하는 오토박싱 기능이 수반되므로 성능상 좋지 못하다.
* 배열의 모든 원소가 핵심 필드라면 Arrays.equals를 사용하자
* null이 의심되는 필드는 Objects.equals(obj, obj)를 이용해 NullPointerException을 예방하자
* 성능을 올리고자 한다면
   * 다를 확률이 높은 필드부터 비교한다.
   * 비교하는 비용(시간복잡도)이 적은 비교를 먼저 수행

# 주의사항
* equals를 재정의 했다면, 대칭성, 추이성, 일관성에 대한 체크를 꼭 하자 (테스트 케이스 작성 요망)
* equals를 작성할 때는 hashcode도 반드시 재정의하자 (Item.11)
* 너무 복잡하게 해결하려 들지 말자
* equals의 파라미터는 Object이외의 타입으로 선언하지 말자 (컴파일 에러 발생)
* 구글에서 만든 @AutoValue을 이용해서 equals와 hashcode를 자동으로 재정의해보자 (Lombok의 @EqualsAndHashCode도 있다.)


# 참고
* Effective Java 3rd Edition - Item 10. equals는 일반 규약을 지켜 재정의하라
