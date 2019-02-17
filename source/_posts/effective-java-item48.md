---
title: Item 48. 스트림 병렬화는 주의해서 적용하라
catalog: true
Categories:
  - Effective-Java
tags:
  - Effective-Java
typora-root-url: effective-java-item48
typora-copy-images-to: effective-java-item48
date: 2019-02-17 16:34:16
subtitle:
header-img:
---

# 서론

주류 언어 중, 동시성 프로그래밍 측면에서는 항상 자바는 앞서왔다.  
처음 릴리즈된 1996년부터 스레드, 동기화, wait/notify를 지원했다. 
* 자바 5부터는 동시성 컬렉션인 java.util.concurrent 라이브러리와 실행자(Excutor) 프레임워크를 지원했다. 
* 자바 7부터는 고성능 병렬 분해(parallel decom-position) 프레임워크인 fork-join 패키지를 추가했다.  
(Fork-join pool에 대한 설명은 https://okky.kr/article/345720 여기 참고)  
* 자바 8부터는 parallel 메서드만 한 번 호출하면 파이프라인을 병렬 실행할 수 있는 Stream을 지원했다.

자바로 동시성 프로그램을 작성하기가 점점 쉬워지고 있지만, 이를 올바르고 빠르게 작성하는 일은 어렵다.  
동시성 프로그래밍을 할 때는 안전성(safety)과 응답 가능(liveness) 상태를 유지하기 애써야 한다.  
병렬 스트림 파이프라인 프로그래밍에서도 다를 바 없다.



# 파이프라인 병렬화가 불가능 한 경우 성능 개선이 되지 않는다.

```java
@RunWith(SpringRunner.class)
public class MersenPrimeTest {

    @Test
    public void test1() {
        long start = System.currentTimeMillis();
        primes().map(p -> TWO.pow(p.intValueExact()).subtract(ONE))
                .filter(mersenne -> mersenne.isProbablePrime(50))
                .limit(20)
                .forEach(System.out::println);
        long end = System.currentTimeMillis();
        System.out.println("running time : " + (end - start));
    }

    @Test
    public void test2() {
        long start = System.currentTimeMillis();
        primes().parallel().map(p -> TWO.pow(p.intValueExact()).subtract(ONE))
                .filter(mersenne -> mersenne.isProbablePrime(50))
                .limit(20)
                .forEach(System.out::println);
        long end = System.currentTimeMillis();
        System.out.println("running time : " + (end - start));
    }

    static Stream<BigInteger> primes() {
        return Stream.iterate(TWO, BigInteger::nextProbablePrime);
    }
}
```
스트림을 이용한 처음 20개의 메르센 소수를 생성하는 프로그램이다.  
이 프로그램을 실행 시켜보니 test1은 8.2초 정도의 수행시간을 보여줬다
test2는 stream의 성능을 향상시켜보고자 parallel()을 호출했다.  
하지만 메르센 소수의 값이 프린트 되지 않았고, 강제로 중지 하기 전까지 계속 돌고 있었다.  
아무것도 안된 원인은 stream 라이브러리가 이 파이프라인을 병렬화 하는 방법을 찾아내지 못했기 때문이다.  

* 데이터 소스가 Stream.iterate인 경우
* 중간 연산으로 limit()를 사용하는 경우
위 두 가지 경우에는 파이프라인 병렬화로 성능 향상을 기대하기 어렵다.  
뿐만 아니라 파이프라인 병렬화는 limit를 다룰 때 **CPU 코어가 남는다면, 원소를 몇 개 더 처리한 후 제한된 개수 이후의 결과를 버려도 아무런 해가 없다고 가정한다.**

따라서 스트림 파이프라인을 마구잡이로 병렬화 해선 안된다. 성능이 오히려 더 나빠질 수 있다.



# 언제 병렬화를 사용하나?

* ArrayList

* HashMap

* HashSet

* ConcurrentHashMap

* 배열(Array)

* int/long 범위


스트림의 데이터 소스가 위와 같은 클래스의 인스턴스 일 때 병렬화의 효과가 가장 좋다.  
위 자료구조들은 모두 데이터를 원하는 크기로 정확하고 손쉽게 나눌 수 있어 다수의 스레드에 분배하기에 좋다.  
나누는 작업은 **Spliterator**가 담당하며, Spliterator 객체는 Stream, Iterable의 spliterator() 메서드로 얻어올 수 있다.

또한 위의 자료구조는 **참조 지역성(locality of reference)** 이 높아 성능이 좋다.



# 참조 지역성 (Locality of Reference)

![locality](./locality.PNG)

만일 캐시(Cache)가 어떤 별도의 알고리즘 없이 중간 매개체로만 사용된다면,  
CPU가 Cache에서 데이터를 읽고 쓰는 속도나 메인 메모리에서 읽고 쓰는 속도나 마찬가지가 될 것이다.  

캐시 메모리가 제 역할을 하는 것은 데이터의 지역성(locality)를 이용하기 때문.  
데이터의 지역성은 다음의 세가지로 분류

* 공간적 지역성 (partial locality) 
  * 메인메모리에서 CPU가 요청한 주소지점의 데이터에 인접한 주소들이 앞으로 참조될 확률이 높음을 의미
  * 이웃한 원소들의 참조가 메모리에 연속적으로 저장되어 있어 다음 참조에 대한 접근 속도가 빠르게 함
* 시간적 지역성 (temporal locality)
  * 한번 참조되었던 데이터는 후에 다시 참조될 가능성이 높음을 의미
* 순차적 지역성 (sequential locality)
  * 따로 분기가 없는 한 데이터가 기억장치에 저장된 순서대로 인출되고 실행될 가능성이 높음을 의미 (FIFO)



# 스트림 파이프라인의 종단 연산

스트림 파이프라인의 종단연산의 동작방식 역시 병렬 수행 효율에 영향을 준다.  
종단 연산 중 병렬화에 가장 적합한 것은 축소(reduction)이다.  
축소는 파이프라인에서 만들어진 모든 원소를 하나로 합치는 작업이다.  
예를 들면 min, max, sum, count 같이 완성된 형태로 제공되는 메서드가 있다.  
anyMatch, allMatch, noneMatch처럼 조건에 맞음년 바로 반환되는 메서드도 병렬화에 적합하다.

반면, 가변 축소(Mutable Reduction)을 수행하는 Stream의 collect 메서드는 병렬화에 적합하지 않다. 
컬렉션들을 합치는 부담이 크기 때문이다.



# 병렬화에 대해 잘모르면 안하는게 낫다

스트림을 잘못 병렬화하면 (응답 불가를 포함해) 성능이 나빠질 뿐만 아니라 결과 자체가 잘못되거나 예상 못한 동작이 발생할 수 있다.  
(결과가 잘못되거나 오동작하는 것은 **안전 실패(safety failure)** 이라 한다.)  
안전 실패는 병렬화한 파이프라인이 사용하는 mappers, filters 혹은 프로그래머가 제공한 다른 함수 객체가 명시한대로 동작하지 않을 때 발생할 수 있다.

Stream 명세는 이때 사용되는 함수 객체에 관한 엄중한 규약을 정의해놨다.

## Stream 명세는 함수 객체에 대한 규약

1. Stream의 reduce 연산에 건네지는 accumulator(누적기)와 combiner(결합기) 함수는 반드시 **결합법칙**을 만족해야 한다.  
(결합 법칙 : (a op b) op c = a op (b op c))
2. 간섭받지 않아야 한다 (non-interfering) - 파이프라인이 수행되는 동안 **데이터소스가 변경되지 않아야한다.**
3. 상태를 갖지 않아야 한다 (stateless)
위의 요구사항을 지키지 못하더라도 순차적으로 실행하면 올바른 결과를 얻을 수 있다.  
하지만 병렬로 수행하면 기대한 결과가 나오지 않을 수 있고, 실패할 수 있으니 주의해야 한다.

# 스트림 병렬화는 오직 성능 최적화 수단임을 기억하라
다른 최적화와 마찬가지로 변경 전후로 반드시 성능테스트를 진행하여 병렬화를 사용할 가치가 있는지 확인해야 한다.  
이상적으로는 운영 시스템과 같은 환경에서 테스트하는 것이 좋다.  
보통은 병렬 스트림 파이프라인도 공통의 포크-조인 풀에서 수행되므로 (같은 스레드 풀을 사용)  
잘못된 파이프라인 하나가 다른 부분의 성능에까지 악영향을 줄 수 있음을 유념하자  
조건이 잘 갖춰지면 parallel 메서드 호출 하나로 거의 프로세서 코어 수에 비례하는 성능 향상을 볼 수 있다.



# 스트림 파이프라인 병렬화가 효과적인 예

```java
static long pi(long n) {
    return LongStream.rangeClosed(2, n)
                     .mapToObj(BigInteger::valueOf)
                     .filter(i -> i.isProbablePrime(50))
                     .count();
}
```

```java
static long pi(long n) {
    return LongStream.rangeClosed(2, n)
                     .parallel()
                     .mapToObj(BigInteger::valueOf)
                     .filter(i -> i.isProbablePrime(50))
                     .count();
}
```

* 𝛑(n) : n 보다 작거나 같은 소수의 개수를 계산하는 함수
* 책에서는 위의 함수를 실행하는데 31초, 아래 parallel()이 추가된 함수를 실행하는데 9.2초가 걸렸다고 한다.



# Random한 수의 경우

무작위 수들로 이뤄진 스트림을 병렬화하려거든 ThreadLocalRandom(혹은 Random) 보다는  
SplittableRandom 인스턴스를 이용하자. SplittableRandom은 정확히 이럴 때 쓰고자 설계된 것이라  
병렬화 하면 성능이 선형으로 증가한다. 한편 ThreadLocalRandom은 단일 스레드에서 사용하고자 만들어 졌다.  
병렬로 사용하는 경우에는 SplittableRandom > SplittableRandom 성능을 보인다.  

그냥 Random의 경우에는 모든 연산을 동기화하기 때문에 병렬 처리하면 최악의 성능을 보일 것이다.



# 요약

* 계산도 올바로 수행하고 성능도 빨라질거라는 확신 없이는 스트림 파이프라인 병렬화는 시도조차 하지 말자
* 스트림 병렬화를 잘못하면 프로그램이 오동작하거나 성능이 급격히 떨어질 수 있다.
* 병렬화를 할 경우에는 성능테스트를 반드시 진행하고, 결과가 정확한지 확인해야 한다.
* 계산도 정확하고 성능도 좋아졌음이 확실할 때, 그럴 때만 병렬화 버전을 운영 코드에 반영하라




# 참고

* Effective Java 3rd Edition - Item 48. 스트림 병렬화는 주의해서 적용하라
* [Thread pool과 ForkJoinPool](https://okky.kr/article/345720)

