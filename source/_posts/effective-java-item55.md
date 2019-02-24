---
title: Item 55. 옵셔널 반환은 신중히 하라
catalog: true
Categories:
  - Effective-Java
tags:
  - Effective-Java
typora-root-url: effective-java-55
typora-copy-images-to: effective-java-55
date: 2019-02-24 14:04:21
subtitle:
header-img:
---



# 서론

자바 8 전에는  메서드가 특정 조건에서 값을 반환할 수 없을 때 취할 수 있는 선택지가 두 가지 있었다.

* Exception Throw
  * 예외는 반드시 예외적인 상황에서만 사용해야 한다.
  * 예외는 실행 스택을 추적(StackTrace)를 캡처하기 때문에 비용이 비싸다.
* Null Return
  * null을 리턴하는 경우에는 NPE(Null Pointer Exception)을 항상 조심해야한다.

자바 8이 등장하면서 Optional이라는 또 하나의 선택지가 추가되었다.



# Optional 이란?

* Optional이란, 값이 있을 수도 있고 없을 수도 있는 객체이다.  참조 타입의 객체를 한번 감싼 일종의 래퍼 클래스 이다.  
* Optional은 원소를 최대 1개 가질 수 있는 불변 Collection이다.
* 자바 8 이전의 코드보다 null-safe한 로직을 처리 할 수 있게 해준다.
* Optional을 반환하여 좀 더 로직을 유연하게 작성할 수 있게 해준다.



# Optional 메서드

* **Optional.empty()**
  * 내부 값이 비어있는 Optional 객체 반환
* **Optional.of(T value)**
  * 내부 값이 value인 Optional 객체 반환
  * 만약 value가 null인 경우 `NullPointerException` 발생
* **Optional.ofNullable(T value)**
  * 가장 자주 쓰이는 Optional 생성 방법
  * value가 null이면, empty Optional을 반환하고, 값이 있으면 Optional.of로 생성
* **T get()**
  * Optional 내의 값을 반환
  * 만약 Optional 내부 값이 null인 경우 `NoSuchElementException` 발생
* **boolean isPresent()**
  * Optional 내부 값이 null이면 false, 있으면 true
  * Optional 내부에서만 사용해야하는 메서드라고 생각
* **boolean isEmpty()**
  * Optional 내부의 값이 null이면 true, 있으면 false
  * isPresent() 메서드의 반대되는 메서드
* **void ifPresent(Consumer<? super T> consumer)**
  * Optional 내부의 값이 있는 경우 consumer 함수를 실행
* **Optional<T> filter(Predicate<T> predicate)**
  * Optional에 filter 조건을 걸어 조건에 맞을 때만 Optional 내부 값이 있음
  * 조건이 맞지 않으면 Optional.empty를 리턴
* **Optional<U> map(Funtion<? super T, ? extends U> f)**
  * Optional 내부의 값을 Function을 통해 가공
* **T orElse(T other)**
  * Optional 내부의 값이 있는 경우 그 값을 반환
  * Optional 내부의 값이 null인 경우 other을 반환
* **T orElseGet(Supplier<? extends T> supplier)**
  * Optional 내부의 값이 있는 경우 그 값을 반환
  * Optional 내부의 값이 null인 경우 supplier을 실행한 값을 반환
* **T orElseThrow()**
  * Optional 내부의 값이 있는 경우 그 값을 반환
  * Optional 내부의 값이 null인 경우 `NoSuchElementException` 발생
* **T orElseThrow(Supplier<? extends X> exceptionSupplier)**
  * Optional 내부의 값이 있는 경우 그 값을 반환
  * Optional 내부의 값이 null인 경우 exceptionSupplier을 실행하여 Exception 발생



# Java8 이전의 코드

```java
public class School {
    private ClassRoom classRoom;
}

public class ClassRoom {
    private Teacher teacher;
    private List<Student> students;
}

public class Teacher {
    private Subject subject;
    private String name;
}

public class Subject {
    private String subjectName;
}
```



학교, 교실, 선생님, 과목이라는 클래스가 주루룩 있을 때 이 학교의 교실의 선생님의 과목을 반환하는 코드를 작성하면

