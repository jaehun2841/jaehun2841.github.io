---
title: Item 5. 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라
catalog: true
Categories:
  - Effective-Java
tags:
  - Effective-Java
typora-root-url: effective-java-item5
typora-copy-images-to: effective-java-item5
date: 2019-01-07 21:03:48
subtitle:
header-img:
---

# 서론
사용하는 자원에 따라 동작이 달라지는 클래스에는 정적 유틸리티 방식이나, 싱글턴 방식이 적합하지 않다.  
자원에 따라 동작이 달라진다면, 자원의 수만큼 인스턴스를 만들어 동작하게 하는 것이 좋다.  
이를 `의존 객체 주입`이라고 하는데 Spring과 같은 프레임워크에서 DI의 개념으로 많이 쓰이고 있다.

의존 객체 주입은 생성자, 정적팩터리, 빌더 혹은 Setter를 이용해서 자원을 넘겨 줄 수 있다.

# 팩터리 메서드 패턴 사용하여 자원 넘겨주기
자바8에서는 Supplier<T> 인터페이스가 팩터리를 표현한 완벽한 예다.  
Supplier<T>를 입력으로 받는 메서드는 일반적으로 한정적 와일드 카드 타입을 사용해 팩터리의 타입 매개변수를 제한한다.

아래의 예제처럼 사용한다.
타일들을 이용해 모자이크를 만드는 예제이다.
```java
Mosaic create(Supplier<? extends Tile> tileFactory){}
```

# 요약
* 클래스가 내부적으로 하나 이상의 자원에 의존하고, 그 자원이 클래스 동작에 영향을 준다면 싱글턴과 정적 유틸 클래스는 사용하지 말자!
* 필요한 자원 또는 팩터리를 생성자나 빌더를 통해 의존 객체를 주입하도록 하자
* 의존 객체 주입은 유연성, 재사용성, 테스트 용이성을 높여 줄 것이다.

# 참고
* Effective Java 3rd Edition - Item 5. 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라
