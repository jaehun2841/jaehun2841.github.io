---
title: Item 56. 공개된 API 요소에는 항상 문서화 주석을 작성하라
catalog: true
Categories:
  - Effective-Java
tags:
  - Effective-Java
typora-root-url: effective-java-item56
typora-copy-images-to: effective-java-item56
date: 2019-02-24 16:55:23
subtitle:
header-img:
---

# 서론

API를 쓸모 있게 하려면 잘 작성된 문서도 곁들여야 한다.  
전통적으로 API는 사람이 직접 작성하므로 코드가 변경되면 매번 함께 수정해야 하는데,  
자바에서는 자바독(JavaDoc)이라는 유틸리티가 이 귀찮은 작업을 도와준다.

문서화 주석을 작성하는 규칙은 공식 언어 명세에 속하진 않지만 자바 프로그래머라면 응당 알아야 하는 업계 표준 API라 할 수 있다.



# 모든 공개된 리소스에 주석을 달아야 한다.

* API를 올바로 문서화 하려면 공개된 클래스, 인터페이스, 메서드, 필드 선언에 문서화 주석을 달아야 한다.  
* 직렬화 할 수 있는 클래스라면 직렬화 형태에 대해서도 적어야 한다.  
* 문서화 주석이 없다면 JavaDoc도 그저 공개 API 요소들의 선언만 나열해 주는게 전부다.  
* 문서가 잘 갖춰지지 않은 API는 쓰기 헷갈려서 오류의 원인이 되기 쉽다.  
* 기본 생성자에는 문서화 주석을 달 방법이 없으니 공개 클래스는 절대 기본 생성자를 사용하면 안된다.  
* 유지 보수까지 고려한다면 대다수의 공개되지 않은 클래스, 인터페이스, 생성자, 메서드, 필드에도 문서화 주석을 달아야 한다.



# 메서드용 문서화 주석에는 규약을 명료하게 기술해야 한다.

* 메서드용 문서화 주석에는 해당 메서드와 클라이언트 사이의 규약을 명료하게 기술해야 한다.
* 메서드가 어떻게 동작하는지를 적는게 아니라 무엇을 하는지 기술해야 한다.  
  (how가 아닌 what을 기술해야한다.)
* 클라이언트가 해당 메서드를 호출하기 위한 전제조건(precondition)을 모두 나열해야 한다.
* 메서드가 성공적으로 수행된 후에 만족해야 하는 사후조건(postcondition)도 모두 나열해야 한다.
* 일반적으로 전제조건은 @throw 태그로 비검사 예외를 선언하여 암시적으로 기술한다.  
  비검사 예외 하나가 전제조건 하나와 연결되는 것이다.
* @param 태그를 이용해 그 조건에 영향받는 매개변수에 기술 할 수도 있다.



# 전제조건과 사후조건 뿐만 아니라 부작용도 문서화 하라

* **부작용**이란 **사후조건으로 명확히 나타나지는 않지만 시스템의 상태에 따라 어떠한 변화**를 가져오는 것을 의미
* 예를들어 메서드에서 백그라운드 스레드를 실행시키는 메서드라면 그 사실을 문서에 밝혀야 한다.



# 문서화 태그

* @param 
  * 메서드의 파라미터에 대한 정보
  * 매개변수가 뜻하는 값을 명사구로 쓴다.
* @return 
  * 메서드의 반환타입이 void가 아니라면 반환 타입을 명시
  * 반환값이 뜻하는 값을 명사구로 작성
  * 드물게는 산술표현식으로도 작성하기도 한다.
* @throws 
  * 발생가능성이 있는 모든 예외에 대해 명시
  * if로 시작해 해당 예외를 던지를 조건을 설명하는 절이 뒤따른다.
* @code
  * 태그로 감싼 내용을 코드용 폰트로 렌더링한다.
  * 태그로 감싼 내용에 포함된 HTML 요소나 다른 JavaDoc 태그를 무시한다.
  * @기호에는 무조건 탈출문자를 붙여야 하니 문서화 주석안의 코드에서 annotation을 사용한다면 주의해야한다.
* @implSpec
  * 자기사용 패턴(self-use pattern)에 대해서도 문서에 남겨 다른 프로그래머에게 그 메서드를 올바르게 재정의 하는 방법을 알려야 한다.
  * 일반적인 문서화 주석은 해당 메서드와 클라이언트 사이의 관계를 설명
  * @implSpec 주석은 해당 메서드와 하위 클래스 사이의 관계를 설명하여, 하위 클래스들이 그 메서드를 상속하거나 super 키워드를 이용해 호출할 때 그 메서드가 어떻게 동작하는지를 명확히 인지하고 사용하게 해야 한다.
  * -tag "implSpec:a:Implementation Requirement" 스위치를 키지 않으면 @implSpec 태그를 무시
