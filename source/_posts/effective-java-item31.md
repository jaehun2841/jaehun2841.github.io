---
title: Item 31. 한정적 와일드 카드(bounded wildcard type)를 사용해 API 유연성을 높여라
catalog: true
Categories:
  - Effective-Java
tags:
  - Effective-Java
typora-root-url: effective-java-item31
typora-copy-images-to: effective-java-item31
date: 2019-01-26 21:16:50
subtitle:
header-img:
---

# 서론
Generic에서 매개변수화 타입(Parameterize Type - List<String>)는 불공변이다.

> 공변 vs 불공변
Java에서 배열은 공변(variant), Generic은 불공변(invariant)이라 한다.  
배열의 경우 Object[]과 String[] 간에는 부모-자식 클래스 관계가 성립한다. 이를 공변이라한다.  
하지만 Generic에서는 List<Object>와 List<String>은 부모-자식 관계가 성립하지 않는다 이를 불공변이라한다.

즉, List<String>은 String타입의 문자열만 넣을 수 있지만, List<Object>는 문자열이건, 숫자건 다 넣을 수 있으므로, 부모클래스에서 가능한 일은 자식클래스에서도 가능해야 한다는 **리스코프 치환원칙**에 위배된다.

하지만 때로는 불공변 방식보다 유연한 무언가가 필요하다.

# 생산자(producer) 와일드카드 적용
```java
Stack<Number> numberStack = new Stack<>();
Iterator<Integer> integers = List.of(1,2,3,4,5);
numberStack.pushAll(integers);
```
Integer는 Number의 하위타입이기 때문에 잘 동작할 것 같지만, 실제로는 컴파일 시에 오류가 발생한다.

Parameterize Type이 불공변이기 때문에 Iterator<Number>와 Iterator<Integer>는 부모-자식 관계가 아니다.  
그렇기 때문에 addAll 메서드의 파라미터로 사용할 수 없는 것이다.

이런 경우 생산자(producer) 파라미터에 와일드카드 타입을 적용하여 유연하게 만들 수 있다.

```java
public void pushAll(Iterator<? extends E> iterator) {
  for (E e : iterator) {
    push(e);
  }
}
```
생산자(producer) 파라미터란, 파라미터로 제공되어 메서드 내에서 사용 될 객체를 공급해 주는 파라미터이다. 

`<? extends E>` 형태로 사용하는 경우 E 타입의 하위타입만 파라미터로 올 수 있게 제약을 거는 것이다.  
`모든 타입은 자기 자신의 하위타입이다!` 라는 규칙이 있으므로 자기 자신에 대한 타입도 들어올 수 있다.



# 소비자(consumer) 와일드카드 적용

```java
public void popAll(Collection<E> dst) {
  while(!isEmpty()) {
    dst.add(pop());
  }
}
```
```java
Stack<Number> numberStack = new Stack<>();
Collection<Object> objects = List.of(1, "String");
numberStack.popAll(objects);
```
컴파일 하게되면 Collection<Object>는 Collection<Number>의 하위타입이 아니다라는 오류가 나온다.  
이를 해결하기 위해서는 super를 이용해 와일드카드를 작성해야 한다.

```java
public void popAll(Collection<? super E> dst) {
  while(!isEmpty()) {
    dst.add(pop());
  }
}
```
소비자(consumer) 파라미터란, 메서드에 제공되어 메서드 내에서 제공되는 객체를 자기 자신이 사용하는 파라미터를 의미한다.

# PECS(Producer-Extends, Consumer-Super)
다음 공식을 외워 어떤 와일드 카드 타입을 쓸지 기억하자
* 생산자(producer) - Generic 클래스 내에 타입 파라미터를 이용해 객체를 제공하는 것 
  * <? extends E>를 사용하여 유연성을 높일 수 있다.
