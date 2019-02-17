---
title: Item 45. 스트림은 주의해서 사용하라
catalog: true
Categories:
  - Effective-Java
tags:
  - Effective-Java
typora-root-url: effective-java-item45
typora-copy-images-to: effective-java-item45
date: 2019-02-16 15:25:05
subtitle:
header-img:
---

# 서론

스트림 API는 다량의 데이터 처리 작업(순차적이든 병렬적이든)을 돕고자 자바8에 추가되었다.  
이 API가 제공하는 추상 개념 중 핵심은 두 가지다. 

* 스트림(Stream)은 데이터 원소의 유한 혹은 무한 시퀀스(sequence)를 의미
* 스트림 파이프라인(Stream Pipeline)은 이 원소들로 수행하는 연산단계를 표현하는 개념



스트림의 원소들은 어디서부터든 올 수 있다. 대표적으로

* 컬렉션 (Collection)
* 배열 (Array)
* 파일 (File)
* 정규표현식 (Regex Pattern Matcher)
* 난수 생성기 (Random Generator)
* 다른 스트림 (Other Stream)



스트림 안의 데이터 원소들은 객체 참조(reference)나 기본 타입(int, long, double)을 지원한다.

* **Stream** : 객체 참조타입에 대한 Stream
* **IntStream** : int 타입에 대한 Stream
* **LongStream** : long 타입에 대한 Stream
* **DoubleStream** : double 타입에 대한 Stream

기본타입의 경우 위의 IntStream, LongStream, DoubleStream과 같은 Stream을 사용하는 것이 성능상 좋다.



# 스트림 파이프라인 (Stream Pipeline)

스트림 파이프 라인은 소스 스트림에서 시작해 종단 연산(terminal operation)으로 끝나며,  
그 사이에 하나 이상의 중간 연산(intermediate operation)이 있을 수 있다.

## 종단 연산 (terminal operation)

- **forEach(Consumer<? super T> consumer)** : Stream의 요소를 순회

-  **count()** : 스트림 내의 요소 수 반환
-  **max(Comparator<? super T> comparator)** : 스트림 내의 최대 값 반환 
-  **min(Comparator<? super T> comparator)** : 스트림 내의 최소 값 반환
-  **allMatch(Predicate<? super T> predicate)** : 스트림 내에 모든 요소가 predicate 함수에 만족할 경우 true
-  **anyMatch(Predicate<? super T> predicate)** : 스트림 내에 하나의 요소라도 predicate 함수에 만족할 경우 true
-  **noneMatch(Predicate<? super T> predicate)** : 스트림 내에 모든 요소가 predicate 함수에 만족하지않는 경우 true
-  **sum()** : 스트림 내의 요소의 합 (IntStream, LongStream, DoubleStream)
-  **average()** : 스트림 내의 요소의 평균 (IntStream, LongStream, DoubleStream)



## 중간 연산 (intermediate operation)

* f**ilter(Predicate<? super T> predicate)** : predicate 함수에 맞는 요소만 사용하도록 필터

* **map(Function<? Super T, ? extends R> function)** : 요소 각각의 function 적용

* **flatMap(Function<? Super T, ? extends R> function)** : 스트림의 스트림을 하나의 스트림으로 변환

* **distinct()** : 중복 요소 제거

* **sort()** : 기본 정렬

* **sort(Comparator<? super T> comparator)** : comparator 함수를 이용하여 정렬

* **skip(long n)** : n개 만큼의 스트림 요소 건너뜀

* **limit(long maxSize)** : maxSize 갯수만큼만 출력




# 스트림의 지연 평가(Lazy evaluation)

스트림에 대한 평가는 종단 연산이 호출될 때 이뤄지며, 종단 연산에 쓰이지 않는 데이터 원소는 계산에 쓰이지 않는다.  
이러한 지연 평가가 무한 스트림을 다룰 수 있게 해주는 열쇠다.  

쉽게 말해 **종단 연산(terminal operation)이 없는 스트림 파이프라인은 아무 일도 일어나지 않는다.**



# Stream 예제 - 아나그램(anagram)
## 일반 loop를 이용한 코드
```java
public static void main(String[] args) {
    File dectionary = new File(args[0]);
    int minGroupSize = Integer.parseInt(args[1]);

    Map<String, Set<String>> groups = new HashMap<>();
    try (Scanner s = new Scanner(dectionary)) {
        while(s.hasNext()) {
            String word = s.next();
            groups.computeIfAbsent(alphabetize(word), 
                                   (unused) -> new TreeSet<>()).add(word);
        }
    } catch (FileNotFoundException e) {
        e.printStackTrace();
    }
    
    for(Set<String> group : groups.values()) {
        if(group.size() >= minGroupSize) {
            System.out.println(group.size() + ": " + group);
        }
    }
}

private static String alphabetize(String s) {
    char[] a = s.toCharArray();
    Arrays.sort(a);
    return new String(a);
}
```

* 자바 8에서 추가된 computeIfAbsent 메서드를 사용했다
* computeIfAbsent : 맵 안에 키가 있는지 찾은 다음, 있으면 단순히 그 키에 매핑된 값을 반환한다.


## Stream을 과도하게 쓴 코드
```java
public static void main(String[] args) throws IOException {
    File dectionary = new File(args[0]);
    int minGroupSize = Integer.parseInt(args[1]);

    try(Stream<String> words = Files.lines(dectionary.toPath())) {
        words.collect(
            groupingBy(word -> word.chars().sorted()
                       .collect(StringBuilder::new,
                                (sb, c) -> sb.append((char) c),
                                StringBuilder::append).toString()))
            .values().stream()
            .filter(group -> group.size() >= minGroupSize)
            .map(group -> group.size() + ": " + group)
            .forEach(System.out::println);
    }
}
```