```java
school.getClassRoom().getTeacher().getSubject().getSubjectName();
```

위와 같은 코드를 작성 할 수 있는데, 위와 같은 코드는 전혀 null-safe 하지 않은 코드가 된다.
그렇게 null처리를 추가해보면..

```java
if(school != null) {
    ClassRoom classRoom = school.getClassRoom();
    if(classRoom != null) {
        Teacher teacher = classRoom.getTeacher();
        if(teacher != null) {
            Subject subject = teacher.getSubject();
            if(subject != null) {
                String subjectName = subject.getSubjectName();
                return subjectName;
            }
        }
    }
}
return null;
```

대충 위와 같은 if 지옥이 발생하게 된다. 조금 다듬어서..

```java
if(school == null) {
    return null;
}

ClassRoom classRoom = school.getClassRoom();
if(classRoom == null) {
    return null;
}

Teacher teacher = classRoom.getTeacher();
if(teacher == null) {
    return null;
}

Subject subject = teacher.getSubject();
if(subject == null) {
    return null;
}

return subject.getSubjectName();
```

이정도로 바꿀 수 있겠지만, 만족스러운 코드는 아니다.



# Optional을 이용한 코드

```java
Optional.ofNullable(school).map(School::getClassRoom)    //Optional<School>
                           .map(ClassRoom::getTeacher)   //Optional<ClassRoom>
                           .map(Teacher::getSubject)     //Optional<Teacher>
                           .map(Subject::getSubjectName) //Optional<Subject>
                           .orElse(null);
```

* map 메서드에서 null인 경우 empty Optional을 반환하기 때문에 NPE가 발생하지 않음
* 최후에 orElse 구문에서 Optional 내부의 값이 null인 경우 파라미터로 들어간 null을 반환하기 때문에
  NPE에 안전하다.



# Optional을 활용한 예제

## Optional을 사용하지 않은 예제

```java
public static <E extends Comparable<E>> E max(Collection<E> c) {
    if(c.isEmpty()) {
        throw new IllegalArgumentException("빈 컬렉션");
    }
    
    E result = null;
    for(E e : c) {
        if(result == null || e.compareTo(result) > 0) {
            result = Objects.requiredNonNull(e);
        }
    }
    
    return result;
}
```



## Optional을 사용한 예제

```java
public static <E extends Comparable<E>> Optional<E> max(Collection<E> c) {
    if(c.isEmpty()) {
        return Optional.empty();
    }
    
    E result = null;
    for(E e : c) {
        if(result == null || e.compareTo(result) > 0) {
            result = Objects.requiredNonNull(e);
        }
    }
    
    return Optional.of(result);
}
```

* Optional을 반환하여 client에서 더욱 더 유연하게 로직을 작성할 수 있다.
* Optional.of에 null을 넣으면 `NullPointerException` 이 발생하니 주의해야 한다.
* Optional을 리턴하는 메서드에서는 null을 리턴해서는 안된다. (Optional의 취지와 맞지 않기 때문)



## Stream을 이용한 버전

```java
public static <E extends Comparable<E>> Optional<E> max(Collection<E> c) {
    return c.stream().max(Comparator.naturalOrder());
}
```



# 왜 Optional을 사용해야 하는가?

기존 로직에서는 null을 반환하거나 예외를 던졌는데, Optional이 등장하고 나서는 Optional을 사용하는 것이 좋다고 한다.  
그걸 구별하는 기준은 무엇일까?  Optional은 검사 예외와 취지가 비슷하다.  
즉, 반환값이 있을 수도 있고, 없을 수도 있음을 API 사용자에게 명확히 알려준다.  
만약 비검사 예외를 던지거나 null을 반환한다면 API 사용자가 그 사실을 인지하지 못해 런타임에서 예상치 못한 장애로 발전할 수 있다.  
하지만 검사 예외(checked Exception)을 던지면 사용자 코드에서는 try-catch 구문을 통해 예외를 처리하는 로직을 추가해야 한다.  
비슷하게, 메서드가 Optional을 반환한다면 클라이언트는 값을 받지 못했을 때의 취할 행동을 선택해야 한다.  
그중 하나는 기본값을 설정하는 것이다.



