---
title: Item 32. 제네릭과 가변인수를 함께 쓸 때는 신중하라
catalog: true
Categories:
  - Effective-Java
tags:
  - Effective-Java
typora-root-url: effective-java-item32
typora-copy-images-to: effective-java-item32
date: 2019-01-28 19:27:16
subtitle:
header-img:
---

# 서론
가변인수(varargs) 메서드와 제네릭은 Java 5버전에 함께 추가되었다.  
서로 잘 어우러지리라 생각하겠지만, 그렇지 않다. 가변인수는 메서드에 넘기는 인수의 개수를 클라이언트가 조절할 수 있게 해주는데, 구현방식에 헛점이 있다.  
가변인수 메서드를 호출하면, 가변인수를 담기 위한 배열이 자동으로 하나 만들어진다.  
그 결과 varargs 매개변수에 제네릭이나 매개변수화 타입이 포함되면 알기 어려운 컴파일 경고가 발생한다.

실제화 불가 타입은 런타임에는 컴파일타임보다 타입관련 정보를 적게 담고 있다`(소거)`

# 제네릭과 varargs를 혼용하면 타입 안정성이 깨진다.
매개변수화 타입(Parameterize Type (예 List<String>))의 변수가 타입이 다른 객체를 참조하면 **힙 오염**이 발생한다.

```java
static void dangerous(List<String>... stringLists) {
  List<Integer> intList = List.of(42);
  //varargs는 내부적으로 배열이고
  //배열은 공변이기 때문에 List<String>타입은 Object의 하위클래스로 인식되어
  //Object[]에 참조 될 수 있다.
  Object[] objects = stringLists;
  Object[0] = intList; //힙 오염 발생
  String s = stringLists[0].get(0); // ClassCastException
}
```
위의 코드를 한줄한줄 분석해보자
1. List<String> varargs형태의 파라미터를 받는 메서드이다.
2. List<Integer> 제네릭타입 객체를 생성하여 42라는 값을 추가하였다.
3. varargs는 내부적으로 배열이고, 배열은 공변이기 때문에 List<String>[] -> Object[]에 참조될 수 있다.
4. Object[0] = intList 초기화  
(내부적으로는 List<String> 타입이지만, 런타임에는 제네릭 타입이 소거되므로 같은 List로만 인식되어 할당이 가능하다. **힙 오염 발생**)
5. stringList[0]을 하면 List가 나오고 List의 0번째 인덱스 위치의 객체를 호출해 눈에 보이지 않는 String으로 형변환한다.  
-> 여기서 ClassCastException이 발생 

이처럼 타입안정성이 깨지기 때문에 제네릭 varargs 배열 매개변수에 값을 저장하는 것은 안전하지 않다.  
(실제로 실무에서는 저런 경우는 거의 없다. 웬만하면 varargs 형태로 받아와서 그대로 사용하기 때문)

# @SafeVarargs
자바 7 전에는 제네릭 가변인수 메서드의 작성자가 호출자 쪽에서 발생하는 경고에 대해서 해줄 수 있는 일이 없었다.  
사용자는 이 경고를 그냥 무시하거나, 사용하는 메서드에서 @SupprssWarning("unchecked") annotation을 달아 경고를 숨겨야 했다.  
자바 7부터 @SafeVarargs annotation이 추가되어 제네릭 가변인수 메서드 작성자가 클라이언트 측에서 발생하는 경고를 숨길 수 있게 되었다.

단, @SafeVarargs annotation은 메서드 작성자가 그 메서드가 타입 안전함을 보장하는 장치이므로  
반드시 메서드가 타입 안전한 경우에만 이 annotation을 붙이는 것이 좋다.

## 어떤게 타입 안전할까?
* 가변인수 메서드를 호출하면 varargs 매개변수를 담는 제네릭 배열이 만들어 진다.  
* 메서드 내에서 이 배열에 아무것도 저장하지 않고, 배열의 참조가 밖으로 노출되지 않는다면 타입 안전하다.
* 순수하게 메서드의 생산자 역할만 충실히 하면 메서드는 안전하다.

## 자신의 제네릭 매개변수 배열의 참조를 노출하는 것은 위험하다.
```java
static <T> T[] toArray(T... args) {
  return args;
}
```
```java
static <T> T[] pickTwo(T a, T b, T c) {
  switch(ThreadLocalRandom.current().nextInt(3)) {
    case 0: return toArray(a, b);
    case 1: return toArray(b, c);
    case 2: return toArray(c, a);
  }
  throw new AssertiionError();
}
```
```java
public static void main(String[] args) {
  String[] attributes = pickTwo("좋은", "빠른", "저렴한");
}
```
아무 문제가 없는 메서드이니 별다른 문제없이 컴파일 된다.  
하지만 실행하면 ClassCastException을 던진다.  
어디서 발생하는 에러일까?

정답은 바로 String[] attributes = pickTwo(); 메서드에서 보이지 않는 형변환 시 발생한다.

실제로는 아래와 같다.
```java
String[] attributes = (String[]) pickTwo("좋은", "빠른", "저렴한");
```
Object[]는 String[]의 상위타입이 아니므로 형변환할 수 없다.  
이 예시는 **제네릭 varargs 매개변수 배열에 다른 메서드가 접근하도록 허용하면 안전하지 않다.** 를 다시금 알려주는 예제이다.

단 예외가 두 가지 있다.
* @SafeVarargs로 선언된 타입 안전성이 보장된 또 다른 varargs 메서드에 넘기는 것은 안전하다.
* 배열 내용의 일부 함수를 호출만 하는 (varargs를 받지않는) 일반 메서드에 넘기는 것도 안전하다.

# 제네릭 varargs 매개변수를 안전하게 사용하는 메서드
```java
@SafeVarargs
static <T> List<T> flatten(List<? extends T>... lists) {
  List<T> result = new ArrayList<>();
  for(List<? extends T> list : lists) {
    result.addAll(list);
  }
  return result;
}
```

위의 메서드는 안전하다.  
varargs 배열을 직접 노출 시키지 않고, T타입의 제네릭 타입을 사용하였기 때문에 ClassCastException 또한 발생할 일이 없다.  
안전한 varargs 메서드에는 @SafeVarargs annotation을 달아서 컴파일러 경고를 없애는 것이 좋다.

# 제네릭 varargs 매개변수를 List로 대체하라
```java
static <T> List<T> flatten(List<List<? extends T>> lists) {
  List<T> result = new ArrayList<>();
  for (List<? extends T> list : lists) {
    result.addAll(list);
  }
  return result;
}
```
이 방식의 장점은 이 메서드의 타입 안전성을 검증할 수 있다는 점이다.  
@SafeVarargs를 달지 않아도 되며 실수로 안전하다고 판단할 걱정도 없다.  
단점은 클라이언트 코드가 살짝 지저분해지고, 속도가 약간 느려질 수 있다는 점이다.


# 정리
* varargs 매개변수는 단순히 파라미터를 받아와 메서드의 생산자(T 타입의 객체를 제공하는 용도)로만 사용하자
* varargs는 read-only라고 생각하고, 아무런 데이터를 저장하지 말자
* varargs 배열을 외부에 리턴하거나 노출하지 말자.  
웬만하면 다시 컬렉션(List)에 담아 리턴하는 안전한 방식을 취하자


# 참고
* Effective Java 3rd Edition - Item 32. 제네릭과 가변인수를 함께 쓸 때는 신중하라