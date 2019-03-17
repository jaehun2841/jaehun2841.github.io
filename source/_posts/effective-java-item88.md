---
title: Item 88. readObject 메서드는 방어적으로 작성하라
catalog: true
Categories:
  - Effective-Java
tags:
  - Effective-Java
typora-root-url: effective-java-item88
typora-copy-images-to: effective-java-item88
date: 2019-03-17 17:11:59
subtitle:
header-img:
---

# 서론

Item 50에서는 불변인 날짜 범위 클래스를 만드는데 가변인 Date 필드를 이용했다.  
그래서 불변식을 지키고 불변을 유지하기 위해 생성자와 접근자에서 Date객체를 방어적으로 복사하느라 코드가 상당히 길어졌다.

```java
public final class Period {
    private final Date start;
    private final Date end;
    
    /**
     * @param start 시작 시각
     * @param end 종료 시각; 시작 시각보다 뒤여야 한다.
     * @throws IllegalArgumentException 시작 시각이 종료 시각보다 늦을 때 발생한다.
     * @throws NullPointerException start나 end가 null이면 발행한다.
     */
    public Period(Date start, Date end) {
        this.start = new Date(start.getTime());
        this.end = new Date(end.getTime());
        if(this.start.compareTo(this.end) > 0) {
            throw new IllegalArgumentException(start + "가 " + end + "보다 늦다.");
        }
    }
    
    public Date start() { return new Date(start.getTime()); }
    public Date end() { return new Date(end.getTime()); }
    public String toString() { return start + "-" + end; }
}
```

이 클래스는 물리적 표현과 논리적 표현이 부합하므로 기본 직렬화를 사용해도 좋다.  
하지만 이렇게 해서는 주요한 Date의 불변식을 보장하지 못한다.



# 필요하다면 매개변수를 방어적으로 복사하라

* readObject 메서드는 실질적으로는 또 다른 public 생성자이기 때문에 생성자와 똑같은 수준으로 주의를 기울여야한다.
* readObject 메서드에서 **인수가 유효한지 검사해야하고 필요하다면 방어적으로 복사하라**
* readObject에서 이 작업을 제대로 하지 못하면 공격자는 쉽게 클래스의 불변식을 깨뜨릴 수 있다.



## 불변식을 깨뜨릴 용도로 스트림을 조작하면 문제가 생긴다

```java
public class BogusPeriod {
    
    private static final byte[] serializedForm = {
        (byte)0xac, (byte)0xed, 0x00, 0x05, 0x73, 0x72, 0x00, 0x06....
    }
    
    public static void main(String[] args) {
        Period p = (Period) deserialize(serializedForm);
        System.out.println(p);
    }
    
    static Object deserialize(byte[] sf) {
        try {
            return new ObjectInputStream(new ByteArrayInputStream(sf)).readObject();
        } catch(IOException | ClassNotFoundException e) {
            throw new IllegalArgumentException(e);
        }
    }
}
```

* 이 코드의 serializedForm에서 상위 비트가 1인 바이트 값들은 byte로 형변환했는데,  
  이는 자바가 바이트 리터럴을 지원하지 않고 byte 타입은 부호가 있는 (signed) 타입이기 때문이다.
* 위의 프로그램을 실행하면   
  `Fri Jan 01 12:00:00 PST 1999 - Sun Jan 01 12:00:00 PST 1984`를 출력한다.
* Period를 직렬화할 수 있도록 선언한 것 만으로도 불변식을 깨뜨리는 객체를 만들 수 있다.



# 역직렬화 시, 불변식을 만족하는 유효성 검사를 해야한다.

```java
private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
    s.defaultReadObject();
    
    // 불변식을 만족하는지 검사한다.
    if(start.compareTo(end) > 0) {
       throw new InvalidObjectException(start + "가 " + end + "보다 늦다.");
    }
}
```

* 위의 if문 추가로 허용되지 않는 Period 인스턴스가 생성되는 일을 막을 수 있지만, 아직도 미묘한 문제가 숨어있다.
* 정상 Period 인스턴스에서 시작된 바이트 스트림 끝에 private Date 필드로의 참조를 추가하면 가변 Period 인스턴스를 만들 수 있다.



# 가변 공격을 막기위해서는 방어적 복사본을 만들어야 한다.

