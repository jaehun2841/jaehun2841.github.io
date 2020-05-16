---
title: Objects Study - Chapter4. 설계 품질과 트레이드오프
catalog: true
Categories:
  - Object-Study
tags:
  - Object-Study
typora-root-url: object-chapter4
typora-copy-images-to: object-chapter4
date: 2020-05-16 16:20:24
subtitle:
header-img:
---

# 데이터 중심의 설계
객체지향 설계에는 두 가지 방법을 이용해 시스템을 객체로 분할할 수 있다.  
*  데이터를 분할의 중심축으로 삼는 방법 (Entity를 먼저 정의하고 비지니스 로직 구현)
   *  데이터 중심의 관점에서는 객체는 **자신이 포함하고 있는 데이터를 조작하는 데 필요한 오퍼레이션을 정의**
   *  객체는 독립된 데이터 덩어리처럼 바라본다
* 책임을 분할의 중심축으로 삼는 방법 (Interface 기반의 비지니스 로직을 구현하고 필요한 속성을 정의)
  * 객체는 다른 객체가 요청할 수 있는 **오퍼레이션을 위해 필요한 데이터만 생성**하는 정도
  * 객체랑 협력하는 공동체의 일환으로 바라본다



# 데이터 중심 설계가 안 좋은 이유
객체지향 설계는 데이터가 아닌 책임에 초점을 맞춰야한다.  

* 객체의 상태는 구현에 속한다

* 구현은 불안정하기 때문에 변화하기 쉽다

* 상태를 객체 분할의 중심축으로 삼으면 구현에 관한 세부사항이 객체의 인터페이스에 스며들어 캡슐화가 무너진다. 

* 상태 변경 -> 인터페이스의 변경 -> 의존성을 갖는 객체에게 변경 영향이 퍼짐

* 데이터에 초점을 맞추는 설계는 변경에 취약하다

* 객체의 내부 구현을 인터페이스의 일부로 만든다. **(캡슐화 위반)**

  * getter/setter와 같은 메서드가 퍼블릭으로 생성되어 어디서든 접근이 가능하다




# 데이터 중심 설계의 문제점

데이터 중심 설계가 변경에 취약한 이유는 두 가지이다.

* 데이터 중심의 설계는 본질적으로 너무 이른 시기에 데이터에 관해 결정하도록 한다.
  * 데이터 중심으로 클래스를 구성하다 보면 책임에 대해 무감각해져 캡슐화를 위반하기 쉽다.
  * 데이터 중심 관점에서는 객체란 그냥 데이터 덩어리이기 때문에 별도로 분리된 객체에서 로직을 구현하게 된다.
    * 이는 getter/setter의 남용을 불러온다.
    * 이로 인해 캡슐화가 무너진다.
  * 데이터를 먼저 잡고 오퍼레이션을 잡기 때문에 데이터에 대한 정보가 인터페이스로 외부에 표현된다.
  * 따라서 객체 내부 구현이 객체의 인터페이스를 어지럽히고 객체의 응집도와 결합도에 나쁜 영향을 준다. (변경에 취약)
* 데이터 중심의 설계에서는 협력이라는 문맥을 고려하지 않고 객체를 고립시킨 채 오퍼레이션을 결정한다.
  * 올바른 객체지향 설계의 무게중심은 항상 객체 내부가 아니라 외부에 포커스가 가야한다.
  * 내부에서 어떤 상태를 가지고 뭘하는지는 부가적인 문제이다.
  * 항상 다른 객체와 어떻게 협력하는지 부터 생각해야 한다.
  * 데이터 중심 설계에서는 이미 구현(=데이터)가 결정된 상태에서 다른 객체와의 협력을 고려하기 때문에 이미 구현된 객체의 인터페이스를 억지로 끼워 맞출 수 밖에 없다.
  * 따라서 내부가 변경되면 협력하는 객체 모두가 영향을 받게 된다.



# 데이터 중심 설계에서 빠지는 착각
데이터 중심으로 객체를 설계하다보면 객체가 필요한 데이터에 집중하게 된다.  
이는 예전 DB 기반의 설계를 하던 것과 유사하다

