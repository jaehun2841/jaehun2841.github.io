---
title: Item 65. 리플렉션보다는 인터페이스를 사용하라
catalog: true
Categories:
  - Effective-Java
tags:
  - Effective-Java
typora-root-url: effective-java-item65
typora-copy-images-to: effective-java-item65
date: 2019-03-03 16:57:44
subtitle:
header-img:
---

# 서론

리플렉션 기능(java.lang.reflect)을 이용하면 프로그램에서 임의의 클래스에 접근할 수 있다.  
Class 객체가 주어지면 클래스 정보를 통해 아래와 같은 인스턴스를 가져올 수 있다.

* Constructor
  * 생성자 시그니처를 가져올 수 있다.
  * 생성자 인스턴스를 통해 객체를 생성할 수 있다.
* Method
  * Method 시그니처를 가져올 수 있다.
  * Method 인스턴스를 통해 Method를 실행시킬 수 있다. (Method.invoke)
* Field
  * 필드타입, 멤버필드 이름등을 가져올 수 있다.



# 리플렉션의 단점

리플렉션을 이용하면 컴파일 당시에 존재하지 않던 클래스도 이용할 수 있다.  
(예를들면.. 외부 라이브러리의 클래스를 리플렉션으로 인스턴스를 생성한다든지…)  

## 컴파일타임 타입 검사가 주는 이점을 누릴 수 없다.

* 예외 검사, 컴파일 타임 에러를 잡아낼 수 없다.  
* 프로그램이 리플렉션 기능을 써서 존재하지 않는 혹은 접근 불가능한 (private 메서드)를 호출하려 하면 런타임 오류가 발생한다.



## 리플렉션을 이용하면 코드가 지저분하고 장황해진다.

* 지루한 일이고 읽기도 어렵다.



## 성능이 떨어진다.

* 리플렉션을 통한 메서드 호출은 일반 메서드 호출보다 훨씬 느리다.
* 고려해야 하는 요소가 많아 정확한 차이는 이야기하기 어렵다
* 하지만 분명 느리다.



## 리플렉션은 아주 제한된 형태로만 사용해야 그 단점을 피할 수 있다

* 컴파일타임에 이용할 수 없는 클래스를 사용해야만 하는 프로그램은 비록 컴파일타임이라도 적절한 인터페이스나 상위 클래스를 이용할 수는 있을것이다.
* 리플렉션은 인스턴스 생성에만 쓰고 이렇게 만든 인터페이스나 상위 클래스로 참조해 사용하자



# 리플렉션의 취약한 예

```java
public static void main(String[] args) {
    
    // 클래스 이름을 Class 객체로 변환
    Class<? extends Set<String>> cl = null;
    try {
        cl = (Class<? extends Set<String>>) Class.forName(args[0]); //비검사 형변환
    } catch (ClassNotFoundException e) {
        fatalError("클래스를 찾을 수 없습니다.");
    }
    
    // 생성자를 얻는다.
    Constructor<? extends Set<String>> cons = null;
    try {
        cons = cl.getDeclaredConstructor();
    } catch (NoSuchMethodException e) {
        fatalError("매개변수 없는 생성자를 찾을 수 없습니다.");
    }
     
    //집합의 인스턴스를 만든다.
    Set<String> s = null;
    try {
        s = cons.newInstance();
    } catch (IllegalAccessException e) {
        fatalError("생성자에 접근할 수 없습니다.");
    } catch (InstantiationException e) {
        fatalError("클래스를 인스턴스화할 수 없습니다.");
    } catch (InvocationTargetException e) {
        fatalError("생성자가 예외를 던졌습니다: " + e.getCause());
    } catch (ClassCastException e) {
        fatalError("Set을 구현하지 않은 클래스입니다.");
    }
    
    //생성한 집합을 사용한다.
    s.addAll(Arrays.asList(args).subList(1, args.length));
    System.out.println(s);
}

private static void fatalError(String msg) {
    System.err.println(msg);
    System.exit(1);
}
```

위의 예시는 리플렉션의 단점을 보여준다.

* 런타임에 총 6가지의 예외를 던질 수 있다.
* 위에서 발생하는 예외는 모두 컴파일타임에 체크할 수 있는 예외들이다.
* 클래스 이름만으로 인스턴스를 생성해내기 위해 무려 25줄이나 되는 코드를 작성했지만, 그게 아닌 경우에는 생성자 1줄이면 끝난다.
* 리플렉션 예외를 각각 잡는 대신 상위 클래스인 ReflectiveOperationException을 잡으면 코드량을 줄일 수 있다.  
  (ReflectiveOperationException은 Java 7부터 지원한다.)



# 리플렉션은 무조건 쓰지 말아야 한다?

* Spring MVC, Serialize/Deserialize, BeanUtils.copyProperties등 실무에서 사용하는 코드에 리플렉션이 적용된 예는 굉장히 많다.
* 단점이 많다고는 하지만 공통적인 기능을 설계하거나, 재사용 가능한 코드를 설계할 경우에는 오히려 리플렉션이 적합할 수 있다.
* 그렇기 때문에 Java 1.3 이후부터 리플렉션에 대한 성능향상을 발전시켜왔다고 한다.
* 이러한 발전으로 리플렉션은 우려할 만큼 성능이 떨어지지는 않는다고 한다.
* 리플렉션을 남발하는 것이 아닌 필요한 상황에 적시적소에 사용한다면 오히려 서비스 개발을 더 단순화 시킬수 있다.



# 참고

* Effective Java 3rd Edition - Item 65. 리플렉션보다는 인터페이스를 사용하라
* [Java 리플렉션의 오해와 진실](https://kmongcom.wordpress.com/2014/03/15/%EC%9E%90%EB%B0%94-%EB%A6%AC%ED%94%8C%EB%A0%89%EC%85%98%EC%97%90-%EB%8C%80%ED%95%9C-%EC%98%A4%ED%95%B4%EC%99%80-%EC%A7%84%EC%8B%A4/)