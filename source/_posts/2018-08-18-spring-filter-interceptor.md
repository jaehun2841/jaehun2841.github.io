---
title: Spring Filter와 Interceptor
catalog: true
date: 2018-08-25 18:30:26
subtitle:
header-img:
Categories:
 - Spring
tags: 
 - Spring
 - Spring Core
typora-root-url: ./2018-08-18-spring-filter-interceptor
typora-copy-images-to: ./2018-08-18-spring-filter-interceptor
---

# 들어가며..

개발 업무를 하다보면 갖가지 인증 처리, 예외 처리등을 위해 Filter나 Interceptor를 사용해야 하는 부분이 많다. 특히 필자의 경우에는 어뷰징 방지등을 위한 코드로 Interceptor를 많이 사용하였다. 
회사 코드 중에서는 어뷰징 방지 코드가 Service Layer에서 이루어지는 경우가 많은데, 비교하는 코드에서 중복코드가 몇 군데 있었고, 다른 코드에서 중복코드를 양산 할 수 있다는 생각이 들어 이번 기회에 Interceptor로 리팩토링을 하였다.
Filter의 경우는 거의 사용해 본 적이 없는데, 자주 쓰는 Interceptor와 비교하여 그 내용을 이번 기회에 정리하고자 한다.



# Spring Request Flow

![spring-request-lifecycle](./spring-request-lifecycle.jpg)



이 이미지가 Spring MVC 구조를 가장 잘 보여주는 도식도 인 것 같아 첨부하였다.
(많은 블로그에서 이 이미지를 사용하고 있는점은 안 비밀이다.)



그림에서 보면 가장먼저 눈에 띄는 것은 Filter와 Interceptor의 실행 위치이다.
Filter는 Dispatcher Servlet 이전에 실행된다. 정확히는 WAS내의 ApplicationContext에서 등록된 필터가 실행 된다. Filter는 `J2EE표준스펙`이며, Servlet 2.3에 등장하였다. 따라서 Spring Framework가 아니어도 Servlet Filter를 사용할 수 있다. 

Interceptor의 실행 위치는 DispatchetServlet 내부에서 실행이 되고 있다.
따라서 Interceptor는 `Spring Framework에서 제공하는 API`이며, 전후처리에 대한 편리한 인터페이스를 제공하고 있다. 공통적으로 Spring에서는 전후처리기로 많이 사용하고 있다. AOP와 함께 핵심 로직에 영향을 주지 않고 요청을 가로채어 처리하는 Spring의 큰 특징 중 하나로 생각된다.



# Filter

위에서 잠깐 설명 했듯이, (Servlet) Filter는 J2EE 표준 스펙으로 Servlet API 2.3 부터 등장 하였다.
실행 위치는 WAS(Web Application Server) 내의 Application Context에 등록 된 필터가 요청 URL Pattern에 따라 실행 되도록 되어있다. 대표적인 예로는 `Spring Security, CORS Filter`등이 있다.



## Filter Chain

![filter-chain](./filter-chain.gif)

Filter의 큰 특징으로는 Filter chain이라는 개념이 있다.
실제 Filter는 web.xml에 등록 된 Filter들을 WAS 구동 시에, WAS내의 ApplicationContext내의 `Standard Context`에 `FilterMap`이라는 Array에 등록된다. 실행 시에 요청 URL의 형식에 맞는 Pattern을 가진 Filter들로 `Filter chain`을 구성하게 되어 순차적으로 실행 되게 된다. 

* [Request에 대한 Filter Chain 생성](https://github.com/apache/tomcat/blob/trunk/java/org/apache/catalina/core/ApplicationDispatcher.java#L705)
* [Request URL에 대한 FilterMap 생성](https://github.com/apache/tomcat/blob/trunk/java/org/apache/catalina/core/ApplicationFilterFactory.java#L53)

※ 본 포스팅에서는 WAS에 대한 기준을 Apache Tomcat 8.5를 기준으로 작성하였다.



Filter Chain이 실행되는 순서는 2가지 원칙에 의해 결정된다.

1. url-pattern 매칭은 web.xml 파일에 표기된 순서대로 필터 체인을 형성한다.
2. servlet-name 매칭이 web.xml 파일에 표기된 순서대로 필터 체인을 형성한다.



## Filter 생성

~~~java
package com.example.springstudy.filter;

import javax.servlet.*;
import javax.servlet.annotation.WebFilter;
import java.io.IOException;

public class TestFilter implements Filter {
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
		//filter 생성 시 처리
    }

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        //다음 Filter 실행 전 처리 (preHandle)
        
        //다음 filter-chain에 대한 실행 (filter-chain의 마지막에는 Dispatcher servlet실행)
		filterChain.doFilter(servletRequest, servletResponse);
        
        //다음 Filter 실행 후 처리 (postHandle)
    }

    @Override
    public void destroy() {
		//filter 제거 시 처리 (보통 자원의 해제처리를 한다.)
    }
}

~~~



## 설정 방법

Filter를 등록하는 방식을 크게 4가지 정도 있다.

