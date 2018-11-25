---
layout: posts
title: SpEL Expression(2)
catalog: true
Categories:
- Spring
tags:
- Spring
- SpEL
date: 2018-11-25 17:11:42
typora-root-url: 2018-11-25-spel-expression-2
typora-copy-images-to: 2018-11-25-spel-expression-2
---

# 리터럴 표현식 (Literal Expression)

리터럴 표현식을 제공하는 타입은 문자열(String), 숫자(int, real, hex), boolean, null이다.

* 문자열 (Strings)는 따옴표(')로 구분된다 `(쌍따옴표가 아님)` \- 문자열 표현 시\, `'Hello World'` 처럼 SpEL을 작성

다음은 리터럴 표현식 (Literal Expression)에 대한 간단한 예제이다.

``` java
ExpressionParser parser = new SpelExpressionParser();

//따옴표로 이루어진 'Hello World' => Hello World라는 문자열로 평가 된다.
String helloWorld = (String) parser.parseExpression("'Hello World'").getValue();
//Double 타입의 숫자로 평가
double avogadrosNumber = (Double) parser.parseExpression("6.0221415E+23").getValue();
//2147483647로 평가
int maxValue = (Integer) parser.parseExpression("0x7FFFFFFF").getValue();

boolean trueValue = (Boolean) parser.parseExpression("true").getValue();
//null로 평가 된다. -> null String이 아니다. (주의)
Object nullValue = parser.parseExpression("null").getValue();
```

* 숫자는 음수기호(-), 지수표시(E), 소수점(.)을 지원한다.
* 기본적으로 실제 숫자는 Double.parseDouble()로 파싱한다.

# 메서드 호출

``` java
//String 리터럴 abc에 대한 substring 메서드 호출 -> bc가 리턴된다.
String c = parser.parseExpression("'abc'.substring(2, 3)").getValue(String.class);

//사용자 정의 메서드 호출 (Return Type : Boolean) -> boolean 타입으로 평가된다.
boolean isMember = parser.parseExpression("isMember('Mihajlo Pupin')").getValue(
        societyContext, Boolean.class);
```

위의 코드는 메서드 호출에 대한 예제 코드이다.

* 메서드는 Java 문법을 사용하여 호출 할 수 있다.
* Literal에 대한 메서드 호출도 가능하다.
* Varargs 형식의 파라미터도 지원하고 있다.

# 프로퍼티, 배열, 리스트, 맵에 대한 접근 지원

``` java
ExpressionParser parser = new SpelExpressionParser();

// 발명품 배열
StandardEvaluationContext teslaContext = new StandardEvaluationContext(tesla);

// "Induction motor"로 평가된다.
String invention = parser.parseExpression("inventions[3]").getValue(teslaContext, String.class); 

// 회원 리스트
StandardEvaluationContext societyContext = new StandardEvaluationContext(ieee);

// "Nikola Tesla"로 평가된다.
String name = parser.parseExpression("Members[0].Name").getValue(societyContext, String.class);

// 리스트와 배열 탐색
// "Wireless communication"로 평가된다.
String invention = parser.parseExpression("Members[0].Inventions[6]").getValue(societyContext, String.class);
```

* 프로퍼티의 맨 첫글자는 대소문자를 구별하지 않는다.
* SpEL은 표준 `dot 표기법`(예: prop1.prop2.prop3)을 사용해서 중첩된 프로퍼티와 프로퍼티의 값 설정도 지원한다.
* public 필드에 대한 접근을 지원한다.

``` java
// Officer의 딕션어리
Inventor pupin = parser.parseExpression("Officers['president']").getValue(societyContext, Inventor.class);

// "Idvor"로 평가된다
String city = parser.parseExpression("Officers['president'].PlaceOfBirth.City").getValue(societyContext, String.class);

// 값을 설정한다
parser.parseExpression("Officers['advisors'][0].PlaceOfBirth.Country").setValue(societyContext, "Croatia");
```

* 대괄호`[]` 를 사용하여 Map의 key를 바탕으로 데이터에 접근한다.
  (위의 예제에서는 Officers가 `Map`, president가 `key`이다.)
* SpEL은 표준 'dot' 표기법(예: prop1.prop2.prop3)을 사용해서 Map내의 value 객체의 프로퍼티에도 접근 가능하다.
* setValue 메서드를 통해 데이터를 수정할 수 있다.

# 인라인 리스트 (Inline list)

리스트는 {} 표기법을 사용하여 리스트로 사용 할 수 있다.

``` java
// 4개의 숫자를 담고 있는 리스트로 평가된다
List<Integer> numbers = (List) parser.parseExpression("{1,2,3,4}").getValue(context); 

//List의 List의 형태의 List로 평가된다.
//listOfLists[0] = {'a', 'b'}
//listOfLists[1] = {'x', 'y'}
List listOfLists = (List) parser.parseExpression("{{'a','b'},{'x','y'}}").getValue(context);
```

# 배열 생성

``` java
int[] numbers1 = (int[]) parser.parseExpression("new int[4]").getValue(context); 

// initializer가진 배열
int[] numbers2 = (int[]) parser.parseExpression("new int[]{1,2,3}").getValue(context); 

// 다차원 배열
int[][] numbers3 = (int[][]) parser.parseExpression("new int[4][5]").getValue(context);
```

* Java 문법과 같은 표현식을 사용하여 배열을 생성 할 수 있다.
* {} 표현식을 통해 초기값을 세팅할 수 있다.
* 2차원 배열이상의 다차원 배열도 선언이 가능하다. (`단, 다차원 배열은 초기값을 설정할 수 없다.`)

# 연산자

## 관계 연산자

표준 관계 연산자를 사용한다. 사용 할 수 있는 관계 연산자는 아래와 같다.

* 같음 (`==`)
* 같지 않음 (`!=`)
* 작음 (`<`)
* 작거나 같음 (`<=`)
* 큼 (`>`)
* 크거나 같음 (`>=`)

``` java
// true로 평가된다
boolean trueValue = parser.parseExpression("2 == 2").getValue(Boolean.class);

// false로 평가된다
boolean falseValue = parser.parseExpression("2 < -5.0").getValue(Boolean.class);

// true로 평가된다
boolean trueValue = parser.parseExpression("'black' < 'block'").getValue(Boolean.class);
```

## 심볼릭 연산자

XML과 같이 문서형식에 관계 연산자가 제한을 받는 경우, 사용 가능한 문자 표현식을 제공한다.
연산자에 대한 대소문자는 구별하지 않는다.

* 같음 (`eq`)
* 같지 않음 (`ne`)
* 작음 (`lt`)
* 작거나 같음 (`le`)
* 큼 (`gt`)
* 크거나 같음 (`gt`)
* div (`/`)
* mod (`%`)
* not (`!`)

``` java
// true로 평가된다
boolean trueValue = parser.parseExpression("22 eq 22").getValue(Boolean.class);

// false로 평가된다
boolean falseValue = parser.parseExpression("'test' eq 'test!'").getValue(Boolean.class);
```

## 정규 표현식

matches를 이용한 정규표현식을 지원한다.

``` java
// true로 평가된다
boolean trueValue = parser.parseExpression("'5.00' matches '^-?\\d+(\\.\\d{2})?$'").getValue(Boolean.class);

// false로 평가된다
boolean falseValue = parser.parseExpression("'5.0067' matches '^-?\\d+(\\.\\d{2})?$'").getValue(Boolean.class);
```

## instanceof

Java에서 super타입에 대한 연산을 위해 instanceof를 표현식으로 사용할 수 있다.

``` java
// false로 평가된다
boolean falseValue = parser.parseExpression("'xyz' instanceof T(int)").getValue(Boolean.class);

// true로 평가된다.
boolean trueValue = parser.parseExpression("'xyz' instanceof T(String)").getValue(Boolean.class);
```

## 논리연산자

AND, OR, NOT에 대한 표현식을 지원한다.

``` java
// -- AND --

// false로 평가된다
boolean falseValue = parser.parseExpression("true and false").getValue(Boolean.class);

// true로 평가된다
String expression =  "isMember('Nikola Tesla') and isMember('Mihajlo Pupin')";
boolean trueValue = parser.parseExpression(expression).getValue(societyContext, Boolean.class);

// -- OR --

// true로 평가된다
boolean trueValue = parser.parseExpression("true or false").getValue(Boolean.class);

// true로 평가된다
String expression =  "isMember('Nikola Tesla') or isMember('Albert Einstien')";
boolean trueValue = parser.parseExpression(expression).getValue(societyContext, Boolean.class);

// -- NOT --

// false로 평가된다
boolean falseValue = parser.parseExpression("!true").getValue(Boolean.class);

// -- AND and NOT --
String expression =  "isMember('Nikola Tesla') and !isMember('Mihajlo Pupin')";
boolean falseValue = parser.parseExpression(expression).getValue(societyContext, Boolean.class);
```

## 수식 연산자

* 더하기 (+) 연산자를 숫자, 날짜, 문자열에 사용할 수 있다.
* 빼기(-) 연산자를 숫자, 날짜에 사용할 수 있다.
* 곱하기(*), 나누기(/), 나머지(%)에 대한 연산은 숫자에만 사용 할 수 있다.
* 연산자 우선 순위 법칙이 적용된다.

``` java
// 더하기
int two = parser.parseExpression("1 + 1").getValue(Integer.class); // 2
String testString = parser.parseExpression("'test' + ' ' + 'string'").getValue(String.class);  // 'test string'

// 빼기
int four =  parser.parseExpression("1 - -3").getValue(Integer.class); // 4
double d = parser.parseExpression("1000.00 - 1e4").getValue(Double.class); // -9000

// 곱하기
int six =  parser.parseExpression("-2 * -3").getValue(Integer.class); // 6
double twentyFour = parser.parseExpression("2.0 * 3e0 * 4").getValue(Double.class); // 24.0

// 나누기
int minusTwo =  parser.parseExpression("6 / -3").getValue(Integer.class); // -2
double one = parser.parseExpression("8.0 / 4e0 / 2").getValue(Double.class); // 1.0

// 계수(Modulus)
int three =  parser.parseExpression("7 % 4").getValue(Integer.class); // 3
int one = parser.parseExpression("8 / 5 % 2").getValue(Integer.class); // 1

// 연산자 우선순위
int minusTwentyOne = parser.parseExpression("1+2-3*8").getValue(Integer.class); // -21
```

## 3항 연산자 (If-Then-Else)

표현식에서 3항연산자를 통해 if-then-else 로직을 수행할 수 있다.
```java
//falseExp로 평가 된다.
String falseString = parser.parseExpression("false ? 'trueExp' : 'falseExp'").getValue(String.class);
```

## Elvis 연산자
* Elvis 연산자는 3항 연산자 구문을 단축시키고 Groovy 언어로 사용된다.
* 3항 연산자는 변수를 2번 반복해서 써야하는 단점이 있다.

```java
String name = "Elvis Presley";
String displayName = name != null ? name : "Unknown";
```

이런 조잡한 표현식 대신에 Elvis 연산자를 사용하면..
```java
ExpressionParser parser = new SpelExpressionParser();
String name = parser.parseExpression("name?:'Unknown'").getValue(String.class);  
System.out.println(name); // 'Unknown'
```

간단하게 변수에 대해 `?` 를 붙여주는 것만으로도 null 체크를 해준다.
? 모양이 엘비스 프레슬리의 머리모양을 닮아서 Elvis 연산자라고 명명하였다고 한다.



## 안전한탐색(Navigation) 연산자
* 안전한 탐색 연산자는 NPE를 피하기 위해 사용, Groovy 언어로 제공
* java로 코딩을 하다보면 습관적으로 if(variable == null) 과 같은 체크를 하게 된다.
* 하지만 Safe Navigation 연산자를 사용하면 NPE를 발생시키지 않고, 그냥 null을 리턴해 준다.

```java
ExpressionParser parser = new SpelExpressionParser();

Inventor tesla = new Inventor("Nikola Tesla", "Serbian");
tesla.setPlaceOfBirth(new PlaceOfBirth("Smiljan"));

StandardEvaluationContext context = new StandardEvaluationContext(tesla);

//placeOfBirth가 null인가? 체크하고 placeOfBirth.city에 대한 연산 수행
String city = parser.parseExpression("PlaceOfBirth?.City").getValue(context, String.class);
System.out.println(city); // Smiljan

tesla.setPlaceOfBirth(null);

 //placeOfBirth가 null인가? 체크하고 placeOfBirth.city에 대한 연산 수행
city = parser.parseExpression("PlaceOfBirth?.City").getValue(context, String.class);
System.out.println(city); // null - null pointer exception이 발생하지 않는다.
```



# 할당

* 할당연산자(`=`)를 이용하여 Context내의 프로퍼티에 value를 할당 할 수 있다.
* 보통은 setValue 메서드를 이용하여 value를 할당
* getValue 메서드를 이용하여 할당 할 수도 있다.

``` java
Inventor inventor = new Inventor();
StandardEvaluationContext inventorContext = new StandardEvaluationContext(inventor);
//inventor.name에 Alexander Seovic2 문자열을 할당
parser.parseExpression("Name").setValue(inventorContext, "Alexander Seovic2");

//할당연산자(`=`)를 이용하여 inventor.name에 Alexander Seovic2 문자열을 할당
String aleks = parser.parseExpression("Name = 'Alexandar Seovic'").getValue(inventorContext, String.class);
```

# 클래스 표현식

* `T` 연산자를 통해 클래스의 인스턴스를 지정하는데 사용할 수 있다.
* `T` 연산자를 통해 클래스의 static method도 사용할 수 있다.
* 웬만하면 full package를 적어준다.
* StandardEvaluationContext는 타입을 찾으려고 TypeLocator를 사용
* StandardTypeLocator는 java.lang 패키지로 만들어진다.
* 따라서 java.lang.String과 같은 타입은 T(String)으로 간단하게 표현 가능하다.

``` java
Class dateClass = parser.parseExpression("T(java.util.Date)").getValue(Class.class);

Class stringClass = parser.parseExpression("T(String)").getValue(Class.class);

boolean trueValue = parser.parseExpression("T(java.math.RoundingMode).CEILING < T(java.math.RoundingMode).FLOOR").getValue(Boolean.class);
```

# 생성자 호출

* 생성자는 새로운 연산자를 사용해서 호출할 수 있다.
* primitive type(원시 타입 int, double등..)과 String 외에는 모두 정규화된 클래스명을 사용해야 한다.

``` java
Inventor einstein = p.parseExpression("new org.spring.samples.spel.inventor.Inventor('Albert Einstein', 'German')").getValue(Inventor.class);

//리스트의 add 메서드내에서 새로운 inventor 인스턴스를 생성한다
p.parseExpression("Members.add(new org.spring.samples.spel.inventor.Inventor('Albert Einstein', 'German'))").getValue(societyContext);
```

# 변수

* `#` 표현식을 통해 표현식 내에 변수를 참조할 수 있다.
* StandardEvaluationContext에서 setVariable 메서드를 사용해서 변수를 설정한다.
* 자주 쓰이는 표현식이다. (@Cacheable Annotation에서도 사용)

``` java
Inventor tesla = new Inventor("Nikola Tesla", "Serbian");
StandardEvaluationContext context = new StandardEvaluationContext(tesla);
context.setVariable("newName", "Mike Tesla"); //newName변수에 대한 value할당

parser.parseExpression("Name = #newName").getValue(context);

System.out.println(tesla.getName()) // "Mike Tesla"
```

## #root

* 변수 #root는 항상 정의되며 루트 컨텍스트 개체의미
* #root는 항상 루트를 나타낸다.
* setRootObject 메서드를 통해 root를 정의한다.

``` java
StandardEvaluationContext context = new StandardEvaluationContext(tesla);

SomeCustomObject someObject = new SomeCustomObject();
context.setRootObject(someObject);

String name = "kocko";
context.setVariable("name", kocko);
String statement = "#root.stringLength(#kocko) == 5";
Expression expression = parser.parseExpression(statement);

boolean result = expression.getValue(context, Boolean.class);
```

* #root는 SomeCustomObject를 의미

## #this
```java
// create an array of integers
List<Integer> primes = new ArrayList<Integer>();
primes.addAll(Arrays.asList(2,3,5,7,11,13,17));

// 파서를 생성하고 'primes' 변수를 정수 배열로 설정한다
ExpressionParser parser = new SpelExpressionParser();
StandardEvaluationContext context = new StandardEvaluationContext();
context.setVariable("primes",primes);

// 리스트에서 10보다 큰 모든 소수(?{...} 선택을 사용)
// [11, 13, 17]로 평가된다
List<Integer>
 primesGreaterThanTen = (List<Integer>) 
parser.parseExpression("#primes.?[#this>10]").getValue(context);
//#primes 변수에 대해 평가하며, #this는 2~17까지 값을 의미한다.
```
* #this는 항상 정의되어 있고, 현재 평가되는 객체를 가리킨다.
* #this는 계속 변경될 수 있다. (위의 예제에서 보면 2,3,5,7등 평가 시점에 따라 달라질수 있다.)



# 사용자 정의 함수
* 표현식 문자열 내에서 호출 가능한 사용자 정의 함수를 등록하여 SpEL의 기능을 확장할 수 있다.
* StandardEvaluationContext에 사용자정의 함수를 등록하여 사용하면 된다.

예시로 문자열을 reverse 하는 메서드를 구현하였다.
```java
public abstract class StringUtils {

    public static String reverseString(String input) {
        StringBuilder backwards = new StringBuilder();
        for (int i = 0; i < input.length(); i++)
            backwards.append(input.charAt(input.length() - 1 - i));
        }
        return backwards.toString();
    }
}
```

메서드를 StandardEvaluationContext에 등록
```java
ExpressionParser parser = new SpelExpressionParser();
StandardEvaluationContext context = new StandardEvaluationContext();

//StringUtils 클래스에 정의된 reserveString 메서드를 등록한다.
//파라미터 타입은 String 타입 객체 1개이다.
context.registerFunction("reverseString", StringUtils.class.getDeclaredMethod("reverseString", new Class[] { String.class }));

//hello 문자열을 뒤집은 olleh가 리턴된다.
String helloWorldReversed = parser.parseExpression("#reverseString('hello')").getValue(context, String.class);
```

# Bean 참조

evaluation context에 Bean Resolver가 설정된 경우 `@` 표현식으로 bean을 사용할 수 있다.
Spring Docs에 있는 예제는 잘 이해가 안되어 직접 테스트 해보았다.


Bean 클래스 생성
```java
@Component
public class TestBean {

	public String test(){
		return "do Test";
	}
}
```

```java
@Controller
public class TestController {

	@Autowired
	private ApplicationContext applicationContext;

	@RequestMapping("/test")
	@ResponseBody
	public String test() {
		ExpressionParser parser = new SpelExpressionParser();
		StandardEvaluationContext context = new StandardEvaluationContext();

		//실용적인 테스트를 위해 Application Context의 BeanFactory를 설정
		DefaultListableBeanFactory factory = (DefaultListableBeanFactory) ((ConfigurableApplicationContext) applicationContext).getBeanFactory();
		context.setBeanResolver(new BeanFactoryResolver(factory));

		//Application Context에 등록된 testBean이라는 bean을 불러와 test메서드를 실행하였다.
		String result = parser.parseExpression("@testBean.test()").getValue(context, String.class);
		return result; //do Test가 리턴되었다.
	}
}
```


# Collection 선택 기능

* 컬렉션 (Collection) 선택 기능은 Context내의 Collection 필드에 접근하여 조건(Criteria)를 기반으로 서브 컬렉션을 반환 할 수 있다.
* `?[selectionExpression]` 표현식을 이용한다.
* 리스트, 맵에서 모두 사용 가능하다
* 객체가 context로 들어오는 경우 object.?[필드에 대한 조건]
* 컬렉션이 context로 들어오는 경우 .?[필드에 대한 조건] (별도의 필드가 없는 경우 #this를 사용)

Twice 클래스를 선언
```java
@NoArgsConstructor
public static class Twice {
    @Getter
    private List<Member> members = new ArrayList<Member>() {
        {
            add(new Member("지효", 23));
            add(new Member("나연", 24));
            add(new Member("모모", 23));
            add(new Member("사나", 23));
            add(new Member("다현", 21));
            add(new Member("미나", 23));
            add(new Member("채영", 19));
            add(new Member("쯔위", 19));
            add(new Member("정연", 23));
        }
    };
}

@NoArgsConstructor
@AllArgsConstructor
@Getter
public static class Member{
    private String name;
    private int age;
}
```

```java
public List<Twice> twice() {
    ExpressionParser parser = new SpelExpressionParser();
    StandardEvaluationContext context = new StandardEvaluationContext();

    // Twice의 멤버 필드에 접근
    // 컬렉션의 age 필드에 대해 20 이하인 대상을 반환
    List<Twice> filterList = (List<Twice>) parser.parseExpression("members.?[age < 20]").getValue(new Twice());
    return filterList; //[{"name":"채영","age":19},{"name":"쯔위","age":19}]
}
```

# Collection 투영 기능
* 투영기능은 컬렉션에 하위표현식을 반영하여 새로운 컬렉션을 리턴한다.
* `![projectionExpression]` 표현식을 사용한다.

위의 Twice 예제를 바탕으로 설명하겠다.
```java
public List<String> twice2() {
    ExpressionParser parser = new SpelExpressionParser();
    StandardEvaluationContext context = new StandardEvaluationContext();

    List<Member> testList = new ArrayList<Member>() {
        {
            add(new Member("지효", 23));
            add(new Member("나연", 24));
            add(new Member("모모", 23));
            add(new Member("사나", 23));
            add(new Member("다현", 21));
            add(new Member("미나", 23));
            add(new Member("채영", 19));
            add(new Member("쯔위", 19));
            add(new Member("정연", 23));
        }
    };
    // Twice 멤버 배열을 받아, 컬렉션 투영기능을 이용하여 그중 name에 대한 필드만 뽑아온 것이다.
    List<String> filterList = (List<String>) parser.parseExpression("![name]").getValue(testList);
    return filterList; //["지효","나연","모모","사나","다현","미나","채영","쯔위","정연"]
}
```



# 표현식 템플릿
* 표현식 템플릿을 사용하면 평가 블록과 Literal text를 혼합할 수 있다.
* 평가블록은 prefix와 subfix로 구분
* 일반적으로 `#{` 와 `}`로 구분한다.
* parseExpression() 메서드의 두번째 파라미터로 사용자 정의 ParseContext 객체를 넣어준다.

```java
public class TemplateParserContext implements ParserContext {

  public String getExpressionPrefix() {
    return "#{";
  }

  public String getExpressionSuffix() {
    return "}";
  }
  
  public boolean isTemplate() {
    return true;
  }
}
```

```java
// "random number is 0.7038186818312008"로 평가된다
String randomPhrase = 
   parser.parseExpression("random number is #{T(java.lang.Math).random()}", new TemplateParserContext()).getValue(String.class);

```