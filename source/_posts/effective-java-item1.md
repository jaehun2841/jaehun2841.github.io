---
title: Item 1. 생성자 대신 정적 팩터리 메서드를 고려하라
catalog: true
Categories:
  - Effective-Java
tags:
  - Effective-Java
typora-root-url: effective-java-item1
typora-copy-images-to: effective-java-item1
date: 2019-01-06 17:42:25
subtitle:
header-img:
---

# 서론
개발자가 클래스의 인스턴스를 생성하는 방법은 new 키워드를 통한 생성자 호출이다.  
이외에도 생성자와 별도로 정적 팩터리 메서드를 이용해서 메서드 내부에서 생성자나, 이미 생성된 인스턴스를 반환하여 클라이언트 코드에서 인스턴스를 사용 할 수 있다.

# 정적(Static) 팩터리 메서드가 좋은 점
## 이름을 가질 수 있다.
생성자의 파라미터 시그니처만으로는 어떤 객체를 반환 할 지에 대한 특성을 이해하기 어렵다.  
하지만, 정적 팩터리 메서드의 경우는 메서드 이름으로 충분히 유추 할 수 있기 때문에 가독성이 좋아진다.


## 호출 될 때 마다 인스턴스를 생성하지 않아도 된다.
new를 통해 인스턴스를 생성하게 되면, 불필요한 중복 객체를 생성할 가능성이 많이진다.  
하나의 객체를 이용해 캐싱하고 쓰는 경우 정적 팩터리 메서드를 사용하는 게 장점이 될 수 있다.

대표적으로 Boolean에 대한 예시가 있다.  
매번 new Boolean(false)처럼 사용을 하면 사용할 때 마다 중복되는 Boolean 객체를 생성하게 된다.  

하지만 대충 이런식으로 정적 팩터리 메서드를 제공하는 경우
```java
Class Boolean{
  private static final Boolean FLASE = new Boolean(false);
  private static final Boolean.TRUE = new Boolean(true);

  public static valueOf(boolean b) {
    return b ? TRUE : FALSE;
  }
}
```
쓸 데 없는 boolean 객체를 만들지 않아도 되기 때문에 메모리 관리 측면에서 유리하다.  
또한 객체를 싱글턴(singleton)으로 제공할 수 있고, 인스턴스화 불가로 만들 수 도 있다.  
> 반복되는 요청에 같은 객체를 반환하는 식으로 정적 팩터리 방식의 클래스는 언제 어느 인스턴스를 살아 있게 할지를 철저히 통제 할 수 있다.  
이런 클래스를 **인스턴스 통제(instance-controlled) 클래스**라 한다.


## 반환타입의 하위타입 객체(Child)를 반환 할 수 있다.
인터페이스를 사용해 하위타입 객체를 반환 할 수 있다. 인터페이스 기반 프레임워크의 핵심 기술이라고 볼 수 있다.  
SpringFramework를 사용하는 경우에도 유용하게 사용 할 수 있다.  
아래와 같은 예시 코드를 보자

```java
private PaymentService kakaoPaymentService;
private PaymentService naverPaymentService;
private PaymentService paycoPaymentService;
private PaymentService rocketPaymentService;

public PaymentService getType(PaymentType payentType) {
  case KAKAO: return kakaoPaymentService;
  case NAVER: return naverPaymentService;
  case PAYCO: return paycoPaymentService;
  case ROCKET: return rocketPaymentService;
  default: throw new IllegalArgumentException();
}
```
외부 결제 모듈을 사용한다고 가정 했을 때 이런식으로 Interface를 통한 하위타입 객체를 제공할 수 있다.

## 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환 할 수 있다.
EnumSet 클래스는 public 생성자 없이 오직 정적 팩터리 메서드만 제공한다.  
OpenJDK에서는 원소가 64개 이하이면 원소들을 long변수 하나로 관리하는 RegularEnumSet의 인스턴스를 리턴하고,  
65개 이상이면 long배열로 관리하는 JumboEnumSet의 인스턴스를 반환한다.

클라이언트는 EnumSet 객체이면 되기 때문에 무슨객체가 리턴되든 알 필요가 없다. 

## 정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.
이러한 유연함은 Service Provider 프레임워크의 근간이 된다.
대표적인 예로 JDBC가 있다.
1. DriverManager.registerDriver() 메서드로 각 DBMS별 Driver를 설정한다. (제공자 등록 API)  
2. DriverManager.getConnection() 메서드로 DB 커넥션 객체를 받는다. (service access API)
3. Connection Interface는 DBMS 별로 동작을 구현하여 사용할 수 있다. (service Interface)

위와 같이 동작하게 된다면 차후에 다른 DBMS가 나오더라도 같은 Interface를 사용하여 기존과 동일하게 사용이 가능하다.

# 정적(Static) 팩터리 메서드의 단점
## 상속을 하려면 public/protected 생성자가 필요하다.
상속을 하기 위해서는 public/protected 생성자가 필요하니, 정적 팩터리 메서드만 제공하면 하위클래스를 만들 수가 없다.
> 컬렉션 프레임 워크의 유틸리티 구현 클래스는 private 생성자만 제공하므로 상속이 불가하다.  
이러한 제약은 상속보다 컴포지션을 사용하도록 유도되어 오히려 더 장점으로 작용한다.

## 정적 팩터리 메서드는 개발자가 찾기 어렵다.
생성자 처럼 API 설명에 명확히 드러나지 않으니 사용자는 정적 팩터리 메서드 방식 클래스를 인스턴스화 할 방법을 찾아야 한다.

# 정적(static) 팩터리 메서드 명명 규칙
* from - 매개변수를 하나만 받아서 해당 타입의 인스턴스를 반환하는 메서드
  * 예시) Date date = Date.from(dateStr);
* of - 여러 매개변수를 받아 적합한 타입의 인스턴스를 반환하는 집계 메서드
  * 예시) Set<Rank> faceCards = EnumSet.of(JACK, QUEEN, KING);
* valueOf - from과 of의 더 자세한 버전
  * BigInteger prime = BigInteger.valueOf(Integer.MAX_VALUE);
* instance(getInstance) - 매개변수를 받는다면, 매개변수로 명시한 인스턴스를 반환하지만, 같은 인스턴스임을 보장하지 않는다.
  * StackWalker luke = StackWalker.getInstance(options);
* create(newInstance) - instance/getInstance와 같지만, 매번 새로운 인스턴스를 반환함을 보장한다.
* get(Type) - getInstance와 맥락은 같으나 특정 Type을 반환할 때 사용
  * Steak steak = Food.getSteak(Meet.BEEF);
* new(Type) - newInstance와 같으나, 생성할 클래스가 아닌 다른 클래스의 팩터리 메서드를 정의 할 때 사용
  * Steak steak = Food.newSteak(Meet.BEEF);
* (Type) - getType, newType의 같결한 버전
  * Steak steak = Food.steak(Meet.BEEF);

# 요약
정적 팩터리 메서드는 각각의 쓰임새가 있으니, 장단점을 잘 인식하고 쓰는게 좋다.  
대부분의 경우가 정적 팩터리 메서드로 인스턴스를 생성하는게 유리하니 무작정 public 생성자만 사용하는 습관은 고치는게 좋다.
# 참고
* Effective Java 3rd Edition - Item 1. 생성자 대신 정적 팩터리 메서드를 고려하라