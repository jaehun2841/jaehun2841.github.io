---
title: Item 60. 정확한 답이 필요하다면 float와 double은 피하라
catalog: true
Categories:
  - Effective-Java
tags:
  - Effective-Java
typora-root-url: effective-java-item60
typora-copy-images-to: effective-java-item60
date: 2019-02-28 19:34:22
subtitle:
header-img:
---

# 서론

float와 double 타입은 과학과 공학 계산용으로 설계되었다.  
이진 부동소수점 연산에 쓰이며, 넓은 범위의 수를 빠르게 정밀한 **근사치** 로 계산하도록 세심하게 설계되었다.  
따라서 정확한 결과가 필요할 때에는 사용해선 안된다  
**float와 double 타입은 특히 금융 관련 계산과는 맞지 않는다.**  
0.1 혹은 10의 거듭 제곱 수를 표현할 수 없기 때문이다.



# 예시 - 금융 계산에 부동소수 타입을 사용

```java
public static void main(String[] args) {
    double funds = 1.00;
    int itemBought = 0;
    for(double price = 0.10; funds >= price; price += 0.10) {
        funds -= price;
        itemsBought++;
    }
    
    System.out.println(itemBought + "개 구입");
    System.out.println("잔돈(달러): " + funds);
}
```

* 프로그램의 실행결과는 사탕 3개를 구입한 후 잔돈은 0.39999999999999달러가 남는다고 나온다.
* 잘못된 결과이며, 올바른 결과를 위해서는 **BigDecimal, int, long** 을 사용해야 한다.



## BigDecimal을 사용한 코드 - 속도가 느리고 쓰기 불편하다.

```java
public static void main(String[] args) {
    final BigDecimal TEN_CENTS = new BigDecimal(".10");
    
    int itemBought = 0;
    BigDecimal funds = new BigDecimal("1.00");
    for(BigDecimal price = TEN_CENT; funds.compareTo(price) >= 0; price = price.add(TEN_CENTS)) {
        funds = funds.subtract(price);
        itemsBought++;
    }
    
    System.out.println(itemBought + "개 구입");
    System.out.println("잔돈(달러): " + funds);
}
```

* 이 프로그램의 실행결과는 사탕 4개 구입 후 잔돈이 0달러가 남는다. 올바른 답이다.
* BigDecimal에는 2가지 단점이 있다.
  * 기본 타입보다 쓰기가 훨씬 불편하고, 느리다. 단발성 계산이라면 문제는 아니지만 쓰기 불편하다
  * BigDecimal의 대안으로 int, long을 사용해도 된다. 하지만 소수점을 직접 관리해야 한다.



## int를 사용한 코드 - Cent로 문제를 해결

```java
public static void main(String[] args) {
    int itemBought = 0;
    int funds = 100;
    for(int price = 10; funds >= price; price += 10) {
        funds -= price;
        itemBought++;
    }
    
    System.out.println(itemBought + "개 구입");
    System.out.println("잔돈(달러): " + funds);
}
```

* int로 사용하면 BigDecimal 보다는 깔끔하고 정확한 답을 얻을 수 있다.



# 정리

* 정확한 답이 필요한 계산에는 float나 double을 피해야한다.
* 소수점 추적은 시스템에 맡기고 코딩시의 불편함이나 성능저하가 중요하지 않다면 BigDecimal을 사용하라
* BigDecimal은 8가지 반올림 모드를 제공하므로 반올림을 거의 완벽하게 제어할 수 있다.
* 성능이 중요하고 소수점을 직접 추적할 수 있고 숫자가 너무 크지 않다면 int나 long을 사용하라
* 숫자를 9자리 10진수로 표현한다면 int를 사용하라
* 숫자를 18자리 10진수로 표현할 수 있다면 long을 사용하라
* 숫자가 18자리가 넘어가면 BigDecimal을 사용해야 한다.



# 참고

* Effective Java 3rd Edition - Item 60. 정확한 답이 필요하다면 float와 double은 피하라