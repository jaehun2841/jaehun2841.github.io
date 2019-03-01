---
title: Item 62. 다른 타입이 적절하다면 문자열 사용을 피하라
catalog: true
Categories:
  - Effective-Java
tags:
  - Effective-Java
typora-root-url: effective-java-item62
typora-copy-images-to: effective-java-item62
date: 2019-03-01 19:53:10
subtitle:
header-img:
---

# 서론

문자열(String)은 텍스트를 표현하도록 설계되었고, 그 일을 아주 멋지게 해낸다.  
그런데 문자열은 워낙 흔하고 자바가 또 잘 지원해주어 원래 의도하지 않는 용도로도 쓰이는 경향이 있다.



# 문자열의 안티패턴

## 문자열은 다른 값 타입을 대신하기에 적합하지 않다.

많은 사람들이 파일, 네트워크, 키보드 입력으로부터 데이터를 받을 때 주로 문자열을 이용한다.  
하지만 **입력받을 데이터가 진짜 문자열인 경우에만 문자열을 사용하는 것이 좋다.**  
데이터가 수치형이면 int, long, double등 수치에 대한 타입으로 사용하는 것이 좋다.  
질문의 답이 예/아니오라면 boolean을 사용하는것이 좋다.  

일반화 하여 얘기하자면,  
**기본 타입이든 참조 타입이든 적절한 값 타입이 있다면 그것을 사용하고 없다면 새로 하나 타입을 만드는 것이 좋다.**



## 문자열은 열거 타입 대신하기에 적합하지 않다.

상수를 열거할 경우에는 문자열 열거 패턴 클래스 보다는 열거 타입(enum)이 훨씬 낫다.



## 문자열은 혼합 타입을 대신하기에 적합하지 않다.

여러 요소가 혼합된 데이터를 하나의 문자열로 표현하는 것은 대체로 좋지 않다.

```java
String compoundKey = className + "#" + i.next();
```

이 방식은 단점이 많다.  
예를 들어 문자열에 # 문자열이 있는 경우 혼란스러운 결과가 발생한다.  
각 요소를 개별적으로 접근하기 위해서는 특정 기준을 통해 문자열을 파싱해야해서 느리고, 귀찮고, 오류 가능성도 크다.

이럴바에는 **차라리 전용 클래스를 새로 만들어서 각 데이터 별로 멤버 변수를 취하는 것이 좋다.**



## 문자열은 권한을 표현하기에 적합하지 않다.

권한(capacity)를 문자열로 표현하는 경우가 종종 있다.

### 잘못된 예 - 문자열을 사용하여 권한을 구분하였다.

```java
public class ThreadLocal {
    private ThreadLocal() {} //객체 생성 불가
    
    // 현 스레드의 값을 키로 구분해 저장한다.
    public static void set(String key, Object value);
    
    // (키가 가르키는) 현 스레드의 값을 반환한다.
    public static Object get(String key);
}
```

이 방식의 문제점은 스레드 구분용 문자열 키가 global namespace에서 공유된다는 점이다.  
이 방식이 의도대로 동작하려면 각 클라이언트가 고유한 키를 제공해야 한다.  
만약 두 클라이언트가 서로 소통하지 못해 같은 키를 쓰기로 결정한다면, 의도치 않게 같은 변수를 공유하게 된다.  
따라서 클라이언트는 제대로 작동하지도 않고 보안에도 취약하다.

이런 경우에는 String으로 권한을 구분하는 것이 아니라 별도의 타입을 만들어 해결해야 한다.

```java
public class ThreadLocal {
    private ThreadLocal() {} //객체 생성 불가
    
    public static class Key {
        key() {}
    }
    
    //위조 불가능한 고유 키를 생성한다.
    public static Key getKey() {
		return new Key();
    }
    
    public static void set(Key key, Object value);
    public static Object get(Key key);
}
```

이 방법은 앞서의 문자열 기반 API의 문제점을 해결해 주지만 개선할 부분이 있다.  
set/get 메서드는 이제 static 메서드일 이유가 없다. 따라서 Key의 인스턴스 메서드로 변경하는 것이 좋다.  
그렇게 하면 Key는 더 이상 스레드 지역변수를 구분하기 위한 키가 아니라 그 자체가 스레드 지역변수가 된다.  

```java
public final class Key {
    public Key();
    public void set(Object value);
    public Object get();
}
```

이 API는 get으로 얻은 Object를 실제 타입으로 타입 캐스팅 해야 해서 타입안전하지 않다.  
하지만 제네릭을 사용한다면 조금 더 타입 안전하게 만들 수 있다.

```java
public final class Key<T> {
    public Key();
    public void set(T value);
    public T get();
}
```



# 정리

* 더 적합한 데이터 티입이 있거나 새로 작성할 수 있다면, 문자열을 쓰지 말자
* 문자열은 잘못 사용하면 번거롭고, 덜 유연하고, 느리고 오류 가능성도 크다.
* 문자열을 잘못 사용하는 흔한 예로는 기본 타입, 열거 타입, 혼합 타입이 있다.



# 참고

* Effective Java 3rd Edition - Item 62. 다른 타입이 적절하다면 문자열 사용을 피하라