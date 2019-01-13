---
title: Shallow Copy와 Deep Copy
catalog: true
Categories:
  - Java
tags:
  - Java
  - Effective-Java
typora-root-url: java-object-copy
typora-copy-images-to: java-object-copy
date: 2019-01-13 18:59:09
subtitle:
header-img:
---

# 객체의 복사(Copy)
객체 지향 프로그래밍에서 객체를 복사하는 방법은 크게 두가지로 나뉜다.  
얇은 복사(Shallow Copy)와 깊은 복사(Deep Copy)가 있다.  
두가지 개념 모두 원본 객체를 바탕으로 새로운 객체를 만들어 낸다는 점에서는 같지만 미묘한 차이가 있다.

설명을 진행하기 전에 아래와 같은 Interface를 구현한 객체에 대해 copy가 진행된다고 가정하겠다.
```java
public interface Copyable<T> {
   T shallowCopy(T t);
   T deepCopy(T t);
}
```



![clone](./clone.jpg)



# Shallow Copy

Shallow copy란 원본 객체를 바탕으로 새로운 객체를 생성한 뒤, 내부 필드의 참조에 대해서는 원본 객체와 같은 필드의 참조를 바라보는 형태이다.

```java
@NoArgsConstructor
@AllArgsConstructor
@Getter
public class Menu implements Copyable<Menu> {

  private String name;
  private int price;
  private Recipe recipe;

  @Override
  public Menu shallowCopy(Menu menu) {
    Menu copyMenu = new Menu(menu.getName(), menu.getPrice(), menu.getRecipe());
    return copyMenu;
  }
}
```
불변 객체에 대해서는 같은 객체를 공유한다는 점에서 메모리에 대한 비용을 아낄 수 있다는 장점이 있고, deep copy보다는 빠르다.

하지만, 가변 객체에 대해서는 원본 객체의 Recipe 객체가 변경되는 경우 의도치 않게 복사한 Recipe도 변경되는 경우가 발생한다.  
이런 경우에는 Deep Copy를 통해 완전히 분리를 시켜줘야 한다.


# Deep Copy
Deep Copy 원본 객체의 타입을 바탕으로 새로운 객체를 생성한 뒤, 내부 필드가 참조하는 객체에 대해서도 새로운 객체를 만들어 주는 방식이다.

```java
@NoArgsConstructor
@AllArgsConstructor
@Getter
@Setter
public class Menu implements Copyable<Menu> {

  private String name;
  private int price;
  private Recipe recipe;

  @Override
  public Menu deepCopy(Menu menu) {
    Menu copyMenu = new Menu();
    copyMenu.setName(new String(menu.getName));
    copyMenu.setPrice(menu.getPrice());
    copyMenu.setRecipe(Recipe.deepCopy(menu.getRecipe));
    return copyMenu;
  }
}
```

위와 같이 필드에 대해 원본객체와 같은 reference를 사용하는 것이 아닌 필드의 reference에 대해서도 새로운 객체를 생성하여 복사하는 방식이다.  

이런 경우 가변객체에 대한 변경에는 안전하다는 장점이 있다.  
서로 다른 객체를 다루기 떄문에 안전하다.  

하지만 객체 생성 비용이 shallow copy보다 비싸고 상대적으로 느리며, 
copy하는 객체 수가 많을 수록 메모리를 많이 점유하게 된다는 단점이 있다.


