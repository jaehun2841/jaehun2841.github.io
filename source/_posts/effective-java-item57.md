---
ㅅtitle: Item 57. 지역변수의 범위를 최소화하라
catalog: true
Categories:
  - Effective-Java
tags:
  - Effective-Java
typora-root-url: effective-java-item57
typora-copy-images-to: effective-java-item57
date: 2019-03-02 18:23:54
subtitle:
header-img:
---

# 서론

**클래스의 멤버와 접근권한을 최소화하라 (Item 15)** 와 취지가 비슷한 장이다.  
지역변수의 유효범위를 최소로 줄이면 코드 가독성과 유지보수성이 높아지고 오류 가능성은 낮아진다.  



# 가장 처음 쓰일 때 선언하라

* 사용하려면 멀었는데 미리 변수부터 선언하는 코드는 어수선하고 가독성이 좋지 못하다. 
* 막상 쓰일 시점에 되서는 무슨 타입이었는지, 무슨 값으로 초기화 했는지 기억이 나지 않을 경우가 있다. 
* 변수가 쓰이는 범위보다 너무 앞서 선언하거나, 다 쓴 뒤에도 할당해제가 되지 않고 여전히 살아있게 된다.
* 변수의 scope를 제한하지 못하면 예상하지 못한 결과를 초래할 수 있으니 주의해야 한다.



# 모든 지역변수는 선언과 동시에 초기화 하라

* 초기화에 필요한 정보가 충분하지 않다면 충분해질 때까지 선언을 미뤄야한다.
* try-catch-finally 구문에서는 예외다.
* 변수를 초기화 하는 표현식에서 checked exception을 던질 가능성이 있으면, try 절 안에서 초기화 해야한다.
* 만약, catch나 finally절에서 변수를 이용해야 한다면, try절 바로 앞에 변수를 선언해야 한다.

* 반복문에서는 반복 변수(loop variable)의 범위가 반복문의 몸체, 그리고 for 키워드와 몸체 사이 괄호 안으로 제한된다.
* 반복자 (index)를 사용해야 하는 경우에는 for-each 구문보다 전통적인 for문이 낫다.

```java
for (Element e : c) {
    // e로 무언가를 반복하는 로직을 구성
}
```

```java
for(Iterator<Element> i = c.iterator(); i.hasNext();) {
    Element e = i.next(); //e와 i로 무언가를 한다.
}
```



# while 문을 사용하는 경우에도 for문을 사용하는 것이 낫다.

```java
Iterator<Element> i = c.iterator();
while(i.hasNext()) {
    doSomeThing(i.next());
}

Iterator<Element> i2 = c2.iterator();
while(i.hasNext()) { 
    doSomeThing(i2.next());
}
```

* 복붙의 여파로 i2.hasNext()이지만 i.hasNext()를 사용하였다.
* 컴파일도 잘되고 실행도 잘되지만, 기대한 결과가 도출되지 않는다.
* for문을 포함한 for-each문을 사용하는 경우에는 이러한 문제를 컴파일 타임에 잡아준다. 
  (반복자의 유효범위가 for문의 종료와 함께 끝나기 때문이다.)

```java
for(Iterator<Element> i = c.iterator(); i.hasNext();) {
    Element e = i.next();
} 

for(Iterator<Element> i2 = c2.iterator(); i.hasNext();) {
    Element e2 = i2.next();
} 
```

* 두번째 코드의 i.hasNext()에서 i를 찾을 수 없다는 오류를 보여준다.



# 메서드를 작게 유지하고 한 가지 기능에 집중하라

* 한 메서드에서 여러 가지 기능을 처리한다면 그중 한 기능과만 관련된 지역변수라도 다른 기능을 수행하는 코드에서 접근할 수 있다.
* 단순히 메서드를 기능별로만 쪼개자





# 참고

* Effective Java 3rd Edition - Item 57. 지역변수의 범위를 최소화하라