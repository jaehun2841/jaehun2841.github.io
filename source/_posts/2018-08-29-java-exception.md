---
title: Java Exception
catalog: true
date: 2018-08-30 00:07:03
subtitle: Java의 기본 예외처리
header-img:
Categories:
 - Java
tags: 
 - Java
typora-root-url: ./2018-08-29-java-exception
typora-copy-images-to: ./2018-08-29-java-exception
---


# 들어가며

Java/Spring 기반의 코드를 작성하다보면, 예외 처리는 필수적이고 늘상 마주하게 되는 문제이다.
예외처리가 견고한 프로그램은 유지보수하기 좋을 뿐만 추가적인 요구사항이 발생하였을 때, 처리하기 쉬워지고 편해진다. 최근 업무 중에 Spring Interceptor 단계에서 예외처리를 구현해야 했다. 그때 사용한 @ControllerAdvice, @ExceptionHandler 등의 어노테이션를 사용하여 쉽게 예외처리를 구현 할 수 있었고, 그에 관련된 추가적인 내용을 기록해보고자 2장의 포스팅을 작성하게 되었다.
이번 장에서는 Java의 기본 예외처리에 대한 내용을 리마인드 해보는 포스팅을 작성하도록 하겠다.



# 에러(Error)와 예외(Exception)

![Exception-Class](./Exception-Class.png)

Java에서는 3가지 종류의 Throwable을 제공한다. 크게는 Error와 Exception으로 구분한다.
Exception은 또 2가지 종류로 구분 할 수 있다.

* Error 
  * 주로 프로그램이 사용 불가능한 상태가 되었을 때 발생한다.
  * 주로 `복구 불가능한 상황`에 대해 사용한다.
  * 주로 JVM 자원 부족이나, 불변식을 사용하는 경우에 발생한다.
  * 관례적으로 Error에 대해서는 Custom Class를 만들지 않는다.
    (어플리케이션 레벨에서 발생하므로 이미 지정된 Error만으로도 충분 - 개발자가 관여x)
* Checked Exception 
  * 점검지정 예외
  * Checked Exception이 발생하는 메소드를 사용하는 경우에는 개발자는 반드시 예외처리에 대한 로직을 구현해야한다.
  * 예외처리를 하지 않을 시, `컴파일 오류가 발생`
  * 주로 `복구 가능한 상황`에 Checked Exception을 사용한다.
  * Custom Checked Exception을 만들 경우 `Exception` 클래스를 상속하여 만든다.
* Unchecked Exception
  * 무점검 예외
  * 프로그래밍 오류를 표현하는 경우 사용
  * 개발자가 굳이 예외처리를 할 필요가 없으며, 메소드 시그니처에 throws를 이용하여 상위 로직으로 Exception을 throw할 필요가 없다.
  * 주로 `복구 불가능한 상황`에 대해 사용한다.
  * Custom Unchecked Exception을 만들 경우 `RuntimeException` 클래스를 상속하여 만든다



## 추가적으로..

개발자 정의 Exception을 생성하다 보면은 주로 RuntimeException으로 많이 생성하게 된다.
아무래도 발생 즉시 시스템 실행 스택을 중단 할 수 있으며, 명시적인 throw처리를 안해줘도 되기 때문에 편리함 때문에 사용하는 것 같다. 최근 면접관으로 들어가서 예외처리에 대한 질문을 하면 Checked Exception은 절대로 쓰면 안된다! 라고 대답을 하는 지원자들이 있었다. Checked Exception을 절대 쓰지 말자? 이는 Exception의 성질을 모르는 사람이 하는 소리라고 생각한다.
Checked Exception으로 처리하는 이유는 API를 사용하여 개발하는 개발자가 반드시 이 단계에서 예외에 대한 처리를 하여 대행 할 수 있는 로직을 추가 하던, 아예 throw를 시켜 중지 하던 개발자에게 어느정도 선택의 기회를 주는 것이라고 생각한다.
지원자가 했던 "Checked Exception 절대 쓰지 말자!" "Checked Exception이 발생하더라도 Unchecked Exception으로 Wrapping 하자!"라는 의견도 어느정도는 맞는 소리이다.
새로운 Checked Exception이 발생하는 코드가 메소드 내의 메소드의 내의 메소드의 내의....로 간다고 하면 밑에서 부터 계속 throw..throw.. 해야 하는 상황이 발생 할 수 있다. 이러한 Exception에 대해서 전역적인 처리를 하는 경우에는 위의 말이 당연히 맞는 소리이다. 
다시 한번 Exception의 성질을 생각해 보고, 왜 Java가 굳이 Checked Exception, Unchecked Exception을 구분했는지를 생각을 해봐야 할 것이다.



# Java에서의 기본적인 예외처리

