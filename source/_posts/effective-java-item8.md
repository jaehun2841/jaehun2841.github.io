---
title: Item 8. finalizer와 cleaner 사용은 피하라
catalog: true
Categories:
  - Effective-Java
tags:
  - Effective-Java
typora-root-url: effective-java-item8
typora-copy-images-to: effective-java-item8
date: 2019-01-08 00:32:01
subtitle:
header-img:
---

# 서론
Java에서는 2가지의 객체 소멸자를 제공한다.
* finalzier
* cleaner

이 2가지 객체 소멸자는 JVM내의 GC가 작동할 때 실행되는 구문이다.  
기본적으로 이 2가지 구문은 **사용하지 말아야 한다.**  
현재 Java 9 버전 부터는 Object 클래스에 대한 finalizer를 `Deprecated` 처리하였다.  
아래에 사용하지 말아햐 하는 이유를 하나씩 보도록 하겠다.

# finalizer와 cleaner를 사용하지 말아야 하는 이유
## 실행을 보장할 수 없다.
finalizer에 특정 로직을 삽입하는 경우 실행을 보장 할 수 없다.  
기본적으로 GC가 발생할 때 실행되는 로직이지만, Java Application이 죽는다던지의 이유로 finalizer 실행이 되지 않을 수 있다.   
그렇기 때문에 제 때 실행되어야 하거나, 뭔가 상태를 수정하는 행위를 절대적으로 하면은 안된다.

## 느리다.
Effective Java 책의 예제에서는 AutoCloseable과 finalizer의 성능비교를 한 문단이 있다.  
Autocloseable을 사용한 GC 수행시간은 12ns였지만, finalizer를 사용한 GC 수행 시간은 550ns가 걸렸다고 한다.  
finalizer가 가비지 컬렉터의 효율을 떨어뜨리기 때문이다.

## 시스템 전체 장애를 유발 할 수 있다.
Java API 문서 상에서는 GC가 UnReachable 상태의 객체를 가비지 컬렉션 할 때 finalizer가 호출된다고 명시하고 있다.  
하지만 finalizer가 실행되는 시점은 GC 발생 시 즉각적으로 이루어지는게 아니다.  
finalizer queue에 삽입되어 순차적으로 finalizer를 실행하게 된다.  
그렇기 때문에 finalizer메소드 실행이 느린 경우 객체 소멸이 느려지므로  `Out of Memory`와 같은 오류를 발생 시킬 가능성이 높아지게 된다.

## finalizer 공격에 취약하다.
위에 적은 것 처럼 finalizer 메소드 실행시간이 오래 걸리게 만들면 시스템에 심각한 장애를 유발할 수 있다.  
finalizer 메서드를 override해서 악의적으로 느리게 할 수 있기 떄문에 finalizer를 사용해야 하는 경우라면  
메소드에 `final` 키워드를 붙여서 상속하지 못하도록 해야한다.

# 그럼 finalizer나 cleaner는 어디서 쓰나?
* 개발자가 객체의 close를 호출하지 않는 경우 -> 자원 해제를 안하느니 느리더라도 하는게 낫다.  
(이 경우는 동의 못하겠다. 개발자가 close를 시켜야 한다.)
* native peer와 연결된 객체
  * native peer는 자바 객체가 아니기 떄문에 가비지 컬렉터의 관리 대상이 아니다.  
  그렇기 때문에 native peer가 회수 될때 finalizer 메서드를 실행해 자원을 해제 할 수 있다.

# finalizer 기능이 필요한 경우에는 어떻게?
Autocloseable Interface를 사용하여 close를 호출시키도록 하자.  
다음장의 try-catch-resource 구문에서 설명하도록 하겠다.

# 참고
* Effective Java 3rd Edition - Item 8. finalizer와 cleaner 사용은 피하라
* [finalize 메소드의 오버라이딩을 자제해야 하는 이유.](http://www.yunsobi.com/blog/entry/finalize-%EB%A9%94%EC%86%8C%EB%93%9C%EC%9D%98-%EC%98%A4%EB%B2%84%EB%9D%BC%EC%9D%B4%EB%94%A9%EC%9D%84-%EC%9E%90%EC%A0%9C%ED%95%B4%EC%95%BC%ED%95%98%EB%8A%94-%EC%9D%B4%EC%9C%A0)