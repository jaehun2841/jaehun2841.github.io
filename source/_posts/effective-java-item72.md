---
title: Item 72. 표준 예외를 사용하라
catalog: true
Categories:
  - Effective-Java
tags:
  - Effective-Java
typora-root-url: effective-java-item72
typora-copy-images-to: effective-java-item72
date: 2019-03-10 18:58:11
subtitle:
header-img:
---

# 서론

숙련된 프로그래머는 그렇지 못한 프로그래머보다 더 많은 코드를 재사용한다.  
예외도 마찬가지로 재사용하는 것이 좋으며, 자바 라이브러리는 대부분 API에서 쓰기에 충분한 수의 예외를 제공한다.



# 표준 예외를 재사용하라

* 표준 예외를 사용하면 다른사람이 API를 익히고 사용하기 쉬워진다 (많은 개발자들이 이미 익숙하게 사용하기 때문)
* 표준 예외를 사용한 API는 다른 개발자가 API를 사용하더라도 낯선 예외를 사용하지 않아 코드의 가독성이 높아진다.
* 예외 클래스 수가 적을수록 메모리 사용량도 줄고 클래스를 로딩하는 시간도 적게 걸린다.  
  (무분별하게 커스텀 예외 클래스를 만들면 빌드하는 시간도 오래 걸리고 클래스 로딩 시간도 더 걸린다는 소리)



# 가장 많이 사용되는 예외


## IllegalArgumentException

* 호출자가 인수로 부적절한 값을 넘길 때 던지는 예외
* 반복 횟수 (loop count)를 지정하는 매개변수에 음수를 건넬 때 쓸 수 있다.
* 메서드의 파라미터로 `null 값`이 들어오면 관례상 IllegalArgumentException보다는 `NullPointerException`을 던진다.
* 시퀀스의 허용 범위를 넘는 값을 건넬 때도 IllegalArgumentException보다는 `IndexOutOfBoundsException`을 던진다.



## IllegalStateException

* 대상 객체의 상태가 호출된 메서드를 수행하기에 적합하지 않을 때 주로 던진다.
* 예를 들면, 초기화되지 않은 객체를 사용하려 할 때 던질 수 있다.



## ConcurrentModificationException

* 단일 스레드에서 사용하려고 설계한 객체를 여러 스레드가 동시에 수정하려 할 때 던진다.  
  (외부 동기화 방식으로 사용하려고 설계한 객체도 마찬가지다.)
* 동시 수정을 확실히 검출할 수 있는 안정된 방법은 없으나, 문제가 생길 가능성 정도만 알려주는 역할도 쓰인다.



## UnsupportedOperationException

* 예외는 클라이언트가 요청한 동작을 대상 객체가 지원하지 않을 때 던진다.
* 보통 구현하려는 인터페이스의 메서드 일부를 구현할 수 없는 경우에 사용



## ArithmeticException, NumberFormatException

* 복소수나 유리수를 다루는 객체를 사용할 때 사용
* 10 / 0 과 같은 연산을 할 때 ArithmeticException이 발생한다. 
* 숫자형 파라미터가 와야 하는 부분에 String이라든지 다른 형식의 데이터가 들어오는 경우 NumberFormatException이 발생



# 예외 사용 시 주의할 점

* Exception, RuntimeException, Throwable, Error는 직접 재사용하지 말자.  
  (다른 예외들의 상위 클래스이기 때문에 안정적으로 테스트 할 수 없다.)
* 예외에서 더 많은 정보를 제공하길 원한다면 표준 예외를 확장해도 좋다.  
  (단 예외는 직렬화할 수 있기 때문에 주의해야 한다.)



# 참고

* Effective Java 3rd Edition - Item 72. 표준 예외를 사용하라