```java
public class Movie {
  private String title;
  private Duration runningTime;
  private Money fee;
  private List<DiscountCondition> discountConditions;
  private MovieType movieType;
  private Money discountAmount;
  private double discountPercent;
  
  public String getTitle() {
    return this.title;
  }
  
  public Duration getRunningTime() {
    return this.runningTime;
  }
  
  public Money getFee() {
    return this.fee;
  }
  
  public void setTitle(String title) {
    this.title = title;
  }
  
  public void setRunningTime(Duration runningTime) {
    this.runningTime = runningTime;
  }

  public void setFee(Money fee) {
    this.fee = fee;
  }
}
```

객체지향의 가장 중요한 원칙은 캡슐화가 맞다.  
내부 데이터가 객체를 빠져나가 외부의 다른 객체들을 오염시키는 것을 막아야 한다.  
이를 위한 가장 간단한 방법은 위의 코드와 같이 접근자(getter) / 수정자(setter)를 추가하는 것이다.  

위와 같은 코드는 직접 객체의 내부에 접근할 수 없기 때문에 캡슐화의 원칙을 지키고 있는 것 처럼 보인다.  
하지만 위의 예시는 객체 내부 상태에 대한 어떤 정보도 캡슐화하지 못한다.  
getter/setter는 **객체 내부에 Title, RunningTime, Fee라는 인스턴스 변수가 존재한다는 사실을  public 인터페이스에 노골적으로 드러낸다.**

캡슐화를 한다고 코드를 위와같이 짰지만 캡슐화를 위반하는 코드가 되는 셈이다.

# 캡슐화
객체지향에서 가장 중요한 원리는 캡슐화다.  
* 외부에서 알 필요가 없는 부분을 감춤으로써 대상을 단순화하는 추상화의 한 종류다 
* 불안정한 구현의 세부 사항을 안정적인 인터페이스 뒤로 숨기는 것이다.

* 캡슐화가 중요한 이유는 불안정한 부분과 안정적인 부분을 분리해서 변경의 영향을 통제할 수 있기 때문이다.
* 즉, 캡슐화란 변경 가능성이 높은 부분을 객체 내부로 숨기는 추상화 기법이다.


## 구현과 인터페이스
* 구현이란 변경될 가능성이 높은 부분을 의미한다
  * 변경될 가능성이 높은 부분은 객체 내부로 숨겨야 한다.
  * 한 곳에서 일어난 변경이 전체 시스템에 영향을 끼치지 않도록 파급효과를 조절하기 위해서다
* 인터페이스란 상대적으로 안정적인 부분이다.



# 응집도와 결합도

객체 지향에서의 좋은 설계란, 높은 응집도와 낮은 결합도로 가진 모듈로 구성된 설계를 의미한다.

* 응집도는 모듈에 포함된 내부 요소들이 연관돼 있는 정도를 나타낸다.
  * 모듈 내의 요소들이 하나의 목적을 위해 긴밀하게 협력한다면, 그 모듈은 높은 응집도를 가진다.
  * 모듈 내의 요소들이 서로 다른 목적을 추구 한다면 그 모듈은 낮은 응집도를 가진다.
  * 객체지향 관점에서 응집도는 객체 또는 클래스에 얼마나 높은 책임들을 할당했는지 나타낸다.
  * 쉽게 말하면 객체 하나가 이 일 저 일 다하면 응집도가 낮은것이다.
* 결합도는 의존성의 정도를 나타내며 다른 모듈에 대해 얼마나 많이 알고 있는지를 나타낸다.
  * A 모듈이 B 모듈에 대해 너무 자세한 구현부까지 알고 있다면 두 모듈은 높은 결합도를 가진다.
  * A 모듈이 B 모듈에 대해 간단한 인터페이스정도만 알고 있다면 두 모듈은 낮은 결합도를 가진다.
  * 결합도는 객체 또는 클래스가 협력에 필요한 적절한 수준의 관계만을 유지하고 있다.

## 높은 응집도와 낮은 결합도를 추구 해야하는 이유
**변경이 발생할 때 모듈 내부에서 발생하는 변경의 정도로 측정할 수 있다.**  
* 하나의 변경에 의해 여러개의 모듈이 변경되면 **응집도가 낮다**, **결합도가 높다**
* 하나의 변경에 의해 하나의 모듈이 변경되면 **응집도가 높다**, **결합도가 낮다**