1. web.xml 등록 방식
2. Java config 등록 방식 -> FilterRegistration Bean을 정의하여 추가할 Filter를 정의
3. java config 등록 방식 -> AbstractAnnotationConfigDispatcherServletInitializer에서 getServletFilter에 추가
4. @WebFilter Annotation 등록 방식



### 1. web.xml 등록 방식

```xml
<filter>
    <filter-name>testFilter</filter-name>
    <filter-class>com.example.springstudy.filter.TestFilter</filter-class>
</filter>
<filter>
    <filter-name>testFilter2</filter-name>
    <filter-class>com.example.springstudy.filter.TestFilter</filter-class>
</filter>

<filter-mapping>
    <filter-name>testFilter</filter-name>
    <url-pattern>/*</url-pattern>
    <!-- url-pattern 대신 Servlet을 지정할 수도 있다. -->
    <servlet-name>testServlet</servlet-name> 
</filter-mapping>
<filter-mapping>
    <filter-name>testFilter2</filter-name>
    <url-pattern>/*</url-pattern>
    <!-- url-pattern 대신 Servlet을 지정할 수도 있다. -->
    <servlet-name>testServlet</servlet-name> 
</filter-mapping>
```



### 2.  FilterRegistration Bean을 정의하여 추가할 Filter를 정의

~~~java
package com.example.springstudy.config;

import com.example.springstudy.filter.TestFilter;
import org.springframework.boot.web.servlet.FilterRegistrationBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class WebApplicationFilterConfig {

    @Bean
    public FilterRegistrationBean testFilterRegistration() {
        FilterRegistrationBean filterRegistrationBean = new FilterRegistrationBean();
        filterRegistrationBean.setFilter(new TestFilter());
        filterRegistrationBean.addUrlPatterns("/*");
        filterRegistrationBean.setName("Test-Filter");
        filterRegistrationBean.setOrder(1);

        return filterRegistrationBean;
    }
    
    @Bean
    public FilterRegistrationBean testFilter2Registration() {
        FilterRegistrationBean filterRegistrationBean = new FilterRegistrationBean();
        filterRegistrationBean.setFilter(new TestFilter2());
        filterRegistrationBean.addUrlPatterns("/*");
        filterRegistrationBean.setName("Test-Filter2");
        filterRegistrationBean.setOrder(2);

        return filterRegistrationBean;
    }
}
~~~



Filter chain의 실행 순서는 Filter1(Prehandle) -> Filter2(Prehandle) -> Filter2 (Posthandle) -> Filter1 (Posthandle) 순으로 실행 된다. 



### 3.  AbstractAnnotationConfigDispatcherServletInitializer에서 getServletFilter에 추가

※ 이 방식은 Spring Boot 환경에서 동작하지 않았다. (Embedded Tomcat이어서 그런가? 잘 모르겠다.)

~~~java
package com.example.springstudy.config;

import org.springframework.web.servlet.support.AbstractAnnotationConfigDispatcherServletInitializer;

import javax.servlet.Filter;

@Configuration
public class WebInitializerConfig extends AbstractAnnotationConfigDispatcherServletInitializer {

    @Override
    protected Filter[] getServletFilters() {
        //추가할 필터 리스트를 추가한다.
        return new Filter[]{new TestFilter(), new TestFilter2()};
    }

    @Override
    protected Class<?>[] getRootConfigClasses() {
        return new Class[0];
    }

    @Override
    protected Class<?>[] getServletConfigClasses() {
        return new Class[0];
    }

    @Override
    protected String[] getServletMappings() {
        return new String[0];
    }
}

~~~



### 4. @Component, @WebFilter, @Order 어노테이션을 이용한 필터 등록

~~~java
package com.example.springstudy.filter;

import org.springframework.core.annotation.Order;
import org.springframework.stereotype.Component;

import javax.servlet.*;
import javax.servlet.annotation.WebFilter;
import java.io.IOException;

@Component
@WebFilter(
        description = "1번째 필터",
        urlPatterns = "/*",
        filterName = "Test-Filter1"
)
@Order(2)
public class TestFilter implements Filter {
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {

    }

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        System.out.println("start testFilter1");
        filterChain.doFilter(servletRequest, servletResponse);
        System.out.println("finish testFilter1");
    }

    @Override
    public void destroy() {

    }
}
~~~

다른 설정 파일 없이 Filter Class파일 하나만 가지고 Filter에 대한 설정을 할 수 있다.
@Component -> Component-scan 시 Spring Bean으로 등록 된다.
@WebFIlter -> Filter등록에 필요한 Interface를 제공한다.
@Order -> @Component 어노테이션 사용 시 `Order Interface` 사용이 가능하다. Filter chain에 대한 순서를 지정 할 수 있다. 
개인적으로 가장 깔끔한 방법이라 생각하지만, Ordering 시 헷갈릴 수 있을 것 같다는 생각이 들었다.



# Interceptor