## try-catch-finally

try-catch-finally 구문은 웬만한 프로그래밍 언어에서 지원하는 구문이다.
프로그래밍 기본을 공부한 사람이라면 모두가 알고 있는 구문이다.

~~~java
try {
    //핵심 로직 수행
} catch(Exception e) {
    //예외 발생 시, 예외에 대한 처리
} finally {
   //try block, catch block 실행 후 반드시 실행하는 로직
   //주로 자원에 대한 해제 로직이 추가된다. 
}
~~~



## try-catch-resources

Java 1.7에 추가된 문법이다. Closeable Interface를 구현하는 객체에 대한 Resource해제를 자동으로 할 수 있도록 해 준다.

~~~java
try (InputStream is = new FileInputStream(file)) { 
     //try block이 종료 되면 InputStream에 대한 close() 메소드를 실행
     //핵심 로직 수행
} catch (Exception e) {
    //예외 발생 시, 예외에 대한 처리
}
~~~



## Multi catch

Java 1.7에 추가된 문법이다. 기존에는 catch 구문을 여러개 둘 수 있었다. Java 1.7부터는 하나의 catch문에서 여러개의 Exception에 대한 처리를 공통적으로 할 수 있다.

~~~java
try (InputStream is = new FileInputStream(file)) { 
     //try block이 종료 되면 InputStream에 대한 close() 메소드를 실행
     //핵심 로직 수행
} catch (NullPointerException | ArrayIndexBoundException e) {
    //예외 발생 시, 예외에 대한 처리
}
~~~



# Exception Handling

Checked Exception가 발생되는 메소드를 사용되는 경우 IDE에서는 개발자에게 2가지 중 하나의 행위를 하라고 한다.

* Method signiture에 throws 구문을 통해 예외처리를 상위 메소드로 전파하여 예외처리를 미루기
* try-catch구문을 통한 예외처리를 하도록 유도

보통의 경우에는 try-catch 구문을 통해서 해결 가능한 에러에 대한 처리로직을 개발자가 구현 해주면 된다.
하지만 Service Layer에서 예외처리를 하는 경우에는 2가지를 모두 다 사용하는게 좋다고 본다.



# Anti Pattern

1. Exception을 무시 하지 말 것
2. exception.printStackTrace()는 쓰는게 아니다.
3. 반복문 내에서는 Checked Exception에 대한 처리는 지양하자



## Exception을 무시 하지 말 것 

간혹 코드를 보다 보면..

~~~java
try {
    //열심히 작성
    veryHardDo();
} catch (Exception e) {
    //아무것도 안해요~
}
~~~

이런 코드를 보는 경우가 있다. 일단 veryHardDo() 메소드에서 Exception을 던져 주니 컴파일 에러가 안나려면 뭔가 처리는 해야 겠고.. 막상 catch구문에 작성할 코드는 없어보이고... 할 때 저런 코드들이 나오게 된다.
이는 매~~우 좋지 않은 코드이다. 실제 RuntimeException이 발생 했을 때도 에러 파악도 안되고, 시스템은 작동 안하고.. 왜 에러나는지 모르는 코드가 되어버린다. 
되도록이면 catch구문을 작성하였으면 반드시 예외처리에 대한 로직을 구현 해주자.. (로그라도 남겨줘 ㅜㅜ)



## exception.printStackTrace()는 쓰는게 아니다.

exception.printStackTrace()는 에러가 발생한 원인에 대한 Stack Trace를 추적하여 개발자에게 디버깅을 할 수 있는 힌트를 제공해 준다 (개발 할 때는 이거만한 게 없긴하다.)
하지만 성능을 생각하는 개발자라면 이 메소드는 사용을 지양해야 한다. 실제로 java reflection을 사용하여 trace를 추적하는 것이기 때문에 꽤 오버헤드가 발생한다. 성능이 중시되는 어플리케이션이라면 stackTrace에 대한 추적 보다는 간단한 로그정도로 에러의 원인을 파악하는 것이 좋다.



## 반복문 내에서는 Checked Exception에 대한 처리는 지양하자

~~~java
for (String item : items) {
    try {
        insert(item);
    }catch (SQLException e) {
        e.printStackTrace();
    }
}
~~~

반복문 내에서 Checked Exception에 대한 예외처리 구문이 들어가게 되면 성능은 2배 3배 떨어지게 된다.
이러한 경우에는 insert에서 예외 발생 시, RuntimeException으로 한번 Wrapping하여 Exception이 발생 되도록 하고 반복문 내에서는 최대한 예외처리에 대한 코드를 제거하는 것이 성능 상 유리하다.



# 참조

Effective Java 2nd Edition (Joshua Bloch)
가장 빨리 만나는 자바8 (카이 호스트만)