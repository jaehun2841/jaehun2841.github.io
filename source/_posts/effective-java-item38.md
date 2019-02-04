---
title: Item 38. 확장할 수 있는 열거 타입이 필요하면 인터페이스를 사용하라
catalog: true
Categories:
  - Effective-Java
tags:
  - Effective-Java
typora-root-url: effective-java-item38
typora-copy-images-to: effective-java-item38
date: 2019-02-04 18:54:37
subtitle:
header-img:
---

# 서론

타입 안전 열거 패턴은 확장이 가능하나, 열거 타입은 확장을 할 수 없다  
다시 말해 타입 안전 열거 패턴은 값을 그대로 가져온 다음 값을 더 추가하여 다른 목적으로 쓸 수 있지만,  
열거 타입은 그럴 수 없다.

하지만 열거타입도 확장할 수 있는 방법이 한 가지 존재한다.  
기본적인 아이디어는 열거 타입이 인터페이스를 구현할 수 있다는 사실을 이용하는 것이다.



# 인터페이스를 이용한 확장 가능 열거 타입

```java
public interface Operation {
    double apply(double x, double y);
}
```

```java
public enum BasicOperation implements Operation {

    PLUS("+") {
        @Override
        public double apply(double x, double y) {
            return x + y;
        }
    },
    MINUS("-") {
        @Override
        public double apply(double x, double y) {
            return x - y;
        }
    },
    TIMES("*") {
        @Override
        public double apply(double x, double y) {
            return x * y;
        }
    },
    DIVIDE("/") {
        @Override
        public double apply(double x, double y) {
            return x / y;
        }
    };

    private final String symbol;

    BasicOperation(String symbol) {
        this.symbol = symbol;
    }
}

```

열거 타입인 BasicOperation은 확장할 수 없지만 인터페이스인 Operation은 확장할 수 있고, 이 인터페이스를 연산의 타입으로 사용하면 된다.



## 다른 열거 타입 추가

```java
public enum ExtendedOperation implements Operation {

    EXP("^") {
        @Override
        public double apply(double x, double y) {
            return Math.pow(x, y);
        }
    },
    REMAINDER("%") {
        @Override
        public double apply(double x, double y) {
            return x % y;
        }
    };

    private final String symbol;

    ExtendedOperation(String symbol) {
        this.symbol = symbol;
    }
}
```

Operation 인터페이스를 구현하면 다른 열거 타입에서도 인터페이스를 구현하여 기능을 확장할 수 있다.



# 열거 타입 확인
## Enum타입을 넘겨 순회하기

```java
public static void main(String[] args) {
        double x = 4;
        double y = 2;
        test(ExtendedOperation.class, x, y);
    }

    public static <T extends Enum<T> & Operation> void test(Class<T> opEnumType, double x, double y) {

        for (Operation op : opEnumType.getEnumConstants()) {
            System.out.printf("%f %s %f = %f%n", x, op, y, op.apply(x, y));
        }
    }
```
* <T extends Enum<T> & Operation> : 타입이 Enum타입이면서, Operation을 구현하는 클래스



## Collection<? extends Operation> 넘겨 순회하기
```java
public static void main(String[] args) {
        double x = 4;
        double y = 2;
        test2(Arrays.asList(ExtendedOperation.values()), x, y);
    }
    
    public static void test2(Collection<? extends Operation> opSet, double x, double y) {
        for (Operation op : opSet) {
            System.out.printf("%f %s %f = %f%n", x, op, y, op.apply(x, y));
        }
    }
```
* 열거 타입의 리스트를 넘겨 <? extends Operation>인 한정적 와일드 카드 타입으로 지정



# 요약

* 확장할 수 있는 열거 타입이 필요한 경우 인터페이스를 정의하여 구현하자
* 열거 타입끼리는 상속이 되지 않는다.
* 여러 열거 타입 간 공유하는 기능이 있으면, 클래스나 도우미 메서드로 분리하자



# 참고
* Effective Java 3rd Edition - Item 38. 확장할 수 있는 열거 타입이 필요하면 인터페이스를 사용하라
