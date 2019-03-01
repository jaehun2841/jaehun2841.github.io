---
title: Item 63. 문자열 연결은 느리니 주의하라
catalog: true
Categories:
  - Effective-Java
tags:
  - Effective-Java
typora-root-url: effective-java-item63
typora-copy-images-to: effective-java-item63
date: 2019-03-01 20:45:07
subtitle:
header-img:
---

# 서론

문자열 연결 연산자(+)는 여러 문자열을 하나로 합쳐주는 편리한 수단이다.  
그런데 한 줄짜리 출력값(`return prefix + str + suffix;` 정도?)  혹은 작고 크기가 고정된 객체의 문자열 표현을 만들 때라면 괜찮다.  
하지만 문자열 여러개를 사용하기 시작하면 성능 저하를 피할 수 없다.  

**문자열 연결 연산자로 문자열 n개를 연결하는 시간은 n^2에 비례한다.**  
문자열은 불변이기 때문에 두 문자열을 연결하는 경우에는 양쪽의 내용을 복사하여 연결한 다음 새로운 String 객체를 만들어야 하기 때문이다.



# 문자열 연결을 잘못 사용한 예 - 느리다!

```java
public String statement() {
    String result = "";
    for (int i = 0; i < numItems(); i++) {
        result += lineForItem(i); //문자열 연결
    }
    return result;
}
```



# StringBuilder를 사용하여 문자열을 연결한 예

```java
public String statement2() {
    StringBuilder sb = new StringBuilder(numItems() * LINE_WIDTH);
    for (int i = 0; i < numItems(); i++) {
       sb.append(lineForItem(i));
    }
    return sb.toString();
}
```

자바 6이후 문자열 연결 성능을 다방면으로 개선 했지만, 두 메서드의 성능 차이는 여전하다.



# String, StringBuffer, StringBuilder의 비교

* String과 StringBuffer는 Java 1.0의 등장과 함께 같이 등장하였다.
* StringBuilder는 조금 뒤인 Java 1.5부터 등장하였다.
* String의 concat연산은 + 기호를 사용하여 concatination을 수행한다.
* StringBuffer와 StringBuilder는 append 메서드를 통해 concatination을 수행한다.
* 정확히 말하면 StringBuffer와 StringBuilder는 AbstractStringBuilder를 상속하고 있으며,  
  결국은 같은 append 메서드를 사용한다.
* StringBuffer와 StringBuilder 차이점은 thread-safe에 있다.
  * StringBuffer의 append 메서드에는 `syncronized` 예약어가 붙어있어 thread-safe하다
  * StringBuilder의 append 메서드는 thread-safe 하지 않다.
  * 따라서 multi-thread 환경에서 문자열 결합을 할 때는 StringBuffer를 사용하는 것이 안전하다.
  * 단일 thread라면 StringBuilder를 사용하는 것이 StringBuffer보다 성능이 더 좋다.  
    (아무래도 동기화 체크를 안해도 되니 말이다.)



## String + String 연산이 느린 이유

* String은 불변 클래스이기 때문에 String + String을 하기 위해서는 
* String내의 char[] 혹은 byte[]를 copy한다.
* 2개의 array의 length를 더한 값으로 새로운 array를 생성한다.
* array에 기존의 값을 채워넣는다.
* new String(byte[]) 생성자를 통해 새로운 String 객체를 생성한다.
* 이런식으로 하면 String + String 연산이 일어날 때마다 String 객체가 생성된다.
* Heap Memory에 String 객체가 많아지면 GC가 돌면서 String 객체를 제거한다.
* GC는 동작 시 stop the world라는 행위를 한다. **(JVM의 작동이 일시적으로 멈춘다.**)
* 위와 같은 행위가 계속되면 당연히 느려 질 수 밖에 없다.



## StringBuilder.append() 메서드 파헤치기

```java
/**
     * Appends the specified string to this character sequence.
     * <p>
     * The characters of the {@code String} argument are appended, in
     * order, increasing the length of this sequence by the length of the
     * argument. If {@code str} is {@code null}, then the four
     * characters {@code "null"} are appended.
     * <p>
     * Let <i>n</i> be the length of this character sequence just prior to
     * execution of the {@code append} method. Then the character at
     * index <i>k</i> in the new character sequence is equal to the character
     * at index <i>k</i> in the old character sequence, if <i>k</i> is less
     * than <i>n</i>; otherwise, it is equal to the character at index
     * <i>k-n</i> in the argument {@code str}.
     *
     * @param   str   a string.
     * @return  a reference to this object.
     */
public AbstractStringBuilder append(String str) {
    if (str == null) {
        return appendNull();
    }
    int len = str.length();
    ensureCapacityInternal(count + len);
    putStringAt(count, str);
    count += len;
    return this;
}
```

* 위의 코드는 AbstractStringBuilder의 append 메서드이다.  
* 새로운 String의 길이 만큼 AbstractStringBuilder의 내의 byte[]의 사이즈를 늘리고 복사한다.
* 그 다음 String에 대한 byte[]를 AbstractStringBuilder의 내의 byte[]에 추가한다.
* String + String 연산과의 차이점은 불필요한 String 객체가 발생하지 않는다는 점이다



# String Concatination의 발전

* Java String 연산에 대한 성능최적화를 다방면으로 생각하고 있고, Java 9 부터 String의 내부 배열을   
  **char[] -> byte[]로 변경**하여 성능을 더 향상 시켰다.
* Java 1.5 버전부터 String + String연산에 대해 **Compile Time에 StringBuilder를 사용하도록 코드를 변경한다.**  
  하지만 JDK가 항상 자동으로 바꿔준다는 보장이 없으니 String + String 보다는 StringBuilder를 사용해야 한다.



# 참고

* Effective Java 3rd Edition - Item 63. 문자열 연결은 느리니 주의하라