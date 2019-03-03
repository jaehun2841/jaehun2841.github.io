---
title: Item 68. 일반적으로 통용되는 명명 규칙을 따르라
catalog: true
Categories:
  - Effective-Java
tags:
  - Effective-Java
typora-root-url: effective-java-item68
typora-copy-images-to: effective-java-item68
date: 2019-03-03 19:12:38
subtitle:
header-img:
---

# 서론

자바 플랫폼은 명명 규칙이 잘 정립되어 있으며, 그중 많은 것이 자바 언어 명세에 기술되어 있다.  
자바의 명명 규칙은 철자와 문법 두 범주로 나뉜다.  
철자 규칙은 패키지, 클래스, 인터페이스, 메서드, 필드, 타입 변수의 이름을 다룬다.  
이 규칙들은 특별한 이유가 없는 한 반드시 따르는게 좋고 이를 어기면 다른 프로그래머들이 그 코드를 읽기 번거로울 뿐 아니라 다른뜻으로 오해할 수도 있고 그로 인해 장애로 발전할 수 있다.



# 명명 규칙

## 패키지 (Package)
* 패키지 (Package)와 모듈 이름은 각 요소를 **점(.)**으로 구분하여 계층적으로 짓는다.
* 요소들은 모두 소문자 알파벳 혹은 숫자로 이뤄진다.
* com.google, kr.co.edu와 같은 식이다.
* 패키지를 설명하는 하나이상의 요소로 이루어져 있다.
* 일반적으로 8자 이하의 짧은 단어로 한다.
* utilities보다는 util처럼 의미가 통하는 약어가 좋다.
* 요소의 이름은 보통 한 단어 혹은 약어로 이루어진다.
* 많은 기능을 제공하는 애플리케이션의 경우에는 계층을 더 많은 요소로 나누는 것이 좋다.



## 클래스, 인터페이스, 열거타입

* 클래스 명은 하나이상의 단어로 구성되며, 첫글자는 대문자로 작성한다.
* 여러 단어의 첫글자만 딴 약자나 널리 통용되는 줄임말을 제외하고는 줄임말을 쓰지 않도록 한다.
* 조합한 단어를 구분할 수 있게 **camel case**로 작성한다.



## 메서드, 필드명

* 첫 글자를 소문자로 작성하고 클래스 명과 같게 단어 별로 **camel case**로 작성한다.
* 첫 단어가 약자라면 단어 전체가 소문자여야 한다.
* 상수 필드는 예외다. 상수 필드를 구성하는 단어는 모두 대문자로 쓰며 단어 사이는 언더바(_)로 구분한다.  
  (VALUES, NEGATIVE_INFINITY등..)
* 상수 필드는 static final인 타입을 의미한다.
* 가르키는 객체가 불변이라면, 그 타입은 가변이어도 상수이다.
* 지역번수에도 동일한 규칙이 적용된다. 단, 문맥에서 의미를 쉽게 유추할 수 있는 경우에는 **약어를 사용해도 좋다.**  
  (i, denom, houseNum등..)
* 타입 매개변수의 이름은 한 글자로 표현한다.
  * T: 임의의 타입 (Type)
  * E: 컬렉션의 원소 (Element)
  * K: 맵의 키 (Key)
  * V: 맵의 값 (Value)
  * X: 예외 (eXception)
  * R: 메서드의 반환타입 (Return)
  * 그 이외의 타입에는 T, U, V 혹은 T1, T2, T3의 식으로 사용



# 명명 규칙2
* 객체를 생성하는 클래스나 열거타입 인터페이스는 **단수 명사나 명사구를 사용한다.**
  * Thread, PriorityQueue, ChessPiece 등..
* 객체를 생성할 수 없는 클래스 (Utils 클래스)에는 보통 **복수형 명사로 짓는다.**
  * Collectors, Collections 등..
* 인터페이스 이름은 클래스명과 동일하게 짓거나, **ible, able로 끝나는 형용사로 짓는다.**
  * Runnable, Iterable, Accessible 등...
* 애너테이션은 워낙 다양하게 활용되어 지배적인 규칙이 없이 명사, 형용사, 동사, 전치사가 두루 쓰인다.
  * @Binding, @Inject, @ImplementsBy, @Singleton 등..
* 메서드의 이름은 동사나 목적어를 포함한 **동사구로 짓는다.**
  * append, drawImage
* boolean 값을 반환하는 메서드라면 **is~, has~로 짓는다.**
  * isDigit, isEmpty, hasSiblings 등..
* 반환타입이 boolean이 아닌경우 보통 명사, 명사구, get~로 짓는다.
  * size, hashcode, getTime 등...
  * get으로 시작하는 형태는 주로 자바빈즈(JavaBeans) 명세에 뿌리를 두고 있다.
  * 보통 getter/setter의 한 묶음 형태로 만드는 경우가 많다.
* 반환타입을 또다른 타입을 반환하는 경우에는**toType** 의 형태로 짓는다.
  * toString, toArray 등..
* 객체의 내용을 다른 뷰로 보여주는 메서드는 **asType** 의 형태로 짓는다.
  * asList, asMap 등..
* 객체의 값을 기본 타입(primitive type)으로 반환하는 경우에는 **typeValue** 의 형태로 짓는다.
  * intValue, longValue 등...
* 정적 팩터리의 이름은 다양하다
  * from, valueOf, getInstance, newInstance 등.. 



# 정리

* 표준 명명 규칙을 체득하여 자연스럽게 사용하도록 연습하자
* 철자 규칙은 직관적이라 모호한 부분이 적지만, 문법 규칙은 더 복잡하고 느슨하다
* 오랫동안 따라온 규칙과 충돌한다면 그 규칙을 맹종해서는 안된다. 상식대로 가자



# 참고

* Effective Java 3rd Edition - Item 68. 일반적으로 통용되는 명명 규칙을 따르라