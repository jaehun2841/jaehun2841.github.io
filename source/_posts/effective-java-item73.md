---
title: Item 73. 추상화 수준에 맞는 예외를 던지라
catalog: true
Categories:
  - Effective-Java
tags:
  - Effective-Java
typora-root-url: effective-java-item73
typora-copy-images-to: effective-java-item73
date: 2019-03-10 19:31:02
subtitle:
header-img:
---

# 서론

수행하려는 일과 관련 없어 보이는 예외가 튀어나오면 당황스럽다.  
메서드가 저수준 예외를 처리하지 않고 바깥으로 throw 해버릴 때 상위 메서드에서 종종 발생하는 일이다.  
내부 구현방식을 상위에 드러내어 윗 레벨 API를 오염 시킬 수 있고, 다음 릴리스에서 구현방식이 변경되면 다른 예외가 튀어나와  
기존 클라이언트 프로그램을 깨지게 할 수도 있다.



# 상위 메서드에서 저수준 예외를 번역해야 한다.

상위 메서드에서는 저수준 예외를 잡아 자신의 추상화 수준에 맞는 예외로 바꿔 던져야 한다.  
이를 예외 번역(Exception Translation)이라 한다.

```java
try {
	..// 저수준 추상화를 이용
} catch (LowerLevelException e) {
  // 추상화 수준에 맞게 번역
  throw new HigherLevelException(...);
}
```



# 저수준 예외의 내용이 필요하다면 예외 연쇄를 사용하라

예외 연쇄(Exception chaining)이란 문제의 근본 원인(cause)인 저수준 예외를 고수준 예외에 실어 보내는 방식이다.  
별도의 접근자 메서드(Throwable의 getCause메서드)를 통해 필요하면 언제든 저수준 예외를 꺼내 볼 수 있다.

```java
try {
  // 저수준 추상화를 이용한다.
} catch (LowerLevelException cause) {
  // 저수준 예외를 고수준 예외에 실어 보낸다.
  throw new HigherLevelException(cause);
}
```

```java
class HigherLevelException extends Exception {
  HigherLevelException(Throwable cause) {
    super(cause);
  }
}
```

대부분의 표준 예외는 예외 연쇄용 생성자를 갖추고 있다.  
그렇지 않은 예외라도 Throwable의 initCause 메서드를 이용해 `원인` 을 직접 못박을 수 있다.  
예외 연쇄는 문제의 원인을 프로그램에서 접근할 수 있게 해주며 원인과 고수준 예외의 Stack trace를 잘 통합해준다.



# 예외 번역을 남용하지 말자

가능하다면 저수준 메서드가 반드시 성공하도록 하여 아래 계층에서는 예외가 발생하지 않도록 하는 것이 최선이다.  
때로는 상위 계층 메서드의 매개변수 값을 아래 계층 메서드로 건네기 전에 Validator를 이용하여 미리 검사하는 것이 좋다. 

아래 계층에서의 예외를 피할 수 없다면, 상위 계층에서 그 예외를 조용히 처리하여 문제를 API 호출자에 전파하지 않는 방법이 있다.  
이 경우 발생한 예외는 log를 활용하여 개발자가 디버깅을 할 수 있는 정도면 충분하다

```java
try {
  lowerLevelMethod();
} catch (Exception e) {
  //Exception을 먹어버리고 상위로 전파되지 않도록 한다.
  log.error(e.getMessage());
}
```



# 정리

* 아래 계층의 예외를 예방하거나 스스로 처리할 수 없고, 그 예외를 상위 계층에 그대로 노출하기 곤란하다면 예외 번역을 사용하라
* 예외 연쇄를 이용하면 상위 계층에는 맥락에 어울리는 고수준 예외를 던지면서 근본 원인도 함께 알려주어 오류를 분석하기에  좋다.



# 참고

* Effective Java 3rd Edition - Item 73. 추상화 수준에 맞는 예외를 던지라