---
title: Item 39. 명명 패턴보다 애너테이션을 사용하라
catalog: true
Categories:
  - Effective-Java
tags:
  - Effective-Java
typora-root-url: effective-java-item39
typora-copy-images-to: effective-java-item39
date: 2019-02-03 21:07:10
subtitle:
header-img:
---

# 서론

전통적으로 도구나 프레임워크가 특별히 다뤄야 할 프로그램 요소에는 딱 구분되는 명명 패턴을 적용해 왔다.  
예컨데 테스트 프레임워크인 JUnit3에서는 테스트 메서드 이름을 **test**로 시작하게 지어야 했다.  

단점은 아래와 같다.

1. 오타가 나면 안된다.  
   실수로 이름을 tset~라고 지으면 그 테스트 메서드는 무시하고 지나가기 때문에 테스트 메서드가 제대로 실행됐는지 어쨌는지 모른다.

2. 올바른 프로그램 요소에서만 사용되리라 보증 할 방법이 없다.  
   예컨데 TestSafetyMechanisms으로 JUnit에 던져줬다고 해보자. 개발자는 이 클래스에 정의된 테스트 메서드들을 수행해 주길 기대하겠지만, JUnit은 클래스 이름에는 관심이 없다. 이번에도 경고조차 출력하지 않고 개발자가 의도한 대로 테스트는 진행되지 않는다.

3. 프로그램 요소를 매개변수로 전달할 마땅한 방법이 없다는 것이다.  
   특정 예외를 던져야만 성공하는 테스트가 있을 때, 기대하는 예외의 타입을 매개변수로 전달해야 하는 상황이다.  
   예외의 이름을 테스트 메서드 이름에 덧붙이는 방법도 있지만, 보기에도 나쁘고 깨지기도 쉽다.

이런 문제를 해결해 주는 개념으로 JUnit4 부터는 애너테이션을 도입하였다.  



# 마커 애너테이션 타입선언

```java
/**
* 테스트 메서드임을 선언하는 애너테이션이다.
* 매개변수 없는 정적메서드 전용이다.
*/
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Test{
}
```

```java
public class Sample {
    @Test
    public static void m1() {
        //성공
    }
  
    public static void m2() {
        //실행되지 않는다.
    }
    @Test
    public static void m3() {
        //실패
        throw new RuntimeException("실패");
    }
    public static void m4() {
        //실행되지 않는다.
    }
    @Test
    public void m5() {
        //잘못 사용한 예
        //static method가 아니다.
    }
    public static void m6() {
        //실행되지 않는다.
    }
    @Test
    public static void m7() {
        //실패
        throw new RuntimeException("실패");
    } 
    public static void m8() {
        //실행되지 않는다.
    }
}
```

@Test와 같은 애너테이션을 **아무 매개변수 없이 단순히 대상에 마킹(Marking)한다**는 뜻에서 마커 애너테이션 (Marker Annotation)이라고 한다.  
이 애너테이션을 사용하면 @Test 애너테이션에 오타를 내면 컴파일 오류를 내준다.

아래 프로그램을 실행하면총 8개의 메서드 중 4개의 테스트 메서드가 실행되고

* 성공 1개
* 실패 2개
* 1개는 잘못 사용한 예이다.



애너테이션은 Sample클래스의 의미에 직접적으로 영향을 주지는 않는다.  
그저 애너테이션에 관심있는 프로그램에게 추가 정보를 제공할 뿐이다.  
다시 말하면, 프로그램 코드에의 의미는 그대로 둔 채 애너테이션에 관심있는 도구에서 특별히 처리하도록 하는 것이다.



## 메타 애너테이션 (Meta Annotation)

애너테이션 타입에 다는 애너테이션을 메타 애너테이션 (Meta Annotation)이라 한다.  
메타 애너테이션의 종류로는 

* **@Documented**: 문서에도 애너테이션 정보가 표현되게 함

* **@Inherited**: 자식클래스가 애너테이션을 상속받을 수 있게 함

* **@Repeatable**: 애너테이션을 반복적으로 사용할 수 있게 함

* **@Retention(RetentionPolicy)**: 애너테이션의 범위를 지정 (어느 시점까지 유효한지?)

  * **RetentionPolicy.RUNTIME**: 컴파일 이후에도 JVM에 의해 참조가 가능 - 보통 이거로 설정
  * **RetentionPolicy.CLASS**: 컴파일러가 클래스를 참조할 때 까지 유효
  * **RetentionPolicy.SOURCE**: 애너테이션 정보가 컴파일 이후 사라짐

