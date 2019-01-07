---
title: Item 4. 인스턴스화를 막으려거든 private 생성자를 사용하라
catalog: true
Categories:
  - Effective-Java
tags:
  - Effective-Java
typora-root-url: effective-java-item4
typora-copy-images-to: effective-java-item4
date: 2019-01-07 20:44:23
subtitle:
header-img:
---

# 서론
단순히 static 메서드와 static 필드만을 담은 클래스를 만들고 싶을 때가 있다.  
보통 Utils 클래스를 정의할 떄 이런식으로 많이 사용하는데, 이러한 클래스는 인스턴스를 만들어서 사용하고자 만든 목적이 아니기 때문에 인스턴스화를 막아야 한다.  
생성자를 하나도 명시 하지 않으면 Java에서는 매개변수가 없는 default 생성자를 만들어 준다. 

# 추상 클래스로 만들면?
추상 클래스로 만드는 것으로는 인스턴스 화를 막을 수 없다.  
단순히 상속을 통해 인스턴스를 만들 수 있기 때문이다. 오히려 abstract 클래스는 하위클래스를 만들어서 사용하라는 뉘앙스가 더 강하다.

# private 생성자를 만들자
private 생성자를 만드는 것 만으로도 인스턴스화를 막을 수 있다.
외부에서 new 키워드를 통해 인스턴스를 만들 수 없기 때문이다.

이중 보안을 하자면.. 
```java
private Utils() {
  throw new AssertionError();
}
```
이런 식으로 Error를 발생 시켜주자.

평소 코딩할 때는 Lombok을 이용해서 깔끔하게 등록해 주는 것도 방법이다.

```java
@NoArgsConstructor(access = AccessLevel.PRIVATE)
class Utils() {
}
```

# 참고
* Effective Java 3rd Edition - Item 4. 인스턴스화를 막으려거든 private 생성자를 사용하라