인터셉터는 주로 세션에 대한 체크, 인증 처리 등에 사용한다. 위에서 말했듯이 필자는 대부분의 어뷰징에 대한 처리를 인터셉터 단에서 처리하고 했다. 또한 특정 상품 진입 시, 권한 체크, 제약 조건 체크들도 인터셉터 단에서 처리 한 경험이 있다.

인터셉터도 필터와 비슷한 방식으로 작동한다. 하지만 필터와는 다르게 `preHandle(), postHandle()` 메소드가 구분 되어져 있어, 분기가 명확하다.
또한 Filter와는 다르게 `handlerMethod`를 파라미터로 제공하여 AOP 비스무리한 효과를 낼 수 있다. handler에서는 실행하고자 하는 컨트롤러에 대한 method 시그니처를 제공하여 좀 더 기능을 확장할 수 있다는 장점이 있다.



## Interceptor 동작 방식

1. 외부로 부터 요청이 들어오면 DispatcherServlet에서 요청을 처리한다.
2. DispatcherServlet의 doDispatch() 메소드에서 [getHandler()](https://github.com/spring-projects/spring-framework/blob/master/spring-webmvc/src/main/java/org/springframework/web/servlet/DispatcherServlet.java#L1013) 메소드로 HandlerExecutionChain를 호출 한다.
   (정확히는 RequestMappingHandlerAdapter의 HandlerExecutionChain)
3.  [getHandler()](https://github.com/spring-projects/spring-framework/blob/master/spring-webmvc/src/main/java/org/springframework/web/servlet/DispatcherServlet.java#L1227) 메소드 내부에는 [getHandlerInternal()](https://github.com/spring-projects/spring-framework/blob/master/spring-webmvc/src/main/java/org/springframework/web/servlet/handler/AbstractHandlerMapping.java#L401) 메소드로 handler를 가져오는 부분이 있다.
   이 부분이 바로 요청 URL과 매칭하는 Controller 메소드를 찾아내는 부분이다.
4. 그 다음 [getHandlerExecutionChain()](https://github.com/spring-projects/spring-framework/blob/master/spring-webmvc/src/main/java/org/springframework/web/servlet/handler/AbstractHandlerMapping.java#L480) 메소드에서 요청 메소드의 URL에 대해 이미 등록 된 interceptor 들의 url-pattern들과 매칭 되는 interceptor 리스트를 추출한다.
5. 추출 된 interceptor들에 대해 [preHandle()](https://github.com/spring-projects/spring-framework/blob/master/spring-webmvc/src/main/java/org/springframework/web/servlet/DispatcherServlet.java#L1032) 메소드를 실행 시킨다.
   (preHandle() 메소드의 리턴 타입은 boolean인데 false가 리턴 되는 경우에는 Controller 메소드를 실행 하지 않는다.)
6. 그 다음 요청 URL에 맞는 Controller 메소드를 실행 시킨다.
7. 메소드 작업이 끝난 뒤 추출 된 interceptor들에 대해 [postHandle()](https://github.com/spring-projects/spring-framework/blob/master/spring-webmvc/src/main/java/org/springframework/web/servlet/DispatcherServlet.java#L1044) 메소드를 실행 시킨다.



## 설정 방법

설정 방법은 크게 2가지로 이루어져 있다.

1. servlet-context.xml에 등록

2. Java-config 방식을 이용한 등록 -> WebMvcConfigurationSupport 이용하여 등록


###  1. Servlet-context.xml에 등록

~~~xml
<mvc:interceptors>
        <mvc:interceptor>
            <mvc:mapping path="/client"/>
            <mvc:mapping path="/client/test1"/>
            <bean id="testInterceptor"
                  class="com.example.springstudy.interceptor.TestInterceptor"/>
        </mvc:interceptor>
</mvc:interceptors>
~~~



### 2. WebMvcConfigurationSupport 이용하여 등록

~~~java
package com.example.springstudy.config;

import com.example.springstudy.interceptor.TestInterceptor;
import com.example.springstudy.interceptor.TestInterceptor2;
import com.example.springstudy.resolver.ClientIpArgumentResolver;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.method.support.HandlerMethodArgumentResolver;
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurationSupport;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurerAdapter;

import java.util.List;

@Configuration
public class WebMvcConfig extends WebMvcConfigurationSupport {

    /**
    * addInterceptors 메소드를 통해 Interceptor 등록
    */
    @Override
    protected void addInterceptors(InterceptorRegistry registry) {
        super.addInterceptors(registry);
        //String... 타입으로 여러개 지정 가능
        registry.addInterceptor(new TestInterceptor())
                .addPathPatterns("/client/test1", "/client/help");
        //List 타입으로 여러개 지정 가능
        registry.addInterceptor(new TestInterceptor2())
                .addPathPatterns(Lists.newArrayList("/client", "/client/test1"));
    }
}

~~~



# 참조

http://javacan.tistory.com/entry/58
http://www.leafcats.com/39
https://stackoverflow.com/questions/6560969/how-to-define-servlet-filter-order-of-execution-using-annotations-in-war
https://supawer0728.github.io/2018/04/04/spring-filter-interceptor/