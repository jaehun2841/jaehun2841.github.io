---
title: Item 58. 전통적인 for 문보다는 for-each문을 사용하라
catalog: true
Categories:
  - Effective-Java
tags:
  - Effective-Java
typora-root-url: effective-java-item58
typora-copy-images-to: effective-java-item58
date: 2019-02-28 18:39:24
subtitle:
header-img:
---

# 서론

스트림(Stream)이 제격인 작업이 있고, 반복이 제격인 작업이 있다.   

```java
for(Iterator<Element> i = c.iterator(); i.hasNext();) {
    Element e = i.next();
} 
```

```java
for(int i = 0; i < a.length; i++) {
    int z = a[i]; // a[i]로 무언가를 한다.
}
```

위의 for문과 같은 관용구들은 while문 보다는 낫지만 가장 탁월한 방법은 아니다.  
**반복자와 인덱스 변수는 모두 코드를 지저분하게 할 뿐 우리에게 필요한 건 원소들 뿐이다.**  
또한 컬렉션이냐 배열이냐에 따라 코드 형태가 달라지기 때문에 주의해야 한다.



# 향상된 for문 (enhanced for statement)

* 반복자와 인덱스 변수를 사용하지 않으니 코드가 깔끔해지고 오류가 날 일이 없다.
* 하나의 관용구로 되어있어서 배열이든 컬렉션이든 코드의 형태가 같다.
* 콜론(:)은 **안의(in)** 라고 읽으면 된다. (elements안의 각 원소 e에 대해)
* 반복 대상이 컬렉션이든 배열이든 for-each문을 사용해도 속도는 그대로이다.
* for-each 문이 만들어 내는 코드는 사람이 손으로 최적화한 것과 사실상 같다.



# 컬렉션이 중첩되는 경우 for-each의 이점이 커진다.

```java
enum Suit { CLUB, DIAMOND, HEART, SPADE}
enum Rank { ACE, DEUCE, THREE, FOUR, FIVE, SIX, SEVEN, EIGHT, NINE, TEN, JACK, QUEEN, KING}

static Collection<Suit> suits = Arrays.asList(Suit.values());
static Collection<Rank> ranks = Arrays.asList(Rank.values());

List<Card> deck = new ArrayList<>();
for(Iterator<Suit> i = suits.iterator(); i.hasNext();) {
    for(Iterator<Rank> j = ranks.iterator(); j.hasNext();) {
        deck.add(new Card(i.next(), j.next()));
    }
}
```

* `deck.add(new Card(i.next(), j.next()));` 줄이 오류를 일으킨다.
* 원래대로 하면 숫자 1개당 rank가 여러번 불려야 하는데 저러면 숫자 1개에 rank 1개 불려 `NoSuchElementException` 을 던진다.



```java
enum Suit { CLUB, DIAMOND, HEART, SPADE}
enum Rank { ACE, DEUCE, THREE, FOUR, FIVE, SIX, SEVEN, EIGHT, NINE, TEN, JACK, QUEEN, KING}

static Collection<Suit> suits = Arrays.asList(Suit.values());
static Collection<Rank> ranks = Arrays.asList(Rank.values());

List<Card> deck = new ArrayList<>();
for(Suit suit : suits) {
    for(Rank rank : ranks) {
        deck.add(new Card(suit, rank));   
    }
}
```

* 위와 같이 for-each를 사용하면 깔끔하고 간단하게 코드를 짤 수 있다.



# for-each를 사용할 수 없는 상황

* **파괴적인 필터링(destructive filtering)**
  * 컬렉션을 순회하면서 선택된 원소를 제거해야 한다면 반복자의 remove 메서드를 호출해야 한다.
  * Java 8 부터 Collection의 removeIf 메서드를 사용해 컬렉션을 명시적으로 순회하지 않을 수 있다.
* **변형(transforming)**
  * 리스트나 배열을 순회하면서 원소 값 일부 혹은 전체를 교체하는 경우에는 인덱스를 사용해야 한다.
* **병렬 반복(parallel iteration)**
  * 여러 컬렉션을 병렬로 순회해야 한다면 각각의 반복자와 인덱스 변수를 사용해 엄격하고 명시적으로 제어해야 한다.



# 정리

* for문 대신 for-each를 사용할 수 있는 경우는 무조건 사용하자
* 전통적인 for문과 비교했을 때, for-each문은 명료하고 유연하고 버그를 예방해 준다.
* for-each문은 컬렉션과 배열은 물론 Iterable 인터페이스를 구현한 객체라면 무엇이든 순회 가능하다.



# 참고

* Effective Java 3rd Edition - Item 58. 전통적인 for 문보다는 for-each문을 사용하라

