---
title: Item 13. clone 재정의는 주의해서 진행하라
catalog: true
Categories:
  - Effective-Java
tags:
  - Effective-Java
typora-root-url: effective-java-item13
typora-copy-images-to: effective-java-item13
date: 2019-01-13 16:31:38
subtitle:
header-img:
---
# 서론
Cloneable은 복제해도 되는 인터페이스 임을 명시하는 용도의 믹스인 인터페이스이다.  
하지만 아쉽게도 의도한 목적을 제대로 이루지 못했다. 가장 큰 문제는 clone메서드가 선언된 곳이 Cloneable이 아닌 Object이고, 그마저도 protected라는데 있다. 그래서 Cloneable을 구현하는 것만으로는 외부에서 clone 메서드를 호출 할 수 없다.

# Cloneable 인터페이스는 무슨일을 할까?
```java
/**
 * A class implements the <code>Cloneable</code> interface to
 * indicate to the {@link java.lang.Object#clone()} method that it
 * is legal for that method to make a
 * field-for-field copy of instances of that class.
 * <p>
 * Invoking Object's clone method on an instance that does not implement the
 * <code>Cloneable</code> interface results in the exception
 * <code>CloneNotSupportedException</code> being thrown.
 * <p>
 * By convention, classes that implement this interface should override
 * {@code Object.clone} (which is protected) with a public method.
 * See {@link java.lang.Object#clone()} for details on overriding this
 * method.
 * <p>
 * Note that this interface does <i>not</i> contain the {@code clone} method.
 * Therefore, it is not possible to clone an object merely by virtue of the
 * fact that it implements this interface.  Even if the clone method is invoked
 * reflectively, there is no guarantee that it will succeed.
 *
 * @author  unascribed
 * @see     java.lang.CloneNotSupportedException
 * @see     java.lang.Object#clone()
 * @since   1.0
 */
public interface Cloneable {
}
```
자바의 Cloneable 인터페이스를 보면 아무런 메서드도 없다.  
아무것도 없지만, 사실은 Object의 clone메서드의 동작방식을 결정한다.  
Cloneable을 구현한 클래스의 인스턴스에서 clone을 호출하면, 그 객체의 필드들을 하나하나 복사한 객체를 반환하며, 그렇지 않은 클래스의 인스턴스에서 호출하면, `ClassNotSupportedException`을 던진다.

# Object 클래스의 clone 규약
```java
    /**
     * Creates and returns a copy of this object.  The precise meaning
     * of "copy" may depend on the class of the object. The general
     * intent is that, for any object {@code x}, the expression:
     * <blockquote>
     * <pre>
     * x.clone() != x</pre></blockquote>
     * will be true, and that the expression:
     * <blockquote>
     * <pre>
     * x.clone().getClass() == x.getClass()</pre></blockquote>
     * will be {@code true}, but these are not absolute requirements.
     * While it is typically the case that:
     * <blockquote>
     * <pre>
     * x.clone().equals(x)</pre></blockquote>
     * will be {@code true}, this is not an absolute requirement.
     * <p>
     * By convention, the returned object should be obtained by calling
     * {@code super.clone}.  If a class and all of its superclasses (except
     * {@code Object}) obey this convention, it will be the case that
     * {@code x.clone().getClass() == x.getClass()}.
     * <p>
     * By convention, the object returned by this method should be independent
     * of this object (which is being cloned).  To achieve this independence,
     * it may be necessary to modify one or more fields of the object returned
     * by {@code super.clone} before returning it.  Typically, this means
     * copying any mutable objects that comprise the internal "deep structure"
     * of the object being cloned and replacing the references to these
     * objects with references to the copies.  If a class contains only
     * primitive fields or references to immutable objects, then it is usually
     * the case that no fields in the object returned by {@code super.clone}
     * need to be modified.
     * <p>
     * The method {@code clone} for class {@code Object} performs a
     * specific cloning operation. First, if the class of this object does
     * not implement the interface {@code Cloneable}, then a
     * {@code CloneNotSupportedException} is thrown. Note that all arrays
     * are considered to implement the interface {@code Cloneable} and that
     * the return type of the {@code clone} method of an array type {@code T[]}
     * is {@code T[]} where T is any reference or primitive type.
     * Otherwise, this method creates a new instance of the class of this
     * object and initializes all its fields with exactly the contents of
     * the corresponding fields of this object, as if by assignment; the
     * contents of the fields are not themselves cloned. Thus, this method
     * performs a "shallow copy" of this object, not a "deep copy" operation.
     * <p>
     * The class {@code Object} does not itself implement the interface
     * {@code Cloneable}, so calling the {@code clone} method on an object
     * whose class is {@code Object} will result in throwing an
     * exception at run time.
     *
     * @return     a clone of this instance.
     * @throws  CloneNotSupportedException  if the object's class does not
     *               support the {@code Cloneable} interface. Subclasses
     *               that override the {@code clone} method can also
     *               throw this exception to indicate that an instance cannot
     *               be cloned.
     * @see java.lang.Cloneable
     */
    @HotSpotIntrinsicCandidate
    protected native Object clone() throws CloneNotSupportedException;
```