* 스트림을 과용하면 프로그램이 읽거나 유지보수 하기 어려워진다.



## Stream을 적절히 활용한 코드

```java
public static void main(String[] args) throws IOException {
    File dectionary = new File(args[0]);
    int minGroupSize = Integer.parseInt(args[1]);

    try(Stream<String> words = Files.lines(dectionary.toPath())) {
        words.collect(groupingBy(word -> alphabetize(word)))
            .values().stream()
            .filter(group -> group.size() >= minGroupSize)
            .forEach(group -> System.out.println(group.size() + ": " + group));
    }
}
```

* 스트림을 적절하게 사용하면 명료해진다.
* 람다에서는 타입 추론 기능을 사용하기 때문에 주로 타입 이름을 생략한다.  
  따라서 매개변수 이름을 잘 지어야 스트림 파이프라인의 가독성이 유지된다.
* 연산에 적절한 이름을 지어 주고 도우미 메서드를 적정히 활용하는 것은 일반 반복코드보다 스트림 파이프라인에서 훨씬 크다



# 기존 코드는 필요한 경우에만 스트림으로 리팩터링 하자

스트림을 처음 쓰기 시작하면 모든 반복문을 스트림으로 바꾸고 싶은 유혹이 생긴다.  
스트림으로 바꾸는 게 가능할 지라도, 가독성과 유지보수 측면에서 볼 때 손해를 볼 수 있기 때문에 무작정 바꾸지는 말자  
스트림과 반복문을 적절히 활용하는 것이 최선이다.



# 코드블럭 vs 람다(lambda)

* 코드블럭에서는 범위 안의 지역변수를 읽고 수정할 수 있다.
* 람다에서는 final이거나 사실상 final인 변수만 읽을 수 있다. (캡처)
  지역 변수를 수정하는 것은 불가능하다.
* 코드 블럭에서는 return을 이용해 메서드를 빠져나갈 수 있다.
* 코드 블럭에서는 break, continue문을 통해 블럭 바깥의 반복문을 종료 할 수 있다.
* 메서드 선언에 명시된 예외(Exception)을 던질 수 있다.
* 하지만 람다로는 모든 것이 불가능하다.



# 스트림이 적합한 경우

* 원소들의 시퀀스를 일관되게 변환한다.
* 원소들의 시퀀스를 필터링한다.
* 원소들의 시퀀스를 하나의 연산을 사용해 결합한다. (더하기, 연결하기, 최소값 등..)
* 원소들의 시퀀스를 컬렉션에 모은다
* 원소들의 시퀀스에서 특정 조건을 만족하는 원소를 찾는다.



# 스트림이 적합하지 않은 경우

* 데이터가 파이프라인의 여러 단계(stage)를 통과할 때 이 데이터의 각 단계에서의 값들에 동시에 접근하기 어려운 경우
* 스트림 파이프라인은 한 값을 다른 값에 매핑하고 나면 원래의 값을 잃는 구조이기 때문



## 예제 - 메르센 소수 출력하기

```java
static Stream<BigInteger> primes() {
    return Stream.iterator(TWO, BigInteger::nextProbablePrime);
}
```

* 위의 코드는 무한 스트림을 반환하는 메서드이다.
* 아직 종단 연산이 없기 때문에 실행되지는 않는 스트림이다.



```java
public static void main(String[] args) {
    primes().map(p -> TWO.pow(p.intValueExact().subtract(ONE)))
            .filter(mersenne -> mersenne.isProbablePrime(50))
            .limit(20)
            .forEach(System.out::println);
}
```

* 위의 코드는 메르센 소수를 20개를 출력하는 프로그램이다.



# 예제 - 데카르트 곱
## 데카르트 곱을 반복을 이용하여 구현
```java
private static List<Card> newDeck() {
    List<Card> result = new ArrayList<>();
    for(Suit suit : Suit.values()) 
        for(Rank rank : Rank.values())
            result.add(new Card(suit, rank));
    return result;
}
```
* 카드는 숫자(rank)와 무늬(suit)를 묶은 불변 값 클래스이거 숫자와 무늬는 열거타입이다.
* for-each를 통해 스트림을 모르는 개발자라도 쉽게 알아 볼 수 있는 코드이다

```java
private static List<Card> newDeck() {
    return Stream.of(Suit.values())
    .flatMap(suit -> Stream.of(Rank.values())
                      .map(rank -> new Card(suit, rank)))
    .collect(toList());
}
```

* Stream을 중첩하여 만든 코드이다.
* 개발자에 따라 어떤 코드가 가독성, 유지보수성이 좋은 코드인지 갈리겠지만,  
  내 기준에는 첫번째 코드가 더 가독성과 유지보수성이 좋아보인다.



# 요약

* 스트림과 함수형 프로그래밍에 익숙한 프로그래머라면 스트림방식이 좀 더 명확하다
* 스트림을 사용할 때가 있고, 반복을 사용할 때가 있다.
* 스트림과 반복 중 어느 쪽이 나은지 확신하기 어렵다면 둘 다 해보고 더 나은 쪽을 택하는 것이 좋다.



# 참고

* Effective Java 3rd Edition - Item 45. 스트림은 주의해서 사용하라