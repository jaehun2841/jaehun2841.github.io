---
layout: posts
title: SpEL Expression(1)
catalog: true
Categories:
- Spring
tags:
- Spring
- SpEL
date: 2018-11-21 14:11:42
typora-root-url: 2018-11-21-spel-expression
typora-copy-images-to: 2018-11-21-spel-expression
---

# 들어가며

Spring Cache Abstraction 기능을 사용하게 되면서, key modeling 시, 필요한 SpEL에 대해 알아보게 되었다.
기본적인 설명과 예제는 Spring Docs를 기반으로 번역

# SpEL (<u>Sp</u>ring <u>E</u>xpression <u>L</u>anguage)이란?

Spring Expression Language (줄여서 SpEL)은 런타임에 쿼리를 지원하고 객체를 조작할 수 있는 강력한 표현식 언어이다. 언어 구문은 Unified EL과 유사하지만, 여러가지 추가 기능을 제공한다.

다른 Java에서 사용가능한 표현식 언어가 있지만, SpEL은 Spring 커뮤니티에 표현식 언어를 제공하기 위해 만들어졌다.
Spring 포트폴리오 내에서 표현식 평가 (Expression evaluation)의 기반으로 SpEL을 제공하지만, SpEL은 Spring과 직접 연계되지 않고 독립적으로 사용 할 수 있다.
독립적으로 사용하기 위해서는 Parser 같은 Bootstraping 기반 클래스에 대한 설정을 몇가지 할 필요가 있다.
하지만 웬만한 개발자들은 이런 기반 클래스에 대한 설정 없이 SpEL 표현식만 잘 작성하면 된다.

# SpEL에서 지원하는 기능

* 리터럴 표현식 (Literal Expression)
* Boolean과 관계형 Operator (Boolean and Relational Operator)
* 정규 표현식 (Regular Expression)
* 클래스 표현식 (Class Expression)
* 프로퍼티, 배열, 리스트, 맵에 대한 접근 지원 (Accessing properties, arrays, lists, maps)
* 메서드 호출 (Method Invocation)
* 관계형 Operator (Relational Operator)
* 할당 (Assignment)
* 생성자 호출 (Calling Constructors)
* Bean 참조 (Bean References)
* 배열 생성 (Array Contruction)
* 인라인 리스트/맵 (Inline List/Map)
* 삼항 연산자 (Ternary Operator)
* 변수 (Variables)
* 사용자 정의 함수 (User defined functions)
* 컬렉션 투영 (Collections Projection)
* 컬렉션 선택 (Collections Selection)
* Template 표현식 (Templated expression)

# Expression 인터페이스를 이용한 표현식 파싱

``` java
public class ExpressionTest {

    public static void main(String[] args) {

        String message = parseExpression("\"Hello World\"", String.class);
        System.out.println(message); //"Hello World"

        String message2 = parseExpression("\"Hello World\".concat('!')", String.class);
        System.out.println(message2); //"Hello World!"
    }

    public static <T> T parseExpression(String expression, Class<T> clazz) {
        ExpressionParser parser = new SpelExpressionParser();
        Expression exp = parser.parseExpression(expression);
        return exp.getValue(clazz);
    }
}
```

* 위의 코드는 SpEL표현식 문자열에 대한 파싱(Parsing) 하는 프로그램의 예제 코드이다.
* ExpressionParser Interface를 구현하는 SpelExpressionParser로 SpEL 표현식의 내용을 파싱(Parsing) 할 수 있다.
* exp.getValue() 메서드를 이용해 파싱된 결과값을 Object 타입으로 얻거나, Class정보를 파라미터로 추가해 자동으로 타입 캐스팅이 가능하다.

# EvaluationContext 인터페이스를 이용한 표현식 파싱

* Property, Method, Field에 대한 파싱을 처리
* 타입변환을 수행하는 표현식을 평가 할 때 EvaluationContext 인터페이스를 사용
* EvaluationContext의 구현체인 StandardEvaluationContext는 객체를 조작하기 위해 java reflection의 기능을 사용
* 성능을 향상 시키기 위해 java.lang.reflect의 Method, Field, Constructor를 캐싱한다.

SpEL의 더 일반적인 사용법은 특정 객체 인스턴스(root객체)에 대해 평가되는 표현식 문자열을 제공하는 것이다.
root객체를 제공하는 방식은 2가지 방식이 있다.

## root 객체를 똑같이 제공하는 경우

