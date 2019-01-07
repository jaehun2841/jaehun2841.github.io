---
title: Item 6. 불필요한 객체 생성을 피하라
catalog: true
Categories:
  - Effective-Java
tags:
  - Effective-Java
typora-root-url: effective-java-item6
typora-copy-images-to: effective-java-item6
date: 2019-01-07 21:33:11
subtitle:
header-img:
---

# 서론
똑같은 기능의 객체를 매번 생성하기 보다는 객체 하나는 재사용하는 편이 나을 때가 많다. (아니 거의 무조건 재사용 할 수 있으면 하는 게 좋다.)  
재사용은 빠르고 세련되며, 특히 불변 객체는 언제든지 안전하게 재사용 할 수 있다.

# 아주 안 좋은 객체 생성의 예
```java
String s = new String("Hello");
```
이러한 코드는 매번 새로운 String 객체를 생성하게 된다.

# String Constant pool
위의 코드를 조금 더 보완하면 아래 코드 처럼 사용할 수 있다.
```java
String s = "Hello";
```
Java JVM에는 String Constant pool 이라는 것이 있다.  
(Java 7 버전을 기점으로 Perm영역 -> Heap 영역으로 변경되었다.)  
위 처럼 쓰는 방식을 String 리터럴 방식이라 한다.  

String 리터럴을 사용할 경우 기본적으로 String 내장 메서드인 intern()이라는 메서드를 호출하게 된다. 
```java
String a = "Hello"; // 1
String b = "Hello"; // 2
```
1. 최초로 Hello라는 String 리터럴을 사용하였기 때문에 intern() 메서드가 호출된다.  
--> String Constant pool에서 해당 문자열을 검색하였지만 존재 하지 않기 때문에 String Constant pool에 넣고 새로운 주소값을 반환한다.

2. 두번째로 Hello라는 String 리터럴을 사용하였기 때문에 마찬가지로 intern() 메서드가 호출된다.  
--> String Constant pool에서 해당 문자열을 검색하니 기존에 등록된 주소 값이 반환된다.

실질적으로 a와 b는 같은 주소값을 가지게 된다.

그렇기 때문에 
```java
System.out.println(a == b);      //true
System.out.println(a.equals(b)); //true
```

위의 코드를 실행해 보면 객체의 동등성 비교와 동일성 비교에서 모두 true가 나온다.
* 동등성(equality) : 두 객체의 내용이 같은지 비교
* 동일성(identity) : 두 객체가 같은 객체인지 hashcode를 비교

그렇기 때문에 String을 사용할 경우에는 new를 이용한 객체 생성 방식보다 String 리터럴을 사용하는 방식이 더 좋다. (같은 객체를 재사용 하기 때문)
> 그렇다고 실제 코드에서 String 리터럴을 사용했다고 `==` 을 이용한 동일성 비교는 하지말자.  
상당히 위험한 코드이고, 다른 결과를 초래 할 가능성이 매우 높다.

# Boolean의 예시
Boolean의 경우 new Boolean(true)보다 Boolean.valueOf를 사용하는 것이 더 좋다.

Boolean 클래스를 보면..
```java
public final class Boolean implements java.io.Serializable,
                                      Comparable<Boolean>
{
    /**
     * The {@code Boolean} object corresponding to the primitive
     * value {@code true}.
     */
    public static final Boolean TRUE = new Boolean(true);

    /**
     * The {@code Boolean} object corresponding to the primitive
     * value {@code false}.
     */
    public static final Boolean FALSE = new Boolean(false);

    /**
     * The value of the Boolean.
     *
     * @serial
     */
    private final boolean value;

    /** use serialVersionUID from JDK 1.0.2 for interoperability */
    private static final long serialVersionUID = -3665804199014368530L;

    @Deprecated(since="9")
    public Boolean(boolean value) {
        this.value = value;
    }

    @Deprecated(since="9")
    public Boolean(String s) {
        this(parseBoolean(s));
    }

    public static boolean parseBoolean(String s) {
        return "true".equalsIgnoreCase(s);
    }

    @HotSpotIntrinsicCandidate
    public boolean booleanValue() {
        return value;
    }

    @HotSpotIntrinsicCandidate
    public static Boolean valueOf(boolean b) {
        return (b ? TRUE : FALSE);
    }
}
```

new Boolean의 경우 그때그때 새로운 객체를 생성하게 된다.  
(로컬 컴퓨터에는 OpenJDK 11이 설치되어있는데 Java 9 버전 부터 Boolean 생성자는 Deprecated 처리 되었다.)

하지만, Boolean.valueOf 라는 정적 메서드는 TRUE, FLASE라는 정적 필드에 이미 생성한 인스턴스를 사용하고 있기 때문에  
객체를 추가적으로 생성하지 않아 성능상 이점이 있기 때문이다.

# Auto Boxing을 주의하라!
오토박싱은 Java 5 부터 나온 기능이다.  
primitive 타입과 Class 타입을 자동으로 변환해 주는 기능이다.  
이 기능에 대해 간과하게 되면 쓸 데 없는 객체를 많이 만들어 낼 수 있다.  

책에 소개 된 예제를 잠깐 돌아보면..
```java
private static long sum() {
  Long sum = 0L;
  for (long i = 0; i <= Integer.MAX_VALUE; i++) {
    sum += i; //i에 대해 Auto Boxing이 일어나고 있다.
  }
  return sum;
}
```
i가 더해질 때 마다 AutoBoxing이 발생하게 된다.  
sum 변수를 쓸데 없이 long으로 선언해서 Long객체가 2^32개 만큼 쓸데 없이 생성 되었다. (책에는 231개라고 나와있는데 오타일 거라 생각한다.)   
sum을 long으로만 바꿔줘도 불필요한 객체가 생성되는 일은 없을 것이며, 성능도 더 빨라지게 된다. (책에서는 6.3초 -> 0.59초로 성능 향상을 보았다고 한다.)


# 나만의 객체 Pool을 만들지 말자
객체를 생성하는 비용이 많이 드는 객체라면 미리 pool을 생성하여 사용하면 좋다.  
JDBC에서 사용하는 Connection pool은 생성비용이 높기 때문에 재사용성을 높이기 위해 pool을 사용하는 것이 좋다.  
하지만 일반적으로 개인이 만든 pool은 코드를 헷갈리게 하고 성능을 떨어뜨린다.  
(요즘 GC는 최적화가 잘되서, pool을 만드는 것보다 그냥 객체를 생성하는게 더 빠르다고 한다.)


# 예외는 있다.
방어적 복사본을 만들어야 하는 경우가 있다.  
불변 객체를 유지하기 위해 객체를 수정할 때 마다 새로운 객체를 만들어서 데이터를 수정하는 방식인데, 얼핏 보면 쓸 떼 없는 객체를 생성하는 것 처럼 보인다.  
하지만 객체를 좀 더 만드는 피해보다, 객체가 재사용 되면서 불변성이 깨져 버그가 발생하는 피해가 더 크다는 사실을 명심해야 한다.

# 참고
* Effective Java 3rd Edition - Item 6. 불필요한 객체 생성을 피하라