```java
public class MutablePeriod {
    //Period 인스턴스
    public final Period period;
    
    //시작 시각 필드 - 외부에서 접근할 수 없어야 한다.
    public final Date start;
    //종료 시각 필드 - 외부에서 접근할 수 없어야 한다.
    public final Date end;
    
    public MutablePeriod() {
        try {
            ByteArrayOutputStream bos = new ByteArrayOutputStream();
            ObjectArrayOutputStream out = new ObjectArrayOutputStream(bos);
            
            //유효한 Period 인스턴스를 직렬화한다.
            out.writeObject(new Period(new Date(), new Date()));
            
            /**
             * 악의적인 '이전 객체 참조', 즉 내부 Date 필드로의 참조를 추가한다.
             * 상세 내용은 자바 객체 직렬화 명세의 6.4절을 참고
             */
            byte[] ref = {0x71, 0, 0x7e, 0, 5}; // 참조 #5
            bos.write(ref); // 시작 start 필드 참조 추가
            ref[4] = 4; //참조 #4
            bos.write(ref); // 종료(end) 필드 참조 추가
            
            // Period 역직렬화 후 Date 참조를 훔친다.
            ObjectInputStream in = new ObjectInputStream(new ByteArrayInputStream(bos.toByteArray()));
            period = (Period) in.readObject();
            start = (Date) in.readObject();
            end = (Date) in.readObject();
        } catch (IOException | ClassNotFoundException e) {
            throw new AssertionError(e);
        }
    }
}
```

다음 공격 코드를 실행하면 이 공격이 실제로 이뤄지는 모습을 확인할 수 있다.

```java
public static void main(String[] args) {
    MutablePeriod mp = new MutablePeriod();
    Period p = mp.period;
    Date pEnd = mp.end;
    
    //시간 되돌리기
    pEnd.setYear(78);
    System.out.println(p); // Wed Nov 22 00:21:29 PST 2017 - Wed Nov 22 00:21:29 PST 1978
    
    //60년대로 회귀
    pEnd.setYear(60);
    System.out.println(p); // Wed Nov 22 00:21:29 PST 2017 - Wed Nov 22 00:21:29 PST 1969
}
```

이 예시에서 Period 인스턴스는 불변식을 유지한 채 생성됐지만 의도적으로 내부의 값을 수정할 수 있었다.  
이처럼 변경할 수 있는 Period 인스턴스를 획득한 공격자는 인스턴스가 불변이라고 가정하는 클래스에 넘겨 엄청난 보안 문제를 일으킬 수 있다.

이 문제의 근원은 **Period의 readObject메서드가 방어적 복사를 충분히 하지 않은 데 있다.**  
객체를 직렬화할 때는 클라이언트가 소유해서는 안 되는 객체 참조를 갖는 필드를 모두 방어적으로 복사해야 한다.  
따라서 `readObject에서는 불변 클래스 안의 모든 private 가변 요소를 방어적으로 복사 해야한다.`



# readObject 메서드에서는 private 가변요소를 방어 복사하라

```java
private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
    s.defaultReadObject();
    
    // 가변 요소들을 방어적으로 복사한다.
    start = new Date(start.getTime());
    end = new Date(end.getTime());
    
    // 불변식을 만족하는지 검사한다.
    if(start.compareTo(end) > 0) {
       throw new InvalidObjectException(start + "가 " + end + "보다 늦다.");
    }
}
```

* 방어적 복사를 유효성 검사보다 앞서 수행하며, Date의 clone 메서드는 사용하지 않았음에 주목하자.  
* 두 조치 모두 Period를 공격으로 부터 보호하는데 필요하다. 
* 또한 final 필드는 방어적 복사가 불가능 하니 주의하자 
* 그래서 이 readObject를 사용하려면 start와 end필드에서 final 한정자를 제거해야 한다.



# 기본 readObject를 사용해도 되는지에 대한 체크리스트

* transient 필드를 제외한 모든 필드의 값을 매개변수로 받아 유효성 검사 없이 필드에 대입하는 public 생성자를 추가해도 괜찮은가?
  * 아니오 -> 커스텀 readObject 메서드를 만들어 유효성 검사와 방어적 복사를 수행
  * 예 -> 기본 readObject 메서드 사용
* 직렬화 프록시 패턴을 사용해도 된다 .
  * 역직렬화를 안전하게 만드는 데 필요한 노력을 경감해 준다. (권장)



# 정리

* readObject 메서드를 작성할 때는 언제나 public 생성자를 만든다고 생각하고 만들어야 한다.
* private 이어야 하는 객체 참조 필드는 각 필드가 가리키는 객체를 방어적으로 복사하라 (불변 클래스 내의 가변 요소)
* 모든 불변식을 검사하여 어긋나는 게 발견되면 InvalidObjectException을 던진다.  
  방어적 복사 다음에는 반드시 불변식 검사가 뒤따라야 한다.
* 역직렬화 후 객체 그래프 전체의 유효성을 검사해야 한다면 ObjectInputValidation 인터페이스를 사용하라
* 직접적이든 간접적이든 readObject메서드에서 재정의 가능한 메서드를 호출해서는 안된다.
* 재정의 가능한 메서드가 재정의되면 하위 클래스의 상태가 완전히 역직렬화 되기전에 하위 클래스에서 재정의된 메서드가 실행되므로  
  프로그램 오작동을 일으킬 수 있다.



# 참고

* Effective Java 3rd Edition - Item 88. readObject 메서드는 방어적으로 작성하라