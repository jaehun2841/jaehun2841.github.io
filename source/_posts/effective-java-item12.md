---
title: Item 12. toString을 항상 재정의하라
catalog: true
Categories:
  - Effective-Java
tags:
  - Effective-Java
typora-root-url: effective-java-item12
typora-copy-images-to: effective-java-item12
date: 2019-01-13 14:59:25
subtitle:
header-img:
---

# 서론
Object의 기본 toString 메서드가 우리가 작성한 클래스에 적합한 문자열을 반환하는 경우는 거의 없다.  
이 메서드는 `PhoneNumber@adbbd`처럼 단순히 **클래스이름@16진수로_표현한_해시코드**를 반환할 뿐이다.  
toString의 일반 규약에 따르면, `간결하면서 사람이 읽기 쉬운 형태의 유익한 정보`를 반환해야 한다.  
toString의 규악은 `모든 하위클래스에서 이 메서드를 재정의하라`라고 하고 있다.

# toString을 재정의 해야하는 이유
* toString을 잘 구현한 클래스는 사용하기 편하고, 그 클래스에 대해 디버깅 하기 쉽다.
  * map객체를 출력하는 경우 `{Jenny=PhoneNumber@addbb}` 보다는 `{Jenney=707-867-5308}`이라는 메세지가 가독서이 좋다.
* 실전에서는 toString메서드를 재작성 할 때, 그 객체가 가진 주요 정보 모두를 반환하는게 좋다.
* toString을 구현할 때면 반환값의 포맷을 문서화 할 지 정해야 한다.
  * 포맷을 명시하면, 그 객체는 표준적이고, 명확하고, 사람이 읽을 수 있게 된다.
  * 포맷을 명시하기로 했으면, 명시한 포맷에 맞는 문자열과 객체를 상호전환 할 수 있는 정적 팩터리나 생성자를 함께 제공하면 좋다.
  * 단, 포맷을 한번 명시하면, 평생 그 포맷에 얽매이게 된다.
  * 포맷을 명시하지 않는다면 다음 릴리즈에 포맷을 변경할 수 있는 유연성을 더 가져갈 수 있다.
* 포맷을 명시하든 아니든, 개발자의 의도는 명확히 밝혀야 한다.
* toString이 반환한 값에 대해 포함된 정보를 얻어올 수 있는 API를 제공하자
  * toString에 있는 getter를 제공하지 않는다면, 클라이언트에서 toString을 파싱하여 사용할 지도 모른다.

## 포맷을 명시하기로 했으면, 명시한 포맷에 맞는 문자열과 객체를 상호전환 할 수 있는 정적 팩터리나 생성자를 함께 제공하면 좋다.
```java

@RunWith(JUnit4.class)
public class ToStringTest {

    @Test
    public void toString테스트() {
        String phoneNumber = "707-908-9999";
        assertEquals(PhoneNumber.parse(phoneNumber), new PhoneNumber(707, 908, 9999));
    }

    @Test(expected = UnknownFormatConversionException.class)
    public void 파싱문자열_오류_테스트() {
        String phoneNumber = "707-908";
        assertEquals(PhoneNumber.parse(phoneNumber), new PhoneNumber(707, 908, null));
    }


    @NoArgsConstructor
    @AllArgsConstructor
    @Builder
    public static class PhoneNumber{
        private Integer areaCode;
        private Integer prefix;
        private Integer lineNum;

        private static final Pattern phoneNumberPattern = Pattern.compile("^\\d{3}-\\d{3}-\\d{4}$");

        @Override
        public boolean equals(Object o) {
            if(!(o instanceof PhoneNumber)) {
                return false;
            }

            PhoneNumber pn = (PhoneNumber) o;
            return this.areaCode.equals(pn.areaCode)
                    && this.prefix.equals(pn.prefix)
                    && this.lineNum.equals(pn.lineNum);
        }

        @Override
        public int hashCode() {
            int result = Integer.hashCode(areaCode);
            result = 31 * result + Integer.hashCode(prefix);
            result = 31 * result + Integer.hashCode(lineNum);
            return result;
        }

        @Override
        public String toString() {
            return String.format("%03d-%03d-%04d", areaCode, prefix, lineNum);
        }

        public static PhoneNumber parse(String phoneNumber) {

            if(!phoneNumberPattern.matcher(phoneNumber).find()) {
                throw new UnknownFormatConversionException(phoneNumber + " cannot be parsed");
            }

            String[] numbers = phoneNumber.split("-");
            return PhoneNumber.builder()
                    .areaCode(Integer.parseInt(numbers[0]))
                    .prefix(Integer.parseInt(numbers[1]))
                    .lineNum(Integer.parseInt(numbers[2]))
                    .build();
        }
    }
```

# toString을 따로 재정의 안해도 되는 경우
* 정적 Utils 클래스는 따로 재정의 하지 않아도 된다.
(객체의 상태(state)를 가지는 클래스가 아니기 떄문)
* enum 타입 또한 이미 완벽한 toString을 제공한다.
* 대다수의 컬렉션 구현체는 추상 컬렉션 클래스(AbstractMap, AbstractSet등)의 toString 메서드를 상속하여 쓴다.
* 라이브러리를 통해 자동생성하자
  * 구글의 @Autovalue
  * Lombok의 @ToString
  * 위의 라이브러리들을 이용해 자동생성하는 편이 더 간편하다.

# 참고
* Effective Java 3rd Edition - Item 12. toString을 항상 재정의하라