* **@Target(ElementType[])**: 애너테이션이 적용될 위치를 선언
  * **ElementType.PACKAGE**:  패키지 선언시
  * **ElementType.TYPE**: 타입 선언시
  * **ElementType.CONSTRUCTOR**: 생성자 선언시
  * **ElementType.FIELD**: 멤버 변수 선언시
  * **ElementType.METHOD**:  메소드 선언시
  * **ElementType.ANNOTATION_TYPE**: 어노테이션 타입 선언시
  * **ElementType.LOCAL_VARIABLE**: 지역 변수 선언시
  * **ElementType.PARAMETER**: 매개 변수 선언시
  * **ElementType.TYPE_PARAMETER**: 매개 변수 타입 선언시
  * **ElementType.TYPE_USE**: 타입 사용시
  



## 마커 애너테이션 processor

```java
public class RunTests {
    public static void main(String[] args) {
        int tests = 0;
        int passed = 0;
        Class<?> testClass = Class.forName(args[0]);
        for (Method m : testClass.getDeclaredMethods()) {
            if (m.isAnnotationPresent(Test.class)) {
                test++;
                try {
                    m.invoke(null);
                    passed++;
                } catch (InvocationTargetException wrappedExc) {
                    Throwable exc = wrappedExc.getCause();
                    System.out.println(m + " 실패: " + exc);
                } catch (Exception e) {
                    System.out.println("잘못 사용한 @Test: " + m);
                }
            }
        }
        
        System.out.printf("성공: %d, 실패: %d%n", passed, tests-passed);
    }
}
```

* **m.isAnnotationPresent(Test.class)**: @Test 애너테이션이 적용된 메서드인지 판별
* **m.invoke()**: @Test 메서드 실행
* **InvocationTargetException**: 테스트 메서드가 예외를 던지면 리플렉션 메커니즘이 InvocationTargetException으로 감싸서 다시 던진다.  
  그래서 이 프로그램은 InvocationTargetException에 대해 catch절을 구성해 원래 예외에 담긴 정보를 출력한다.
* **두번째 catch**: 두번째 catch블럭은 잘못 사용해서 발생한 예외를 처리



# 매개변수 하나짜리 애너테이션 타입선언

특정 예외를 던져야만 성공하는 테스트도 있을 것이다.  
특정 예외가 발생했을 때 성공하는 테스트를 지원하도록 해보자.

```java
/**
* 명시한 예외를 던져야만, 성공하는 테스트케이스 애너테이션
*/
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest{
    //한정적 와일드카드를 통해 Throwable을 상속한 모든 타입을 지정
    Class<? extends Throwable> values(); 
}
```



## 예제 프로그램

```java
public class Sample2 {
    @ExceptionTest(ArithmeticException.class)
    public static void m1() {
        int i = 0;
        i = i / i; //divide by zero. ArithmeticException 예외를 발생시킴 -> 성공
    }
    @ExceptionTest(ArithmeticException.class)
    public static void m2() {
        int[] a = new int[0];
        int i = a[1]; //IndexOutOfBoundsException 발생 -> ArithmeticException가 아니므로 실패
    }
    @ExceptionTest(ArithmeticException.class)
    public static void m3() {} // 아무 Exception도 발생하지 않음 -> 실패
}
```



## 매개변수 하나짜리 애너테이션 processor

```java
if(m.isAnnotaionPresent(ExceptionTest.class)) {
    test++;
    try {
        m.invoke(null);
        System.out.printf("테스트 %s 실패: 예외를 던지지 않음%n", m);
    } catch (InvocationTargetException wrappedExc) {
        Throwable exc = wrappedExc.getCause();
        Class<? extends Throwable> excType = m.getAnnotation(ExceptionTest.class).value();
        if(excType.isInstance(exc)) {
            passed++;
        } else {
            System.out.printf("테스트 %s 실패: 기대한 예외 %s, 발생한 예외 %s%n", m, excType.getName(), exc);
        }
    } catch (Exception e) {
        System.out.println("잘못 사용한 @ExceptionTest: " + m);
    }
}
```

위의 @Test와의 차이는 애너테이션의 매개변수를 추출하여 테스트 메서드가 올바른 메서드를 던졌는지 확인하는데 사용한다.  
m.getAnnotation(ExceptionTest.class).value()를 실행할 때 ArithmeticException.class가 리턴된다.



# 배열 매개변수를 받는 애너테이션 타입선언

Exception이 발생하는 종류를 묶어서 처리하고 싶을 때도 있다.  
@ExceptionTest에서 배열형태로 Exception클래스를 받을 수 있도록 배열로 선언하였다.

```java
/**
* 명시한 예외를 던져야만, 성공하는 테스트케이스 애너테이션
*/
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest{
    //한정적 와일드카드를 통해 Throwable을 상속한 모든 타입을 지정
    Class<? extends Throwable>[] values(); 
}
```



## 예제 프로그램

