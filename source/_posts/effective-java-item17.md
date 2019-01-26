---
title: Item 17. 변경 가능성을 최소화하라
catalog: true
Categories:
  - Effective-Java
tags:
  - Effective-Java
typora-root-url: effective-java-item17
typora-copy-images-to: effective-java-item17
date: 2019-01-20 17:53:58
subtitle:
header-img:
---

# 불변 클래스(immutable class)란?
* 인스턴스의 내부 값을 수정할 수 없는 클래스를 말한다.  
* 불변 클래스의 인스턴스는 객체가 생성되는 시점에 초기화되고 소멸이 될 때까지 값이 변경되지 않는다.  
* 불변 클래스는 가변 클래스보다 설계하고 구현하고 사용하기 쉽다.
* 값이 변경되지 않으니 오류가 발생할 일도 적고 안전하다.

# 불변 클래스를 만들기 위한 5가지 조건
## 객체의 상태를 변경하는 메서드(변경자)를 제공하지 않는다.
* 쉽게 말해 setter나, 필드 정보를 변경하는 메서드를 제공하지 않는다.
## 클래스를 확장 할 수 없도록 한다.
* 클래스를 final로 선언한다.
* 모든 생성자는 private으로 만들고 생성자 정적 팩터리 메서드를 제공한다.
* 정적 팩터리 메서드는 유연성을 제공한다.
* 다음 릴리즈에서 Boolean처럼 `캐싱`을 이용해 성능을 끌어올릴 수도 있다.

```java
@NoArgsConstructor(access = AccessLevel.PRIVATE)
@AllArgsConstructor(access = AccessLevel.PRIVATE, staticName="of")
public class Member {
  private String name;
  private int age;
}
```
## 모든 필드를 final로 선언한다.
**내용 수정 필요!!!! final도 된다!!!!**
```java
@AllArgsConstructor(access = AccessLevel.PRIVATE, staticName="of")
public class Member {
  private final String name;
  private final int age;
}
```
* 필드에 대해 수정을 막겠다는 설계자의 의도를 보여준다. - final클래스는 생성자에서 딱 `1회 초기화` 할 수 있기 때문이다.
* 인스턴스에 대한 동기화(syncronized)처리 없이도 다른 스레드에서 문제없이 처리 된다.

## 모든 필드를 private로 선언한다.
* 필드가 참조하는 가변 객체를 클라이언트에서 직접 접근하여 수정하는 일을 막는다.
* public final로만 처리해도 되지만, 그 안의 내용을 바꿀 수 있고 다 릴리즈에서 내부 표현을 변경하지 못하므로 사용하지 않는게 좋다.

## 자신(객체) 이외에는 내부의 가변 컴포넌트에 접근 할 수 없도록 한다.
* 클라이언트에서 인스턴스 내에 가변 객체의 참조를 얻을 수 없게 해야한다. - Collection에 대한 getter를 제공하면 안된다!
* 생성자, 접근자(getter), readObject 메서드에서 모두 `방어적 복사`를 수행해야 한다.

# 함수형 프로그래밍
* 피연산자에 함수를 적용해 결과를 반환하는 프로그래밍 기법
* 피연산자에 대해 연산하지만 피연산자는 값이 변하지 않음 
(캡처 - 피연산자는 사실상 final)
* 코드의 불변이 영역이 되는 비율이 높아져 안전하다.

# 불변 객체의 장점
* 불변 객체는 생성 시점부터 소멸될 때 까지 변하지 않는다. - 서비스 로직에 더 안전하다.
* 불변 객체는 근본적으로 스레드 안전하여 따로 동기화 할 필요가 없다. - 스레드에 따라 값이 변하지 않기 때문이다.
* 불변 객체는 안심하고 공유 할 수 있다. - 불변 클래스라면 최대한 재사용 하라 (캐싱)
* 불변 객체는 방어적 복사본이 필요없다 - 바뀌지 않으므로 방어할 일이 없다.
* 불변 객체는 clone메서드를 제공하지 않는게 좋다. - 의미가 없다.
* 불변 객체를 key로 하면 이점이 많다.
  * Map의 key
  * Set의 원소
* 불변 객체는 그 자체로 실패 원자성을 제공한다.
  * 메서드에서 예외가 발생하더라도 객체의 변경내용이 없다.
* 불변 객체끼리는 자유롭게 내부 데이터를 공유 할 수 있다.
  * BigInteger 클래스에서 mag는 크기를 나타내는 배열
  * signum은 부호를 나타내는 int 필드
  * mag는 배열이지만 불변이므로 방어적 복사본을 만들지 않고 원래 객체의 mag를 써도 된다.
  
```java
    /**
     * Returns a BigInteger whose value is {@code (-this)}.
     *
     * @return {@code -this}
     */
    public BigInteger negate() {
        return new BigInteger(this.mag, -this.signum);
    }
```
* 불변 객체는 값이 다르면 반드시 독립된 객체로 만들어야 한다.

# 요약
* 접근자(getter)를 제공한다고 습관처럼 수정자(setter)를 만들지 말자
  * 꼭 필요한 경우가 아니라면 클래스는 불변이어야 한다.
  * 장점이 훨씬많고, 단점이라면 잠재적 성능저하 뿐이다.
* 불변으로 만들 수 없는 클래스라도 변경할 수 있는 부분을 최소한 줄이자
  * 객체가 가질 수 있는 상태를 제한하면, 예외를 예측하기 쉽고, 오류가 생길 가능성이 줄어든다.
  * 꼭 변경할 필드가 아닌 필드는 final로 선언하자 - 웬만하면 final로 해야한다.
* 생성자는 불변식 설정이 모두 완료된, 초기화가 완벽히 끝난 상태의 객체를 생성해야 한다.
  * 확실한 이유가 없다면, 생성자와 정적팩터리 외에는 어떤 초기화 메서드도 public으로 제공해서는 안된다.
  * 특히! 객체를 재활용할 목적으로 상태를 다시 초기화 하는 메서드도 안된다.
  * 복잡성만 커지고 성능 이점은 거의 없다.

# 참고
* Effective Java 3rd Edition - Item 17. 변경 가능성을 최소화하라