``` java
// Create and set a calendar
GregorianCalendar c = new GregorianCalendar();
c.set(1856, 7, 9);

// name, birthday, nationality를 파라미터로 갖는 생성자이다.
Inventor tesla = new Inventor("Nikola Tesla", c.getTime(), "Serbian");

ExpressionParser parser = new SpelExpressionParser();
Expression exp = parser.parseExpression("name"); //tesla객체의 name 프로퍼티가 파싱

//Context에 tesla객체를 넣어준다.
EvaluationContext context = new StandardEvaluationContext(tesla);
String name = (String) exp.getValue(context); //name = "Nikola Tesla"
```

* StandardEvaluationContext는 `name` 프로퍼티가 평가 될 객체를 지정하는 클래스이다.
  (위의 예제는 root객체, 즉 tesla 객체를 고정하여 사용하는 예제)
* 위의 SpelExpressionParser가 표현식을 평가 할 때, 표현식의 대한 기준을 Tesla 객체로 지정하는 부분이다.
* `parser.parseExpression("name");` 가 실행되면, tesla 객체의 name 프로퍼티가 파싱된다.
* name변수에는 `Nikola Tesla` 문자열이 리턴된다.
* 타입 캐스팅 실패 시에는, `EvaluationException`이 throw된다.

## root 객체가 계속 변경되는 경우

``` java
// Create and set a calendar
GregorianCalendar c = new GregorianCalendar();
c.set(1856, 7, 9);

// name, birthday, nationality를 파라미터로 갖는 생성자이다.
Inventor tesla = new Inventor("Nikola Tesla", c.getTime(), "Serbian");

ExpressionParser parser = new SpelExpressionParser();
Expression exp = parser.parseExpression("name"); //tesla객체의 name 프로퍼티가 파싱

String name = (String) exp.getValue(tesla);
```

getValue 메서드 호출 시에, StandardEvaluationContext를 사용하지 않고 root 객체를 직접 지정해 준다.

## 두 가지 방식의 차이

* StandardEvaluationContext를 사용하는 경우 생성 비용이 상대적으로 많이 든다.
* 반복적으로 사용하는 동안 `Field에 대해 캐싱되고 있기 때문에 표현식 파싱이 더 빠르다`는 장점이 있다.
* 설정 파일에서 SpEL을 사용하는 경우에는, root 객체를 고정하고 SpEL 표현식만 개발자에게 설정 하도록 하는 것이 좋다.


# Bean을 정의하는 표현식
* BeanDefinition을 정의하는 XML이나, Annotation 기반의 설정에는 MetaData와 SpEL표현식을 같이 사용할 수 있다.
* `#{<expression>}` 문법으로 사용한다.


## XML Based
* 프로퍼티나 생성자 Argument 값을 표현식으로 나타낼 수 있다.
```xml
<bean id="numberGuess" class="org.spring.samples.NumberGuess">
  <property name="randomNumber" value="#{ T(java.lang.Math).random() * 100.0 }"/>
</bean>
```

systemProperties bean을 미리 지정하여 사용할 수 있다. (bean을 사용하였지만 `@` 문자를 안붙인 것을 주의)
```xml
<bean id="taxCalculator" class="org.spring.samples.TaxCalculator">
  <property name="defaultLocale" value="#{ systemProperties['user.region'] }"/>
</bean>
```

아래 예제 처럼 다른 bean의 Properties를 사용할 수 있다.
```xml
<bean id="numberGuess" class="org.spring.samples.NumberGuess">
  <property name="randomNumber" value="#{ T(java.lang.Math).random() * 100.0 }"/>
</bean>

<bean id="shapeGuess" class="org.spring.samples.ShapeGuess">
  <property name="initialShapeSeed" value="#{ numberGuess.randomNumber }"/>
</bean>
```

## Annotation Based
* 기본값을 지정하기 위해 @Value Annotation을 이용해 Field, Method, Method-Parameter에 붙일 수 있다.

```java
public static class FieldValueTestBean

  @Value("#{ systemProperties['user.region'] }")
  private String defaultLocale;

  public void setDefaultLocale(String defaultLocale)
  {
    this.defaultLocale = defaultLocale;
  }

  public String getDefaultLocale() 
  {
    return this.defaultLocale;
  }
}
```

```java
public static class PropertyValueTestBean

  private String defaultLocale;

  @Value("#{ systemProperties['user.region'] }")
  public void setDefaultLocale(String defaultLocale)
  {
    this.defaultLocale = defaultLocale;
  }
}
```

```java
  private MovieFinder movieFinder;
  private String defaultLocale;

  @Autowired
  public void configure(MovieFinder movieFinder, 
                   @Value("#{ systemProperties['user.region'] }"} String defaultLocale) {
    this.movieFinder = movieFinder;
    this.defaultLocale = defaultLocale;
  }
```




# 참고

* https://docs.spring.io/spring/docs/4.3.10.RELEASE/spring-framework-reference/html/expressions.html
* https://blog.outsider.ne.kr/835
* https://blog.outsider.ne.kr/837