---
title: Item 74. 메서드가 던지는 모든 예외를 문서화하라
catalog: true
Categories:
  - Effective-Java
tags:
  - Effective-Java
typora-root-url: effective-java-item74
typora-copy-images-to: effective-java-item74
date: 2019-03-23 20:07:57
subtitle:
header-img:
---

# 서론

메서드가 던지는 예외는 그 메서드를 올바로 사용하는 데 아주 중요한 정보다.  
따라서 각 메서드가 던지는 예외 하나하나를 문서화하는 데 충분한 시간을 쏟아야 한다.  



# 검사 예외는 @throws 태그로 문서화하라

검사 예외(Checked Exception)는 항상 따로따로 선언하고, 각 예외가 발생하는 상황을 자바독의 @throws 태그를 사용하여 정확히 문서화하자.  
공통 상위 예외 클래스 하나로 뭉뚱그려 선언하는 일은 삼가야 한다.  
극단적인 예로 Exception이나 Throwable을 던진다고 선언해서는 안된다.  
메서드 사용자에게 각 예외에 대처할 수 있는 힌트를 주지 못할뿐더러 같은 맥락에서 발생할 여지가 있는 다른 예외들 까지 삼켜버릴 수 있기 때문에 API 사용성을 크게 떨어뜨린다.



## 잘못된 방법
```java
public void method() throws Exception {
}
```

* 같은 맥락에서 발생할 수 있는 다른 예외들까지 삼켜버려 API 사용성이 떨어진다.
* main메서드는 JVM만이 호출하므로 Exception을 던지도록 선언해도 괜찮다.



## 권장하는 방법

```java
/**
 *
 * @throws IllegalStateException
 * @throws SQLException
 * @throws NumberFormatException - params가 숫자형 데이터가 아닌경우 throw
 */
public void method(String params) throws IllegalStateException, SQLException, NumberFormatException {
  
}
```



# 비검사 예외(Runtime Exception)도 문서로 남기면 좋다.

자바 언어에서 요구하는 것은 아니지만 비검사 예외(Runtime Exception)도 검사 예외(Checked Exception) 처럼 정성껏 문서화 해두면 좋다.  
비검사 예외(Runtime Exception)는 일반적으로 프로그래밍 오류를 뜻하는데 자신이 일으킬 수 있는 오류들이 무엇인지 알려주면   
프로그래머는 자연스럽게 해당 오류가 나지 않도록 코딩하게 된다.  
잘 정비된 비검사 예외 문서는 그 메서드를 성공적으로 수행하기 위한 전제조건이 된다.

public 메서드라면 필요한 전제조건을 문서화해야 하며, 그 수단으로 가장 좋은 것이 비검사 예외들을 문서화 하는것이다.  
**특히 인터페이스에서 중요하다.**  
이 조건이 인터페이스의 일반 규약에 속하게 되어 인터페이스를 구현한 모든 구현체가 일관되게 동작하도록 해주기 때문이다.  



# 비검사 예외(Runtime Exception)은 메서드 시그니처에 추가하지 말자

메서드가 던질 수 있는 예외를 각각 @throws 태그로 문서화하되, 비검사 예외는 메서드 선언의 throws 목록에 넣지 말자.  
검사냐 비검사냐에 따라 사용자가 해야할 일이 달라지므로 이 둘을 확실히 구분하는게 좋다.  
자바독은 메서드 시그니처에 throws 절에 등장하고 메서드 주석의 @throws 태그에도 명시한 예외와 @throws 태그에만 명시한 예외를 시각적으로 구분해준다.  
그래서 프로그래머는 어떤 것이 비검사 예외인지 바로 알 수 있다.



# 거의 모든 메서드에서 같은 예외를 던진다면 Class 설명에 추가하라

한 클래스에 정의된 웬만한 메서드에서 같은 이유로 같은 예외를 던진다면 그 예외를 각각의 메서드가 아니라 클래스 설명에 추가하는 방법도 있다.  
NullPointerException이 가장 흔한 사례다.  
이럴 때는 클래스의 문서화 주석에 **이 클래스의 모든 메서드는 인수로 null이 넘어오면 NullPointerException을 던진다.** 라고 적어도 좋다.



# 참고

* Effective Java 3rd Edition - Item 74. 메서드가 던지는 모든 예외를 문서화하라