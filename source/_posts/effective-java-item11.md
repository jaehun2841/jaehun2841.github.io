---
title: Item 11. equals를 재정의하려거든 hashcode도 재정의하라
catalog: true
Categories:
  - Effective-Java
tags:
  - Effective-Java
typora-root-url: effective-java-item11
typora-copy-images-to: effective-java-item11
date: 2019-01-12 22:22:20
subtitle:
header-img:
---

# 서론
`equals를 재정의한 클래스에서는 hashcode도 재정의 해야한다.`  
그렇지 않으면 hash를 사용하는 HashMap, HashSet과 같은 컬렉션의 원소로 사용될 때 문제가 발생할 것이다.

이번 글을 읽기 전에 아래 글을 먼저 읽어 해시, 해시테이블에 대한 용어를 먼저 익히는 것이 도움이 될 것 같다.
* [해쉬 테이블의 이해와 구현 (Hashtable) - 조대협의 블로그](http://bcho.tistory.com/1072)
* [Java HashMap은 어떻게 동작하는가? - Naver D2](https://d2.naver.com/helloworld/831311)

# hashcode의 규약
* equals비교에 사용되는 정보가 변경되지 않았다면, 객체의 hashcode 메서드는 몇번을 호출해도 항상 일관된 값을 반환해야 한다.  
(단, Application을 다시 실행한다면 값이 달라져도 상관없다. (메모리 소가 달라지기 때문))
* equals메서드 통해 두 개의 객체가 같다고 판단했다면, 두 객체는 똑같은 hashcode 값을 반환해야 한다.
* equals메서드가 두 개의 객체를 다르다고 판단했다 하더라도, 두 객체의 hashcode가 서로 다른 값을 가질 필요는 없다. (Hash Collision)  
단, 다른 객체에 대해서는 다른 값을 반환해야 해시테이블의 성능이 좋아진다.

# equals 메서드는 재정의했지만, hashcode를 재정의하지 않은 경우
hashcode의 규약 2번째 조건을 위반하는 행위이다.  
Effective Java 11장에서 설명하는 PhoneNumber 클래스를 생성해 테스트 해 보았다.
```java
@RunWith(JUnit4.class)
public class HashCodeTest {

    @Test
    public void hashcode_재정의() {
        HashMap<ExtendedPhoneNumber, String> map = new HashMap<>();
        map.put(new ExtendedPhoneNumber(707, 867, 5307), "제니");
        System.out.println("Instance 1 hashcode : " + new ExtendedPhoneNumber(707, 867, 5307).hashCode());
        System.out.println("Instance 2 hashcode : " + new ExtendedPhoneNumber(707, 867, 5307).hashCode());
        Assert.assertEquals(map.get(new ExtendedPhoneNumber(707, 867, 5307)), "제니");
    }

    @Test
    public void hashcode_재정의안함() {
        HashMap<PhoneNumber, String> map = new HashMap<>();
        map.put(new PhoneNumber(707, 867, 5307), "제니");
        System.out.println("Instance 1 hashcode : " + new PhoneNumber(707, 867, 5307).hashCode());
        System.out.println("Instance 2 hashcode : " + new PhoneNumber(707, 867, 5307).hashCode());
        Assert.assertNotEquals(map.get(new PhoneNumber(707, 867, 5307)), "제니");
    }

    @NoArgsConstructor
    @AllArgsConstructor
    public static class PhoneNumber {
        protected int firstNumber;
        protected int secondNumber;
        protected int thirdNumber;

        @Override
        public boolean equals(Object o) {
            if (!(o instanceof PhoneNumber)) {
                return false;
            }

            PhoneNumber p = (PhoneNumber) o;
            return this.firstNumber == p.firstNumber &&
                    this.secondNumber == p.secondNumber &&
                    this.thirdNumber == p.thirdNumber;
        }
    }

    @NoArgsConstructor
    public static class ExtendedPhoneNumber extends PhoneNumber {

        public ExtendedPhoneNumber(int firstNumber, int secondNumber, int thirdNumber) {
            super(firstNumber, secondNumber, thirdNumber);
        }

        @Override
        public int hashCode() {
            int c = 31;
            int hashcode = Integer.hashCode(firstNumber);
            hashcode = c * hashcode + Integer.hashCode(secondNumber);
            hashcode = c * hashcode + Integer.hashCode(thirdNumber);
            return hashcode;
        }
    }
}
```
## Test.1 hashcode를 재정의 하지 않은 경우
PhoneNumber 클래스에서는 객체 동치성 체크를 위해 equals메서드를 재정의했다. **하지만 hashcode 메서드는 재정의 하지 않았다.** 
new를 통해 새로운 객체를 만들 때마다 늘 다른 hashcode를 리턴하여, 실제 HashMap에 key로 PhoneNumber클래스를 삽입하더라도  
hashcode가 다르기 때문에 다른 버킷에가서 데이터를 조회 하려고해서 결과적으로 null이 나와 기댓값인 `"제니"`를 만족하지 못했다. 

실제 로그를 찍어보니
```
Instance 1 hashcode : 1686369710
Instance 2 hashcode : 194706439
```
두 개의 다른 객체에 대해 다른 hashcode를 리턴하고 있었다.

## Test.2 hashcode를 재정의 한 경우
ExtendPhoneNumber 클래스에서는 PhoneNumber를 상속 받은 뒤, hashCode 메서드를 재정의하였다.  
(equals메서드는 PhoneNumber의 equals를 그대로 사용하였다.) 

실제 로그를 찍어보니
```
Instance 1 hashcode : 711611
Instance 2 hashcode : 711611
```
두 개의 ExtendPhoneNumber 객체에 대해 같은 hashcode를 리턴하고 있었다.  
그렇기 때문에 HashMap에 삽입한 ExtendPhoneNumber 객체에 대해 new ExtendPhoneNumber()로 조회를 하니 같은 hashCode의 버킷을 조회하여  
`"제니"`라는 데이터를 얻어올 수 있었다.


# 최악의 hashcode 구현
```java
@Override
public int hashCode() {
  return 42;
}
```
이 코드는 동치인 모든 객체에서 똑같은 해시코드를 반환하니 적법한 해시코드 처럼 보인다.  
하지만, 모든 객체에 대해 똑같은 해시코드를 반환하니 모든 객체가 같은 해시테이블 버킷에 담겨 연결리스트(Linked List)처럼 동작하게 된다.  
평균 수행시간이 O(1)에서 O(n)으로 느려져서, 성능이 매우 낮아질 뿐더러 버킷에 대한 overflow가 발생하는 경우 데이터가 누락될 수도 있다.

## hashcode가 같으면 HashMap에서 어떻게 동작할까?
```java
@RunWith(JUnit4.class)
public class HashCodeTest2 {

    @Test
    public void hashcode_재정의() {
        HashMap<ExtendedPhoneNumber, String> map = new HashMap<>();
        map.put(new ExtendedPhoneNumber(707, 867, 5307), "제니");
        System.out.println("Instance 1 hashcode : " + new ExtendedPhoneNumber(707, 867, 5307).hashCode());
        System.out.println("Instance 2 hashcode : " + new ExtendedPhoneNumber(707, 867, 5301).hashCode());
        //다른 객체를 넣어 데이터를 조회해 보았다.
        Assert.assertEquals(map.get(new ExtendedPhoneNumber(707, 867, 5301)), "제니");
    }

    @NoArgsConstructor
    @AllArgsConstructor
    public static class PhoneNumber {
        protected int firstNumber;
        protected int secondNumber;
        protected int thirdNumber;

        @Override
        public boolean equals(Object o) {
            if (!(o instanceof PhoneNumber)) {
                return false;
            }

            PhoneNumber p = (PhoneNumber) o;
            return this.firstNumber == p.firstNumber &&
                    this.secondNumber == p.secondNumber &&
                    this.thirdNumber == p.thirdNumber;
        }
    }

    @NoArgsConstructor
    public static class ExtendedPhoneNumber extends PhoneNumber {

        public ExtendedPhoneNumber(int firstNumber, int secondNumber, int thirdNumber) {
            super(firstNumber, secondNumber, thirdNumber);
        }

        @Override
        public int hashCode() {
            return 42;
        }
    }
}
```

```
Instance 1 hashcode : 42
Instance 2 hashcode : 42

java.lang.AssertionError: 
Expected :null
Actual   :제니
 <Click to see difference>
```

이 결과로 보아하니, hashcode가 같으면 같은 버킷에 LinkedList로 저장되면서 객체에 대해 equals 체크를 통해 데이터를 조회하는 것으로 판단 된다.

> 설사 두 인스턴스가 같은 버킷에 담았더라도, hashmap.get메서드는 null을 반환한다.  
만약 hashcode가 다른경우에는 동치성비교를 실행하지 않도록 최적화 되어있기 때문이다.  
hashcode가 같더라도 동치성비교(equals)를 실행하여 객체에 대한 값(value)를 조회한다.

# 좋은 해시 함수 만들기
좋은 해시 함수는 서로 다른 인스턴스에 대해 다른 해시코드를 반환한다.  
이게 바로 hashcode의 세번째 규약이 요구하는 속성이다.  
이상적인 해시 함수는 주어진 (서로 다른) 인스턴스들을 32비트 정수 범위에 균일하게 분배해야 한다.

```java
@Override
public int hashCode() {
    int c = 31;
    //1. int변수 result를 선언한 후 첫번째 핵심 필드에 대한 hashcode로 초기화 한다.
    int result = Integer.hashCode(firstNumber);

    //2. 기본타입 필드라면 Type.hashCode()를 실행한다
    //Type은 기본타입의 Boxing 클래스이다.
    result = c * result + Integer.hashCode(secondNumber);

    //3. 참조타입이라면 참조타입에 대한 hashcode 함수를 호출 한다.
    //4. 값이 null이면 0을 더해 준다.
    result = c * result + address == null ? 0 : address.hashCode();

    //5. 필드가 배열이라면 핵심 원소를 각각 필드처럼 다룬다.
    for (String elem : arr) {
      result = c * result + elem == null ? 0 : elem.hashCode();
    }

    //6. 배열의 모든 원소가 핵심필드이면 Arrays.hashCode를 이용한다.
    result = c * result + Arrays.hashCode(arr);

    //7. result = 31 * result + c 형태로 초기화 하여 
    //result를 리턴한다.
    return result;
}
```

* hashcode를 다 구현했다면, 이 메서드가 동치인 인스턴스에 대해 똑같은 해시코드를 반환할지 TestCase를 작성해보자
* 파생필드는 hashcode 계산에서 제외해도 된다.
* equals 비교에 사용되지 않는 필드는 반드시 제외한다.
* 31 * result를 곱하는 순서에 따라 result 값이 달라진다.
* 곱할 숫자를 31로 지정하는 이유는 31이 홀수 이면서 소수(prime)이기 때문이다. 2를 곱하는 것은 shift연산과 같은 결과를 내기 떄문에 추천하지 않는다.
* 31을 이용하면 (i << 5) - i와 같이 최적화 할 수 있다.

# hashcode를 편하게 만들어 주는 모듈
* Objects.hash()
  * 내부적으로 AutoBoxing이 일어나 성능이 떨어진다.
* Lombok의 @EqualsAndHashCode
* Google의 @AutoValue

# hashcode를 재정의 할 때 주의 할 점!
* 불변 객체에 대해서는 hashcode 생성비용이 많이 든다면, hashcode를 캐싱하는 것도 고려하자
  * 스레드 안전성까지 고려해야 한다.
* 성능을 높인답시고 hashcode를 계산할 떄 핵심필드를 생략해서는 안된다.
  * 속도는 빨라지겠지만, hash품질이 나빠져 해시테이블 성능을 떨어뜨릴 수 있다 (Hashing Collision)
* hashcode 생성규칙을 API사용자에게 공표하지 말자
  * 그래야 클라이언트가 hashcode값에 의지한 코드를 짜지 않는다.
  * 다음 릴리즈 시, 성능을 개선할 여지가 있다.

# 참고
* Effective Java 3rd Edition - Item 11. equals를 재정의하려거든 hashcode도 재정의하라