## Optional 활용1 - 기본값(defalut)를 정해둘 수 있다.

```java
String lastWordInLexicon = max(words).orElse("단어 없음..");
```

* 실제로 예외를 던진 것이 아니라, empty Optional이 리턴되기 때문에 
  예외 생성 비용이 들지 않는다.



## Optional 활용2 - 원하는 예외를 던질 수 있다.

```java
Toy myToy = max(toys).orElseThrow(TemperTantrumException::new);
```

* 값이 없는 경우 원하는 예외를 던질 수 있다.

## Optional 활용3 - 항상 값이 채워져 있는 경우

```java
Element lastNobleGas = max(Elements.NOBLE_GASES).get();
```

* 값이 없는 경우에는 NoSuchElementException이 발생하니 반드시 값이 있는 경우에만 사용해야 한다.

## Optional 활용4 - 기본값을 설정하는 비용이 큰 경우

```java
Connection connection = getConnection(datasource).orElseGet(() -> getLocalConnection());
```

* 기본값을 설정하는 비용이 아주 커서 부담이 되는 경우 orElseGet을 사용하면,  
  값이 처음 필요할 때 Supplier를 사용해 생성하므로 초기 생성비용을 낮출 수 있다.



# Optional 안티 패턴

## Collection, Stream, 배열은 Optional로 감싸지 말자

Optional<List<T>>를 반환하기 보다는 빈 ArrayList를 반환하는 것이 좋다. 그렇게 하면 클라이언트 코드에서 Optional 처리 코드를 넣지 않아도 된다.



## Optional을 Map의 키나 값으로 사용하지 말자

만약 Optional을 Map에서 사용한다면 모호한 상황이 발생한다. 

* Key 자체가 없는 경우
* Key는 있지만, 속이 빈 Optional인 경우

쓸데없이 복잡도만 높아지게 되고 전혀 쓸모없는 짓이니 사용하지 말자.  
일반화 하자면, **Optional은 Collection의 키, 값, 원소나 배열의 원소로 사용하는게 적절한 상황은 거의 없다.**



## isPresent()를 사용하지 말자

위에서 설명 했듯이 isPresent()는 Optional 객체 내부의 값이 있는경우 true, 없는 경우 false를 반환한다.  
위의 학교예제를 잠시 따오면..

```java
if(school.isPresent()) {
    Optional<ClassRoom> classRoom = school.getClassRoom();
    if(classRoom.isPresent()) {
        Optional<Teacher> teacher = classRoom.getTeacher();
        if(teacher.isPresent()) {
            Optional<Subject> subject = teacher.getSubject();
            if(subject.isPresent()) {
                String subjectName = subject.getSubjectName();
                return subjectName;
            }
        }
    }
}
return null;
```

이와 같이 사용 될 수 있다.  기존의 if 지옥과 별 다를게 없는 로직이며, Optional의 취지를 제대로 이해하지 못한 로직이다.

```java
Optional.ofNullable(school).map(School::getClassRoom)    //Optional<School>
                           .map(ClassRoom::getTeacher)   //Optional<ClassRoom>
                           .map(Teacher::getSubject)     //Optional<Teacher>
                           .map(Subject::getSubjectName) //Optional<Subject>
                           .orElse(null);
```

반드시 이런식으로 사용하도록 하는 것이 좋겠다.



# 추가 내용

* 박싱된 기본타입을 사용할 때에는 OptionalInt, OptionalDouble, OptionalLong을 사용하자
  * 박싱된 기본타입을 담는 Optional은 기본타입 보다 무거울 수 밖에 없다. 
  * 따라서 OptionalInt, OptionalDouble, OptionalLong을 사용하는 것이 조금 더 낫다.
* 값을 반환하지 못할 가능성이 있고, 호출할 때마다 반환값이 없을 가능성을 염두에 둬야 하는 메서드라면  
   Optional을 반환해야 하는 상황일 수 있다.
* Optional을 반환값 이외의 용도로 쓰는 경우는 거의 없다.  



# 참고

* Effective Java 3rd Edition - Item 55. 옵셔널 반환은 신중히 하라