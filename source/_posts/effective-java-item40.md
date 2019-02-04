---
title: Item 40. @Override 애너테이션을 일관되게 사용하라
catalog: true
Categories:
  - Effective-Java
tags:
  - Effective-Java
typora-root-url: effective-java-item40
typora-copy-images-to: effective-java-item40
date: 2019-02-04 19:51:36
subtitle:
header-img:
---



# 서론

자바가 기본으로 제공하는 애너테이션 중 보통의 프로그래머에게 가장 중요한 것은 @Override일 것이다.  
@Override는 메서드 선언에만 달 수 있으며, 이 애너테이션의 의미는 상위 클래스의 메서드를 재정의 했음을 의미한다.



# @Override를 선언하지 않은 메서드

```java
public class Bigram {
    private final char first;
    private final char second;

    public Bigram(char first, char second) {
        this.first = first;
        this.second = second;
    }

    public boolean equals(Bigram bigram) {
        return bigram.first == this.first &&
                bigram.second == this.second;
    }

    public int hashCode() {
        return 31 * first + second;
    }

    public static void main(String[] args) {
        Set<Bigram> s = new HashSet<>();
        for (int i = 0; i < 10; i++) {
            for (char ch = 'a'; ch <= 'z'; ch++) {
                s.add(new Bigram(ch, ch));
            }
        }

        System.out.println(s.size()); //260
    }
}
```

Bigram이라는 문자 2개를 갖는 클래스에 a~z까지 26개의 문자를 넣고, 10개씩 만든다음에 HashSet에 삽입했다.  
생각해보면 26개가 나올 것 같지만, 실제로는 260개가 발생한다.

왜 그럴까?

HashSet은 내부적으로 equals 메서드를 기반으로 객체의 논리적 동치적(equals) 검사를 실시한다.  
하지만 자세히 보면 equals메서드의 파라미터 타입이 Bigram이다. equals 메서드를 재정의 한게 아니라 Overloading 한 꼴이다.  

equals를 재정의 하려면 파라미터 타입이 Object이어야 한다.

```java
@Override
public boolean equals(Bigram bigram) {
    return bigram.first == this.first &&
        bigram.second == this.second;
}
```

위와 같이 변경하고 컴파일 해보면, 

```
Error:(15, 5) java: method does not override or implement a method from a supertype
```

이러한 컴파일 에러가 발생한다.



잘못된 부분을 명확히 알려주므로 곧장 수정할 수 있다.

```java
@Override
public boolean equals(Object bigram) {
    if(!(o instanceof Bigram)) {
        return false;
    }
    Bigram b = (Bigram) bigram;
    return b.first == this.first &&
        b.second == this.second;
}
```



# 요약

* 상위 클래스의 메서드를 재정의 하는 모든 메서드에 @Override 애너테이션을 달자
* 굳이 @Override를 달지 않아도 동작은 한다. 하지만 일괄적으로 붙여주는게 좋다.
* 인터페이스를 상속한 구체 클래스인데 아직 구현하지 않은 추상 메서드가 남아있다면,  
  컴파일러가 바로 사실을 알려준다.
* Java 8 부터 Default 메서드의 사용이 가능해 지면서, 인터페이스의 메서드를 재정의 할 때도 사용할 수 있다.
* 구현하려는 인터페이스에 Default 메서드가 없음을 안다면 @Override를 생략해 코드를 조금 깔끔히 유지해도 좋다.
* 웬만하면 추상클래스나 인터페이스에서는 상위 클래스나 상위 인터페이스를 재정의하는 모든 메서드에 @Override를 다는것이 좋다.  
  (실수 했을 때 컴파일러가 알려준다.)



# 참고

* Effective Java 3rd Edition - Item 40. @Override 애너테이션을 일관되게 사용하라