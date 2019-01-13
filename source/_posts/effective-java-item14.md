---
title: Item 14. Comparable을 구현할지 고려하라
catalog: true
Categories:
  - Effective-Java
tags:
  - Effective-Java
typora-root-url: effective-java-item14
typora-copy-images-to: effective-java-item14
date: 2019-01-13 19:26:38
subtitle:
header-img:
---

# 서론
Comparable을 구현하는 이유는 클래스의 인스턴스들간의 Ordering을 목적으로 구현하는 클래스이다.  
따라서 Comparable을 구현한 클래스에 대한 배열은 다음처럼 손쉽게 정렬 할 수 있다.

```java
Arrays.sort(arr);
```

사실상 자바 플랫폼 라이브러리의 모든 값 클래스와 열거타입은 Compareable을 구현했다.  
알파벳, 숫자, 연대 같이 순서가 명확한 값 클래스를 작성한다면, 반드시 Comparable 인터페이스를 구현하자

# compareTo 메서드 규약

이 객체와 주어진 객체의 순서를 비교한다.  
이 객체가 주어진 객체보다 작으면 음의 정수를, 같으면 0을, 크면 양의 정수를 리턴한다.  
이 객체와 비교할 수 없는 타입이 주어지면 ClassCaseException을 던진다.

## 대칭성
* Comparable을 구현한 클래스는 모든 x, y에 대해 x.compareTo(y) == (y.compareTo(x)) * (-1)을 만족해야 한다.
* 따라서 x.compareTo(y)가 예외를 던지는 경우, y.compareTo(x)도 예외를 던져야 한다.

## 추이성
* Comparable을 구현한 클래스는 모든 x, y, z에 대해 x.compareTo(y) > 0 이고 y.compareTo(z)이면, x.compareTo(z)를 만족해야 한다.

## 반사성
* Comparable을 구현한 클래스 z는 x.compareTo(y) == 0 이면, sgn(x.compareTo(z)) == sgn(y.compareTo(z))를 만족해야 한다.

## equals 
* Comparable을 구현한 클래스는 모든 x, y에 대해 x.compareTo(y) == 0 이면, x.equals(y)를 만족하는 것이 좋다. (권고사항은 아니다.)  
이 권고를 지키지 않으려면, `주의: 이 클래스의 순서는 equals 메서드와 일관되지 않다.`라고 명시해 주자.

# equals와 compareTo 차이점
compareTo 규약 4번째의 예시로 BigDecimal 클래스가 있다.  
new Decimal("1.0")과 new Decimal("1.00")이 있다고 할 때 두 객체를 HashSet<Decimal>에 담게 되면 size는 2개가 된다.  
하지만 TreeSet<Decimal>에 담게 되면 size는 1개가 된다.  

왜 이런 결과가 나올까?  
HashSet에서는 equals를 기반으로 비교하기 때문에 new Decimal("1.0")과 new Decimal("1.00")은 서로 다른 객체이다. 그렇기 때문에 size가 2개가 된다.  
하지만 TreeSet에서는 객체에 대한 동치성 비교를 compareTo로 하기 때문에 new Decimal("1.0")과 new Decimal("1.00")의 compareTo는 0을 리턴한다.  
따라서 같은 객체로 인식하여 size가 1개가 된다.

# compareTo 안티패턴
compareTo 메서드에서 관계연산자 (`<` 와 `>`)를 사용하지 말아야 한다.  
대신 Type.compare(T t1, T t2)를 사용하여 비교하는 것이 좋다.

안티 패턴 코드
```java
public int compareTo (int x, int y) {
  return x < y ? 1 : (x == y) ? 0 : -1;
}
```

hashcode의 차를 이용한 비교는 안된다. (추이성에 위배된다.)

```java
static Comparator<Object> hashCodeOrder = new Comparator<>() {
  (Object o1, Object o2) -> o1.hashCode() - o2.hashCode();
}
```
위의 코드 처럼 실행하면, 정수 overflow를 일으키거나 IEEE754 부동소수점 계산방식에 따른 오류를 발생 시킬 수 있다.  
따라서 아래 코드로 고쳐서 사용하는 것이 좋다.
```java
static Comparator<Object> hashCodeOrder = new Comparator<>() {
  (Object o1, Object o2) -> Integer.compare(o1.hashCode(), o2.hashCode())
}
```
```java
static Comparator<Object> hashCodeOrder = Comparator.comparingInt(o -> o.hashCode());
```

# 사용 예제
## 기본 타입 필드가 여러 개 일때 비교자
```java
public int compareTo(PhoneNumber pn) {
  int result = Short.compare(this.areaCode, pn.areaCode);
  if(result == 0) {
    result = Short.compare(this.prefix, pn.prefix);
    if(result == 0) {
      result = Short.compare(this.line
      Num, pn.lineNum);
    }
  }
}
```
필드의 정렬 우선순위를 정해 같은 값이 있을 때 마다 조건을 추가한다.

## 비교자 생성 메서드를 이용한 비교자
```java
private static final Comparator<PhoneNumber> COMPARATOR 
                        = comparingInt((PhoneNumber pn) -> pn.areaCode)  
                                      .thenComparingInt(pn -> pn.prefix)  
                                      .thenComparingInt(pn -> pn.lineNum)
```
comparingInt라는 static 메서드를 import하여 사용하고, 2번째 조건 부터 thenComparingInt를 사용하여 비교자를 추가 할 수 있다.  
최초 사용 시, PhoneNumber pn을 사용하여 람다식에서 타입을 추론 할 수 있도록 코드를 추가 하였다.  
thenComparingInt부터는 자바 컴파일러가 충분히 타입을 추론 할 수 있으므로 명시적으로 지정하지 않았다.  
Long타입과 Double타입에 대해서는 각각 comparingLong, thenComparingLong과 comparingDouble, thenComparingDouble이 별도로 있다.


# 참고
* Effective Java 3rd Edition - Item 14. Comparable을 구현할지 고려하라