* @literal
  * 이 태그는 <, >와 같은 HTML 태그를 무시하게 해준다.
  * @code와 비슷하지만, 코드 폰트로 렌더링 하지는 않는다.
  * A geometric series converges if {@literal |r| < 1}. 처럼 사용 할 수 있다.
  * API 문서에서 가독성을 높이기 위해 사용한다.



관례상 @param, @return, @throws 태그의 설명에는 마침표를 붙이지 않는다.  
(한글로 작성하는 경우에는 온전한 종결어미로 끝나면 마침표를 붙여주는게 일관돼 보인다.)

```java
/**
* Returns the element at the specified position in this list.
*
* <p>This method is <i>not</i> guaranteed to run in constant time.
* In some implementations it may run in time proportional to the element position.
* @param index index of the element to return
* @return the element at the specified position in this list
* @throws IndexOutOfBoundsException if the index is out of range
*         ({@code index < 0 || index >= size()})
*/
E get(int index);
```

```java
/**
* 이 리스트에서 지정한 위치의 원소를 반환한다.
*
* <p>이 메서드는 상수시간에 수행됨을 보장하지 <i>않는다.</i> 
* 구현에 따라 원소의 위치에 비례해 시간이 걸릴 수도 있다.
*
* @param index 반환할 원소의 인덱스
* @return 이 리스트에서 지정한 위치의 원소
* @throws IndexOutOfBoundsException index가 범위를 벗어나면,
*         즉, ({@code index < 0 || index >= size()}) 이면 발생한다.
*/
E get(int index);
```



# html 태그의 사용

```java
/**
* Returns the element at the specified position in this list.
*
* <p>This method is <i>not</i> guaranteed to run in constant time.
* In some implementations it may run in time proportional to the element position.
* @param index index of the element to return
* @return the element at the specified position in this list
* @throws IndexOutOfBoundsException if the index is out of range
*         ({@code index < 0 || index >= size()})
*/
E get(int index);
```

* 위에서 보면 <p>나 <i>와 같은 html 태그가 사용되었다.
* JavaDoc 유틸리티는 문서화 주석을 HTML로 변환하므로 문서화 주석 안의 HTML 요소들이 최종 HTML문서에 반영된다.



# 요약 설명

* 각 문서화 주석의 첫 번째 문장은 해당 요소의 요약 설명(Summary description)으로 간주된다.

* `Returns the element at the specified position in this list.` 와 같은 문장이 이에 속한다.

* 첫 번째 `.` 기호가 나타나기 전까지의 문장을 첫 번째 문장으로 간주한다.

* 예를들어 "머스터드 대령이나 Mrs. 피콕 같은 용의자."라고 하면 첫 번째 마침표가 나오는  
  "머스터드 대령이나 Mrs." 까지만 요약 설명이 된다.

* 위와 같은 예제를 해결하기 위해선 @literal을 사용한다.  
  ex)  머스터드 대령이나 {@literal Mrs.} 피콕 같은 용의자.

* Java 10부터는 {@summary}라는 요약 설명 전용 태그가 추가되어 한번에 사용 가능하다.  
  {@summary 머스터드 대령이나 Mrs. 피콕 같은 용의자.}

* **요약 설명이란, 문서화 주석의 첫 문장이다.** 라고 말하면 살짝 오해의 소지가 있다.  
  주석 작성 규약에 따르면 요약 설명은 완전한 문장이 되는 경우가 드물기 때문이다.  
  메서드와 생성자의 요약 설명은 해당 메서드와 생성자의 동작을 설명하는 (주어가 없는) 동사구여야 한다.

  ```
  ArrayList(int initialCapacity): Constructs an empty list with the specified initail capacity
  Collection.size(): Returns the number of elements in this collection.
  ```

* 클래스나 인터페이스, 필드의 요약설명은 대상을 설명하는 명사절이어야 한다.  
  클래스와 인터페이스의 대상은 그 인스턴스이고, 필드의 대상은 필드 자신이다.  

  ```
  Instant: An instantaneous point on the time-line (타임라인상의 특정순간(지점))
  Math.PI: The double value that is closer than any other to pi, the ratio of the circumference of a circle to its diamter (원주율(pi)에 가장 가까운 double 값)
  ```

  