Object에 명시된 clone 규약이 주석으로 쓰여져 있다.
* x.clone() != x은 참이다.  
복사한 객체와 원본 객체는 서로 다른 객체이다.
* x.clone() .getClass() == x.getClass()은 일반적으로 참이다.
하지만 반드시 만족해야 하는 것은 아니다.
* x.clone.equals(x) 은 참이다.  
복사한 객체와 원본객체는 논리적 동치성이 같다.
* x.clone().getClass() == x.getClass()은 참이다.  
관례상, 이 방법으로 반환된 객체는 독립성이 있어야 한다.  
이를 만족하려면 super.clone으로 얻은 객체의 필드 중 하나 이상을 반환 전에 수정해야 할 수도 있다.
* Cloneable을 구현하지 않은 클래스의 경우, CloneNotSupportedException이 throw된다.
* 모든 Array는 Cloneable을 구현하도록 고려되었다. clone 메서드는 T[]를 리턴하도록 설계
* T는 기본타입 또는 참조타입으로 설계
* 기본적으로 Object.clone은 clone대상 클래스에 대해 새로운 객체를 new로 생성
* 모든 필드들에 대해 초기화를 진행
* 하지만 필드에 대한 객체를 다시 생성하지 않는 Shallow copy 방식으로 수행한다 (deep copy아님)
* 클래스에 대한 복제본을 원하지 않는 경우 clone메서드를 재정의하여 CloneNotSupportedException을 throw하도록 함

# clone 메서드 재정의 시, 주의할 점
## 기본적인 clone메서드 재정의
```java
class PhoneNumber implements Cloneable {
  @Override
  public PhoneNumber clone() {
    try {
      return (PhoneNumber) super.clone();
    } catch(ClassNotSupportedException e) {
      //아무처리를 하지 않거나, RuntimeException으로 감싸는 것이 사용하기 편하다.
    }
  }
}
```
* super.clone()을 실행하면 PhoneNumber에 대한 완벽한 복제가 이루어진다.
* super.clone()의 리턴 타입은 Object이지만, 자바의 공변 반환타이핑 기능을 통해 PhoneNumber 타입으로 캐스팅하여 리턴하는 것이 가능하다.  
(추천하는 기능)
* try-catch 부분으로 감싼것은 super.clone() 메서드에서 ClassNotSupportedException이라는 checked exception을 리턴하기 떄문에 처리 해주었다. 
  * 하지만 PhoneNumber가 Cloneable을 구현하기 떄문에 절대 실패하지 않는다.  
  따라서 이부분은 RuntimeException으로 처리하거나, 아무것도 설정하지 않아야 한다.

## 가변 상태를 갖는 필드에 대한 복제
```java
public class Stack implements Cloneable{
  private Object[] elements;
  private int size = 0;
  private static final int DEFAULT_INITIAL_CAPACITY = 16;

  public Stack() {
    this.elements = new Object[DEFAULT_INITIAL_CAPACITY];
  }

  public void push(Object o) {}
  }
  ...

  @Override
  public Stack clone() {
    try {
      Stack result = (Stack) super.clone();
      result.elements = 
    } catch(CloneNotSupportedException e) {
    }
  }
}
```
이 클래스가 단순히 clone메서드를 이용해 super.clone()만 실행하게 된다면,  
new Stack()을 통해 새로운 객체가 생성되고 필드모두 원본 객체와 동일하게 초기화가 될 것이다.  
하지만, 위에서 얘기한 Object의 clone 기본규약에는 Deep copy가 아닌 Shallow Copy를 이용해 초기화를 진행한다고 적혀있다.  
따라서 배열과 같은 가변필드는 원본 필드와 객체를 공유하게 된다. 

Clone메서드는 사실상 생성자와 같은 효과를 낸다.  
즉, clone은 원본 객체에 아무런 해를 끼치지 않는 동시에 복제된 객체의 불변식을 보장해야 한다.  
그렇기 때문에 제대로 복제하기 위해서는, elements라는 배열을 똑같이 복사해서 만들어줘야 한다.