클래스의 구현이 아닌 인터페이스에 의존하도록 코드를 작성해야 낮은 결합도를 얻을 수 있다.

## 결합도가 높아도 상관 없는 경우도 있다
일반적으로 변경될 확률이 매우 적은 안정적인 모듈에 의존하는 것은 아무런 문제가 되지 않는다.  
표준 라이브러리에 포함된 모듈이나 성숙 단계에 접어든 프레임 워크에 의존하는 경우가 여기에 속한다.  
ex) String, ArrayList와 같은 클래스는 변경될 확률이 매우 낮기 때문에 결합도에 대해 고민할 필요가 없다   

하지만 직접 작성한 코드는 항상 불안정하며 언제라도 변경될 가능성이 높기 때문에 주의해야 한다.  
**이런 높은 응집도와 낮은 응집도를 실현하기 위해서는 캡슐화를 향상 시키면 된다.**

# 캡슐화를 항상 생각하자
* 캡슐화는 설계의 제 1원칙이다.  
* 데이터 중심의 설계가 낮은 응집도와 높은 결합도라는 문제로 몸살을 앓게 된 근본적인 원인은 바로 캡슐화를 위반했기 때문이다.  
* 객체는 자신이 어떤 데이터를 가지고 있는지를 내부에서만 알아야 하고 외부에 공개하면 안된다.  (getter/setter를 만들지 마라)



## Rectangle 클래스로 보는 캡슐화의 중요성
```java
class Rectangle {
  private int left;
  private int top;
  private int right;
  private int bottom;
  
  public Rectangle(int left, int top, int right, int bottom) {
    this.left = left;
    this.top = top;
    this.right = right;
    this.bottom = bottom;
  }
  
  public int getLeft() { return left; }
  public void setLeft(int left) { this.left = left }
  public int getRight() { return right; }
  public void setRight(int right) { this.right = right }
  public int getTop() { return top; }
  public void setTop(int top) { this.top = top }
  public int getBottom() { return bottom; }
  public void setBottom(int bottom) { this.bottom = bottom }
}
```

```java
class AnyClass {
  void anyMethod(Rectangle rectangle, int multiple) {
    rectangle.setRight(rectangle.getRight() * multiple)
    rectangle.setBottom(rectangle.getBottom() * multiple)
    rectangle.setLeft(rectangle.getLeft() * multiple)
    rectangle.setTop(rectangle.getTop() * multiple)
  }
}
```

위와 같은 코드는 많은 문제점이 있다. 
* `코드중복`이 발생할 확률이 높다.
  * 다른 곳에서도 사각형의 너비와 높이를 증가시키는 코드가 필요하면 또 이 코드를 작성해야 한다.
  * 악의 근원인 setter를 남발하고 있다.
* 변경에 취약하다
  * Rectangle이 right와 bottom 대신 length와 height를 이용해서 사각형을 표현하게 되면 여러 부분에 코드를 수정해야 한다.
  * 이런 변경은 기존 getter/setter를 사용하던 코드 전반적으로 영향을 미치게 된다



해결 방법은 캡슐화를 강화시키는 것이다.

```java
class Rectangle {
  public void enlarge(int multiple) {
    right *= multiple;
    bottom *= multiple;
  }
}
```

자신의 크기를 코드 외부에서 증가시키는 것이 아닌 Rectangle 클래스 내부에서 스스로 할 수 있도록 **책임을 이동하였다.**



# 캡슐화의 진정한 의미

캡슐화는 단순히 객체 내부의 데이터를 외부로부터 감추는 것 이상의 의미를 가진다.  
사실 상 변경될 수 있는 어떤 것이라도 감추는 것이 진정한 캡슐화이다.  
내부 속성을 외부로 부터 감추는 것은 **데이터 캡슐화**라고 불리는 캡슐화의 한 종류일 뿐이다.  
**만약 클래스 내부 구현의 변경으로 인해 외부의 객체가 영향을 받는 다면 그것은 캡슐화를 위반한 것이다.**  
정리하자면 캡슐화란 무엇이든 변하는 것은 감추는 것이다.




# 참고
* ​	Objects(코드로 이해하는 객체지향 설계) - Chapter4. 설계 품질과 트레이드오프