* 소비자(consumer) - Generic 클래스 내의 자원을 사용하는 객체를 파라미터로 전달하는 것 
  * <? super E>를 사용하여 유연성을 높일 수 있다.
  * Comparable, Comparator는 소비자로 사용된다.


```java
Set<Integer> integers = Set.of(1,2,3);
Set<Double> doubles = Set.of(1.0, 2.0, 3.0);
Set<Number> numbers = union(integers, doubles);
```
```java
public static <E> Set<E> union(Set<E> s1, Set<E> s2) {
  Set<E> set = new HashSet<>();
  set.addAll(integers);
  set.addAll(doubles);
  return set;
}
```
위와 같이 메서드가 만들어지면 타입이 맞지 않는다고 컴파일 오류가 나게 된다.  
Set<E>은 불공변이기 때문에 Set<Integer>와 Set<Double>은 Set<Number>의 하위타입이 아니기 떄문이다.

```java
public static <E> Set<E> union(Set<? extends E> s1, Set<? extends E> s2) {
  Set<E> set = new HashSet<>();
  set.addAll(integers);
  set.addAll(doubles);
  return set;
}
```
위와 같이 변경하면 컴파일 오류도 해결되고 잘 실행 된다.
이렇게 공통 API를 작성할 때는 와일드 카드를 적절하게 쓰도록 하자.  
클래스 사용자가 와일드카드 타입을 신경써야 한다면, 그 API에 문제가 있을 가능성이 크다.

## Java7에서는...
```java
Set<Number> numbers = Union.<Number>union(integers, doubles);
```
위와 같은 형태로 명시적으로 타입인수를 사용해야 한다.  
타입추론능력이 부족하기 때문이다.

# 심화 - Comparable
```java
public static <E extends Comparable<E>> E max(List<E> list)
```
와일드카드를 통해 좀 더 다듬은 모습이다.  
**주의: Comparable을 구현한 클래스만 파라미터로 올 수 있다**
```java
 public static <E extends Comparable<? super E>> E ma(List<? extends E> list) {
            list.sort(Comparator.reverseOrder());
            return list.get(0);
        }
```
* 일반적으로는 Comparable<E>보단 Comparable<? super E>를 사용하는게 낫다. (대부분 소비자로 사용)

# 심화2 - 와일드카드를 적절히 사용하라
```java
public static <E> void swap(List<E> list, int i, int j);
public static void swap(List<?> list, int i, int j);
```
리스트 내의 특정 인자들의 위치를 뒤바꿔주는 메서드이다.  
어느 메서드가 더 좋을까?

기본적으로는 메서드 선언에 타입 매개변수가 한 번만 나오면 와일드카드로 대체하는 것이 좋다.

```java
public static void swap(List<?> list, int i, int j) {
  list.set(i, list.set(j, list.get(i)));
}
```
위의 코드는 list.get하는 부분에서 컴파일 오류가 난다.
비한정적 와일드카드를 사용하고 있기 때문에 list.set을 하는 경우에는 null밖에 넣을 수가 없다.  
이런 경우 도우미 메서드를 따로 이용한다.
```java
public static <E> void swapHelper(List<E> list, int i, int j) {
  list.set(i, list.set(j, list.get(i)));
}
```
이 경우에는 List<E>의 리턴타입이 항상 E인 것을 알기 때문에  
런타임 시, 타입안정성을 보장 할 수 있고 set하는 경우에도 E 타입을 set할 것을 알기 때문에 컴파일 오류 없이 작동 한다.

# 정리
조금 복잡하더라도 와일드카드 타입을 적용하면 API가 훨씬 유연해진다.  
그러나 널리 쓰일 라이브러리를 작성한다면 반드시 와일드카드 타입을 적절히 사용해야 한다.  
PECS 공식을 기억하여 생산자에는 extends, 소비자에는 super를 사용하자.

# 참고
* Effective Java 3rd Edition - Item 31. 한정적 와일드 카드(bounded wildcard type)를 사용해 API 유연성을 높여라