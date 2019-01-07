---
title: Item 2. 생성자에 매개변수가 많다면 빌더를 고려하라
catalog: true
Categories:
  - Effective-Java
tags:
  - Effective-Java
typora-root-url: effective-java-item2
typora-copy-images-to: effective-java-item2
date: 2019-01-06 19:40:34
subtitle:
header-img:
---

# 서론
정적 팩터리 메서드 팩터리와 생성자에는 똑같은 제약이 하나 있다.  
선택적 매개변수가 많을 경우에 적절하게 대응하기 어렵다는 점이다.  
매개변수가 3~4개까지는 어떻게 해보겠지만 엄청 많은 경우에는 생성자를 이용해 객체를 생성하는 과정부터가 곤욕이다.

아래의 예시를 보자
```java

class NutritionFacts {

  private final int servingSize;
  private final int servings;
  private final int calories;
  private final int fat;
  private final int sodium;
  private final int carbohydrate;

  public NutritionFacts() {
  }

  public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium, int carbonhydrate) {
    this.servingSize = servingSize;    //필수
    this.servings = servings;          //필수
    this.calories = calories;          //선택
    this.fat = fat;                    //선택
    this.sodium = sodium;              //선택
    this.carbohydrate = carbohydrate;  //선택
  }
}
```
매개변수가 int만 6개로 이루어져 있다.  
그리고 default value가 정해져 있지 않기 때문에, servingSize 같은 변수는 정의하기 싫으면 0으로 해야할 지, 아니면 최소값이 있을지 알기가 어렵다.  
그리고 매개변수 선언 순서가 바뀌면 의도하지 않은 객체가 생성되기 때문에 코딩 할 때 매우 주의를 요해야 한다.

# 점층적 생성자 패턴
위와 같은 문제를 조금이나마 해결해 보려는 노력이 점층적 생성자 패턴이다.
```java
class NutritionFacts {

  private final int servingSize;
  private final int servings;
  private final int calories;
  private final int fat;
  private final int sodium;
  private final int carbohydrate;

  public NutritionFacts() {
  }

  public NutritionFacts(int servingSize, int servings) {
    this.servingSize = servingSize;    //필수
    this.servings = servings;          //필수
  }

   public NutritionFacts(int servingSize, int servings, int calories) {
     this(servingSize, servings);
     this.calories = calories;
  }

   public NutritionFacts(int servingSize, int servings, int calories, int fat) {
     this(servingSize, servings, calories);
     this.fat = fat;     
  }

  public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium, int carbonhydrate) {
    this.servingSize = servingSize;    //필수
    this.servings = servings;          //필수
    this.calories = calories;          //선택
    this.fat = fat;                    //선택
    this.sodium = sodium;              //선택
    this.carbohydrate = carbohydrate;  //선택
  }
}
```

이것도 선택사항에 대해 하나의 방법으로 작용할 수 있지만, 코드가 길어지고 가독성이 떨어지게 된다.  
실제로 매개변수의 위치에 따라 의도하지 않은 객체가 생성될 수 있기 때문에 주의를 요해야 하는 코드이다.

# Java Beans 패턴
```java
class NutritionFacts {

  private final int servingSize = -1; 
  private final int servings = -1;
  private final int calories = 0;
  private final int fat = 0;
  private final int sodium = 0;
  private final int carbohydrate = 0;

  public NutritionFacts() {}

  public void setServingSize(int servingSize) {
    this.servingSize = servingSize;
  }
  public void setServings(int servings) {
    this.servings = servings;
  }
  public void setCalories(int calories) {
    this.calories = calories;
  }
  public void setFat(int fat) {
    this.fat = fat;
  }
  public void setSodium(int sodium) {
    this.sodium = sodium;
  }
  public void setCarboHydrate(int carbohydrate) {
    this.carbohydrate = carbohydrate;
  }
```
```java

NutritionFacts nutritionFacts = new NutritionFacts();

nutritionFacts.setServingSize(240);
nutritionFacts.setServings(8);
nutritionFacts.setCalories(100);
nutritionFacts.setFat(20);
nutritionFacts.setSodium(35);
nutritionFacts.setCarboHydrate(27);
```
자바 빈즈 패턴에서는 객체 하나를 만드느느데 메서드를 여러개 호출 해야한다.  객체가 완전히 생성되기 전까지는 일관성이 무너진 상태에 놓이게 된다.  
또한 객체의 불변셩이 깨지게 되어 코드에서 버그를 생성할 수 있다.

# Builder 패턴
```java
class NutritionFacts {

  private final int servingSize;
  private final int servings;
  private final int calories;
  private final int fat;
  private final int sodium;
  private final int carbohydrate;

  public static class Builder {
    private final int servingSize;
    private final int servings;
    private final int calories = 0;
    private final int fat = 0;
    private final int sodium = 0;
    private final int carbohydrate = 0;

    public Builder(int servingSize, int servings) {
      this.servingSize = servingSize;    //필수
      this.servings = servings;          //필수
    }

    public Builder calories(int calories) {
      this.calories = calories;
    }
    public Builder fat(int fat) {
      this.fat = fat;
    }
    public Builder sodium(int sodium) {
      this.sodium = sodium;
    }
    public Builder carbohydrate(int carbohydrate) {
      this.carbohydrate = carbohydrate;
    }

    public NutritionFacts build() {
      return new NutritionFacts(this);
    }

    private NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium, int carbonhydrate) {
      this.servingSize = servingSize;    //필수
      this.servings = servings;          //필수
      this.calories = calories;          //선택
      this.fat = fat;                    //선택
      this.sodium = sodium;              //선택
      this.carbohydrate = carbohydrate;  //선택
    }
  }
```

```java
  NutritionFacts nutritionFacts = new NutritionFacts.Builder(240, 8)
                                                    .calories(100).sodium(35).carbohydrate(27).build()
```
빌더 패턴은 (파이썬과 스칼라에 있는) 명명된 선택적 매개변수를 흉내낸 것이다.  
이런 식으로 하면 Java Beans 패턴의 set 역할을 해주면서 build()를 호출 하는 시점에 변수를 freezing 시켜 불변식을 유지 할 수 있다.
하지만 시간이 지날 수록 매개 변수가 늘어날 가능성이 있음을 항상 주의 해야 한다.


# 참고
* Effective Java 3rd Edition - Item 2. 생성자에 매개변수가 많다면 빌더를 고려하라