```java
public class Sample3 {
    @ExceptionTest({IndexOutOfBoundsException.class,
                    NullPointerException.class})
    public static void m1() {
        List<String> list = new ArrayList<>();
        //자바 명세에 따르면, 다음 메서드는 IndexOutOfBoundsException이나,
        //NullPointerException을 던질 수 있다.
        //예외 발생 시 성공
        list.addAll(5, null);
    }
}
```



## 배열 매개변수를 받는 애너테이션 processor

```java
if(m.isAnnotaionPresent(ExceptionTest.class)) {
    test++;
    try {
        m.invoke(null);
        System.out.printf("테스트 %s 실패: 예외를 던지지 않음%n", m);
    } catch (InvocationTargetException wrappedExc) {
        Throwable exc = wrappedExc.getCause();
        Class<? extends Throwable>[] excTypes = m.getAnnotation(ExceptionTest.class).value();
        
        int oldPassed = passed;
        for(Class<? extends Throwable> excType : excTypes) {
            if(excType.isInstance(exc)) {
            	passed++;
                break;
            }
        }
        
        if(passed == oldPassed) {
            System.out.println("테스트 %s 실패: %s %n", m, exc);
        }
    } catch (Exception e) {
        System.out.println("잘못 사용한 @ExceptionTest: " + m);
    }
}
```

위의 코드 중 변경되는 부분은  Class<? extends Throwable>[] excTypes을 배열형태로 받아,  
이미 지정한 Exception 클래스 중에 맞는 클래스가 있는지 확인하는 코드가 변경되었다.



# 반복 가능 애너테이션 @Repeatable

자바8에서는 여러개의 값을 받는 애너테이션을 다른 방식으로도 만들 수 있다.  
배열 방식의 매개변수를 사용하는 대신 애너테이션에 @Repeatable 메타애너테이션을 다는 방식이다.   
@Repeatable 애너테이션은 하나의 메서드에 여러개의 애너테이션을 지정할 수 있다.

1. @Repeatable을 단 애너테이션을 반환하는 **컨테이너 애너테이션**을 하나 더 정의한다.
2. @Repeatable에 이 컨터이너 애너테이션의 class 객체를 매개변수로 전달해야 한다.
3. 컨테이너 애너테이션은 내부 애너테이션 타입의 배열을 반환하는 value 메서드를 정의한다.
4. 컨터이너 애너테이션에는 @Retention과 @Target을 적절히 명시한다.  
   (그렇지 않으면 컴파일되지 않는다.)



```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@Repeatable(ExceptionTestContainer.class)
public @interface ExceptionTest {
    Class<? extends Throwable> value();
}

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTestContainer {
    ExceptionTest[] value();
}
```



## 예제 프로그램 - 반복 가능 애너테이션을 적용한 예

```java
@ExceptionTest(IndexOutOfBoundsException.class)
@ExceptionTest(NullPointerException.class)
public static void m1() {...}
```

반복 가능 애너테이션은 처리할 때 주의를 요한다.
반복 가능 애너테이션을 여러개 달면, 하나만 달았을 때와 구분하기 위해 해당 **컨테이너** 애너테이션 타입이 적용 된다.

* getAnnotationByType 메서드는 이 둘을 구분하지 않아 @ExceptionTest와 @ExceptionTestContainer를 모두 가져온다.
* isAnnotationPresent는 둘을 구분한다.
  * 만약 @ExceptionTest를 여러번 단 다음, isAnnotationPresent로 ExceptionTest를 검사하면 false가 나온다.  
    (@ExceptionTestContainer로 인식하기 때문)
  * 반대로 @ExceptionTest를 한번 만 단 다음, isAnnotationPresent로 ExceptionTestContainer를 검사하면 false가 나온다.
    (@ExceptionTest가 적용되었기 때문)



## 반복 가능 애너테이션 processor

```java
try {
        m.invoke(null);
        System.out.printf("테스트 %s 실패: 예외를 던지지 않음%n", m);
    } catch (InvocationTargetException wrappedExc) {
        Throwable exc = wrappedExc.getCause();
        int oldPassed = passed;
    
        ExceptionTest[] excTests = m.getAnnotationByType(ExceptionTest.class);
        for(ExceptionTest excType : excTypes) {
            if(excType.isInstance(exc)) {
            	passed++;
                break;
            }
        }
        
        if(passed == oldPassed) {
            System.out.println("테스트 %s 실패: %s %n", m, exc);
        }
}
```

반복 가능 애너테이션을 사용한 경우 getAnnotationByType를 사용해 애너테이션 정보를 가져오는 것이 좋다.



# 정리

* 애너테이션이 명명패턴을 이용할 때 보다 확실히 낫다.

* 애너테이션으로 할 수 있는 일을 명명 패턴으로 처리할 이유는 없다.

  

# 참고

* Effective Java 3rd Edition - Item 39. 명명 패턴보다 애너테이션을 사용하라