## 배열 복사 
배열을 복제하는 방법 중 가장 권장하는 방법은 
array.clone()을 이용해 복사하는 방법이다.  
사실, 배열은 clone기능을 가장 제대로 사용하는 유일한 예이다.

하지만, array 필드가 final이 적용되어 있다면 array.clone()을 통한 초기화는 할 수 없다. (final이기 떄문에 객체 생성 이후 초기화 불가)  
**Cloneable 아키텍처는 가변객체를 참조하는 필드는 final로 선언하라 라는 일반 용법과 충돌한다.**  
그래서 복제 할 수 있는 클래스를 만들기 위해 일부 필드에서 final 한정자를 제거해야 할 수도 있다.

## Stack overflow 문제
```java
public class HashTable implements Cloneable  {
  private Entry[] buckets = ...;
  private static class Entry {
    final Object key;
    Object value;
    Entry next;

    Entry(Object key, Object value, Entry next) {
      this.key = key;
      this.value = value;
      this.next = next;
    }
  }

  @Override
  public HashTable clone() {
    try {
      HashTable result = (HashTable) super.clone();
      result.buckets = buckets.clone();
      return result;
    } catch(CloneNotSupportedException e) {
      throw new Assertion();
    }
  }
}
```
복제본은 자신만의 버킷 배열은 갖지만, 배열내의 Entry는 원본과 같은 연결리스트를 참조하여, 불변성이 꺠지게 된다.

그래서 HashTable.Entry 클래스는 내부적으로 deep copy를 지원하도록 보강되었다. 연결리스트에 대한 next를 복제할 때 재귀적으로 clone을 호출하도록 되어있는데, 재귀 호출 떄문에 연결리스트의 size만큼 스택프레임을 소비하여, 리스트가 길면 stackoverflow에러를 발생시킬 위험이 있다.  
이 문제를 해결하기 위해서는 재귀 호출을 통한 deepcopy대신 반복자를 써서 순회하는 방법으로 수정해야 한다.

## 안전하지만 느린 copy
HashTable을 예로 들면, buckets필드를 새로운 배열로 초기화 한다음 원본테이블의 있는 버킷의 내용을 put(key, value) 메서드를 호출해  
새로 버킷을 만드는 방법이 있다.  
이 방법은 안전하긴 하지만, low level에서 작동하는 copy보다는 느리다.

## 생성자 내에서는 재정의 될 수 있는 메서드를 호출 하지 말자
만약 clone이 하위 클래스에서 재정의한 메서드를 호출하면, 하위 클래스는 복사과정에서 자신의 상태를 교정할 기회를 잃게되어  
원본과 복제본의 상태가 달라질 수 있다.

## ClassNotSupportedException
Object의 clone메서드는 CloneNotSupportedException을 던진다고 선언했지만 재정의한 메서드는 그렇지 않다.  
public인 clone 메서드에서는 throws절을 없애던가, RuntimeException으로 throws 해야한다.  
그렇지 않으면 clone 메서드를 호출 할 떄마다, 예외처리를 해야 해서 사용하기가 불편하다.

하위 클래스에서 Clone을 방지하기 위해 동작하지 않게 만들수도 있다.
```java
@Override
protected final Object clone() throws CloneNotSupportedException {
  throw new CloneNotSupportedException();
}
```

## 스레드 안전성을 고려한다면 적절히 동기화해야 한다.
스레드 안정성을 고려한다면 clone 메서드에 대해 적절히 동기화 처리 해야 한다.

```java
@Override
public synchronized Object clone() {
  try {
    Object result = super.clone();
  } catch(CloneNotSupportedException e) {
  }
}
```

## 복사 생성자와 복사 팩터리 메서드
```java
public Yum(Yum yum) {}
```
```java
public static Yum newInstance(Yum yum) {}
```

복사 생성자와 복사 팩터리 메서드는 Cloneable/clone 방식보다 나은 면이 많다. 
* 언어 모순적이고 위험한 객체 생성 메커니즘을 사용하지 않는다. (super.clone())
* clone 규약에 기대지 않는다.
* 정상적인 final필드 용법과도 충돌하지 않는다.
* 불필요한 check exception 처리가 필요없다.
* 형변환도 필요없다.
* 복사 생성자와 복사 팩터리는 인터페이스 타입의 인스턴스를 인수로 받을 수 있다. 

# Deep copy vs Shallow copy
[Shallow Copy와 Deep Copy](http://localhost:4000/2019/01/13/java-object-copy) - 이 포스팅을 참고하자


# 참고
* Effective Java 3rd Edition - Item 13. clone 재정의는 주의해서 진행하라