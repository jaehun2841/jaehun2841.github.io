---
title: Item 37. ordinal 인덱싱 대신 EnumMap을 사용하라
catalog: true
Categories:
  - Effective-Java
tags:
  - Effective-Java
typora-root-url: effective-java-item37
typora-copy-images-to: effective-java-item37
date: 2019-02-04 17:33:38
subtitle:
header-img:
---



# ordinal()을 배열 인덱스로 사용해선 안된다.

```java
class Plant {
    enum Lifecycle { ANNUAL, PERENNIAL, BIENNIAL }
    final String name;
    final Lifecycle lifeCycle;
    
    Plant(String name, Lifecycle lifeCycle) {
        this.name = name;
        this.lifeCycle = lifeCycle;
    } 
}
```

```java
Set<Plant>[] plantsByLifeCycle = (Set<Plant>[]) new Set[Plant.LifeCycle.values().length];
for (int i = 0 ; i < plantsByLifeCycle.length ; i++) {
    plantsByLifeCycle[i] = new HashSet<>();
}

for (Plant p : garden) {
    plantsByLifeCycle[p.lifeCycle.ordinal()].add(p); //이런 코드는 제발 쓰지 말자
}

// 결과 출력
for (int i = 0; i < plantsByLifeCycle.length; i++) {
    System.out.printf("%s: %s%n", Plant.LifeCycle.values()[i], plantsByLifeCycle[i]);
}
```



**문제점**

* `Set<Plant>[] plantsByLifeCycle = (Set<Plant>[]) new Set[Plant.LifeCycle.values().length]`  
  배열은 제네릭과 호환되지 않는다. (비검사 형변환이 수행, 컴파일이 안된다.)
* ordinal()은 상수 선언 순서에 따라 변한다.
* 잘못된 값을 사용하면 이상한 동작을 유발한다.



# EnumMap을 사용해 매핑

```java
Map<Plant.LifeCycle, Set<Plant>> plantsByLifeCycle = new EnumMap<>(Plant.LifeCycle.class);
for (Plant.LifeCycle lc : Plant.LifeCycle.values()) {
    plantsByLifeCycle.put(lc, new HashSet<>());
}
for (Plant p : garden) {
    plantsByLifeCycle.get(p.lifeCycle).add(p);
}
```

* 더 간단명료하게 로직이 변환되었다.
* 맵의 키인 열거 타입이 그 자체로 출력용 문자열을 제공하니 출력결과에 별도의 formatting이 필요없다.
* EnumMap의 성능이 ordinal을 쓴 배열과 같은 이유는 EnumMap 내부에서 ordinal을 사용한 배열을 사용하기 때문이다.
* 개발자가 직접 제어하지 않고 Map을 사용하여, 타입안정성을 얻을 뿐더러 성능상의 이점까지 그대로 가져간다.



# Stream을 이용한 코드

```java
//HashMap을 이용한 데이터와 열거타입 매핑
Arrays.stream(garden)
      .collect(groupingBy(p -> p.lifeCycle))
```
```java
//EnumMap을 이용해 데이터와 열거타입 매핑
Arrays.stream(garden)
      .collect(groupingBy(p -> p.lifeCycle, () -> new EnumMap<>(LifeCycle.class),
                         toSet()));
```

* 두 방식의 차이 
  * HashMap을 이용한 방식에는 garden에 있는 키만 만든다.
  * EnumMap을 이용한 방식에는 garden에 데이터가 없어도 모든 키가 다 만들어 진다. 



# 추가 예제

## ordinal()을 배열의 인덱스로 사용한 예

```java
public enum Phase {
    SOLID, LIQUID, GAS;

    public enum Transition {
        MELT, FREEZE, BOIL, CONDENSE, SUBLIME, DEPOSIT;
        
        // 행은 from의 ordinal을, 열은 to의 ordinal을 인덱스로 사용
        private static final Transition[][] TRANSITIONS = {
            { null, MELT, SUBLIME },
            { FREEZE, null, BOIL },
            { DEPOSIT, CONDENSE, null }
        };
        
        // 한 상태에서 다른 상태로의 전이를 반환한다.
        public static Transition from(Phase from, Phase to) {
            return TRANSITIONS[from.ordinal()][to.ordinal()];
        }
    }
}
```

 이 예제는 결국 SOLID, LIQUID, GAS의 상태 변화(from~to)에 대한 배열로 맵을 만든 것이다.  
이렇게 되면 Phase가 추가 될 때마다 배열을 수정해 줘야 하는 불상사가 발생한다.



## EnumMap을 이용

```java
public enum Phase {

    SOLID, LIQUID, GAS;

    public enum Transition {
        MELT(SOLID, LIQUID),
        FREEZE(LIQUID, SOLID),
        BOIL(LIQUID, GAS),
        CONDENSE(GAS, SOLID),
        SUBLIME(SOLID, GAS),
        DEPOSIT(GAS, SOLID);


        private final Phase from;
        private final Phase to;

        Transition(Phase from, Phase to) {
            this.from = from;
            this.to = to;
        }

        public static final Map<Phase, Map<Phase, Transition>> m = Stream.of(values())
                .collect(groupingBy(t -> t.from, () -> new EnumMap<>(Phase.class),
                        toMap(t -> t.to, //key-mapper
                                t -> t,  //value-mapper
                                (x, y) -> y, //merge-function
                                () -> new EnumMap<>(Phase.class))));

        public static Transition from (Phase from, Phase to) {
            return m.get(from).get(to);
        }
    }
```

EnumMap을 사용하여 간단하게 바꾼 모습이다.  
Map<Phase, Map<Phase, Transition>>을 초기화하는 부분이 복잡하다.

1. 일단 from으로 grouping 하여 EnumMap을 하나 생성
2. toMap으로 하위 Map을 생성
3. 첫 번째 인자는 Map의 key를 설정하는 Function이다. - Phase to로 선언
4. 두 번째 인자는 Map의 value를 설정하는 Function이다. - 자기 자신을 참조
5. 세 번째 인자는 merge-function이다. - 얘는 별 의미없다.
6. 네 번째 인자는 EnumMap으로 내부 Map을 선언한다.

따라서 from 메서드에서 Phase별 from~to에 대해 Map -> Map에 접근하여 Transition을 리턴 할 수 있다.



## 새로운 Phase가 추가되는 경우

```JAVA
public enum Phase {

    SOLID, LIQUID, GAS, PLASMA;

    public enum Transition {
        MELT(SOLID, LIQUID),
        FREEZE(LIQUID, SOLID),
        BOIL(LIQUID, GAS),
        CONDENSE(GAS, SOLID),
        SUBLIME(SOLID, GAS),
        DEPOSIT(GAS, SOLID),
        IONIZE(GAS, PLASMA),
        DEIONIZE(PLASMA, GAS);

        private final Phase from;
        private final Phase to;
```

PLASMA라는 Phase가 추가되어도 Transition에 IONIZE, DEIONIZE를 간단히 추가하여 유연하게 대응이 가능하다.



# 요약

* 배열의 인덱스를 얻기 위해 ordinal을 쓰는 것은 일반적으로 좋지 않으니, 대신 EnumMap을 사용하라.

  


# 참고
* Effective Java 3rd Edition - Item 37. ordinal 인덱싱 대신 EnumMap을 사용하라