# 색인(Index) 기능

* 자바 9부터는 JavaDoc이 생성한 HTML 문서에 대해 검색(색인) 기능이 추가되어 광대한 API 문서를 누비는 일이 수월해짐

* @index 태그를 사용해 API에서 중요한 용어를 추가로 색인화 할 수 있다.

  ```
  This method compiles with the {@index IEEE 754} standard.
  ```

  

# 제네릭 타입이나 제네릭 메서드의 주석

* 제네릭 타입이나 제네릭 메서드를 문서화 할 때는 모든 타입 매개변수에 주석을 달아야 한다.

```java
 /* 
 * An object that maps keys to values.  A map cannot contain duplicate keys;
 * each key can map to at most one value.
 *
 * @param <K> the type of keys maintained by this map
 * @param <V> the type of mapped values
 */
public interface Map<K, V> {
```

```java
 /* 
 * 키와 값을 매핑하는 객체, 맵은 키를 중복해서 가질 수 없다.
 * 즉, 키 하나가 가리킬 수 있는 값은 최대 1개다.
 *
 * @param <K> 이 맵이 관리하는 키의 타입
 * @param <V> 매핑된 값의 타입
 */
public interface Map<K, V> {
```



# 열거타입에는 상수별로 주석을 달아라

* 열거 타입을 문서화할 때는 상수들에도 주석을 달아야 한다.
* 열거 타입 자체와 열거 타입의 public 메서드도 물론이다.

```java
/**
* An instrument section of a symphony orchestra
*/
public enum OrchestraSection {
    /** WoodWinds, such as flute, clarinet and oboe */
    WOODWIND,
    /** Brass instruments, such as french horn and trumper */
    BRASS,
    /** Percussion instruments, such as timpani, cymbals */
    PERCUSSION,
    /** Stringed instruments, such as violin and cello */
    STRING
}
```



# 애너테이션 타입을 문서화 할 때는 멤버에도 주석을 달아라

* 애너테이션 타입을 문서화할 때는 멤버들에도 모두 주석을 달아야 한다.
* 애너테이션 타입 자체도 물론이다.
* 필드 설명은 명사구로 한다.
* 애너테이션 타입의 요약 설명은 프로그램 요소에 이 애너테이션을 단다는 것이 어떤 의미인지를 설명하는 동사구로 한다.

```java

/**
 * Indicates that the annotated method is a test method that 
 * must throw the designated exception to pass
 */
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
    /**
     *  The exception that the annotated test method must throw
     *  in order to pass. (The test is permitted to throw any subtype
     *  of the type described by this class object.)
     */
    Class<? extends Throwable> value();
}
```



# 패키지를 설명하는 문서화 주석은 package-info.java에 작성한다.

* 패키지를 설명하는 문서화 주석은 package-info.java에 명시한다.
* 패키지 선언을 반드시 포함해야 하며 패키지 선언관련 애너테이션을 추가로 포함할 수도 있다.
* 모듈 시스템을 사용한다면, module-info.java 파일에 작성하면 된다.



# 스레드 안전 수준을 반드시 API 설명에 포함해야 한다.

* 클래스 혹은 정적 메서드가 스레드 안전하든 그렇지 않든, 스레드 안전 수준을 반드시 API 설명에 포함해야 한다.
* 직렬화할 수 있는 클래스라면 직렬화 형태도 API 설명에 기술해야 한다.



# JavaDoc은 메서드 주석을 상속 시킬 수 있다.

* 문서화 주석이 없는 API 요소를 발견하면 JavaDoc이 가장 가까운 문서화 주석을 찾아준다.
* 상위 클래스보다 구현한 인터페이스 주석을 더 먼저 찾는다.
* @inheritedDoc 태그를 사용해 상위 타입의 문서화 주석 일부를 상속할 수 있다.
* 클래스는 자신이 구현한 인터페이스의 문서화 주석을 재사용할 수 있다.



# 정리

* 문서화 주석은 API를 문서화 하는 가장 훌륭하고 효과적인 방법이다.

* 공개 API라면 빠짐없이 설명을 달아야 한다.

* 표준 규약을 일관되게 지키자.

* 문서화 주석이외에 HTML 태그를 사용할 수 있다.

  


# 참고

* Effective Java 3rd Edition - Item 56. 공개된 API 요소에는 항상 문서화 주석을 작성하라