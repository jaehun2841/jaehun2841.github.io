---
layout: post
title:  "Spring AOP (1)"
subtitle: "1. AOP를 사용해야 하는 이유"
date:   2018-07-15 22:28:13 +0900
background: '/img/posts/06.jpg'
---



개발 하던 중에 Custom Annotation을 구현해서 사용해야 할 일이 생겼다. 
Custom Annotation을 구현하기 위해선 Spring AOP와 AspectJ의 기능을 사용하여 구현해야 하는데, 이번 기회에  다시 한번 Spring AOP에 대해 정리 해보려고 이 포스팅을 기록 하게 되었다.



# 들어가면서..



정리하고자 하는 내용

1. AOP를 사용해야하는 이유
2. AOP에서 사용되는 용어 정리
3. Spring AOP
4. AspectJ
5. Custom Annotation



내용이 많아 질 것 같아서 위의 순서대로 별도로 포스트를 작성할 예정이다.



# 1. AOP를 사용 해야 하는 이유

## AOP란?

 AOP(Aspect Oriented Programming) 란 우리말로 관점지향 프로그래밍이라고 한다.
객체지향의 5대원칙 중에는 단일책임의 원칙 (Single Responsibility Programming 이하 SRP) 라는 원칙이 있다.
하나의 클래스, 메서드는 하나의 역할만을 가진다는 원칙이다. 하지만 객체지향 프로그래밍(OOP)를 기반으로 로직을 설계하다 보면 단일책임의 원칙을 지키지 못하는 경우가 다반사로 발생한다.

예를 들어 결제를 담당하는 소스가 있다고 할 때

~~~java
public PaymentResponse doPaymentByCash(PaymentRequest paymentRequest) {
    
    //결제 요청에 대한 현금 결제처리
    PaymentResponse response = paymentModule.paymentCash(paymentRequest);
    
    //결제 요청에 대한 로그 데이터 DB Insert
    paymentLogRepository.insert(paymentRequest, response);
    
    return response;
}

public PaymentResponse doPaymentByCreditCard(PaymentRequest paymentRequest) {
    
    //결제 요청에 대한 카드 결제 처리
    PaymentResponse response = paymentModule.paymentCreditCard(paymentRequest);
    
    //결제 요청에 대한 로그 데이터 DB Insert
    paymentLogRepository.insert(paymentRequest, response);
    
    return response;
}
~~~



위의 doPaymentByXX라는 메서드는 실제 결제 요청 데이터에 대해 결제를 수행하는 메서드이다.
결제 처리 아랫쪽의 로그 데이터를 DB에 저장하는 부분은 doPaymentByXX라는 메서드 명에 어울리지 않는다.
굳이 변경하자면 doPaymentByXXWithLog 정도로 메서드명을 변경해야 의미가 맞겠지만, SRP의 원칙에 맞지 않는다고 생각한다.

여기서 로그 데이터를 삽입하는 로직은 결제 로직에 대해서는 횡단 로직으로 다른 관심사로 볼 수 있다.
뿐만 아니라 결제 수단 별로 나뉘어진 메서드에 대해 중복되는 로직으로 볼 수 있다.

이런 경우 우리는 결제 로그 삽입 이라는 다른 관심사로 분리하여 코드를 작성 할 수 있다.

그리고 이렇게 횡단 관심사로 분리되어진 공통 로직을 프로그램이 
컴파일 시에, 런타임 시에, 클래스가 로드되는 시점에 각 로직을 삽입 해주는 것을 AOP라는 개념으로 볼 수 있다.



AOP를 적용한다면 아래처럼 코드를 만들 수 있겠다.

~~~java
public PaymentResponse doPaymentByCash(PaymentRequest paymentRequest) {
    
    //결제 요청에 대한 현금 결제처리
    PaymentResponse response = paymentModule.paymentCash(paymentRequest);
    return response;
}

public PaymentResponse doPaymentByCreditCard(PaymentRequest paymentRequest) {
    
    //결제 요청에 대한 카드 결제 처리
    PaymentResponse response = paymentModule.paymentCreditCard(paymentRequest);
    return response;
}
~~~



~~~java
@Aspect
@Component
public class PaymentLogAdvisor {
    
    @Autowired
    private PaymentLogRepository paymentLogRepository;
    
    @AfterReturning("execution(* com.example.payment.doPayment*(..))", returning="paymentResponse")
    public void afterPayment(JoinPoint jp, PaymentResponse paymentResponse) {
        Object[] args = jp.getArgs();
        if(args.length == 0) {
            throw new illegalArgumentException("argument is none");
        }
        
        if(!(args[0] instanceof PaymentRequest)) {
            throw new Exception("argument is not PaymentRequest Type");
        }
        
        PaymentRequest paymentRequest = (PaymentRequest) args[0];
        
        //결제 로그 삽입
        paymentLogRepository.insert(paymentRequest, paymentResponse);
    }
}
~~~

