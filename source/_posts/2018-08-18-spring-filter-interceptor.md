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

# Prolog

개발 업무를 하다보면 갖가지 인증 처리, 예외 처리등을 위해 Filter나 Interceptor를 사용해야 하는 부분이 많다. 특히 필자의 경우에는 어뷰징 방지등을 위한 코드로 Interceptor를 많이 사용하였다. 회사 코드 중에서는 어뷰징 방지 코드가 Service Layer에서 이루어지는 경우가 많은데, 비교하는 코드에서 중복코드가 몇 군데 있었고, 다른 코드에서 중복코드를 양산 할 수 있다는 생각이 들어 이번 기회에 Interceptor로 리팩토링을 하였다.
Filter의 경우는 거의 사용해 본 적이 없는데, 자주 쓰는 Interceptor와 비교하여 그 내용을 이번 기회에 정리하고자 한다.



# Spring Request Flow

![spring-request-lifecycle](/spring-request-lifecycle.jpg)



이 이미지가 Spring MVC 구조를 가장 잘 보여주는 도식도 인 것 같아 첨부하였다.
(많은 블로그에서 이 이미지를 사용하고 있는점은 안 비밀이다.)



그림에서 보면 가장먼저 눈에 띄는 것은 Filter와 Interceptor의 실행 위치이다.
Filter는 Dispatcher Servlet 이전에 실행된다. 정확히는 WAS내의 ApplicationContext에서 등록된 필터가 실행 된다. Filter는 J2EE표준 스펙이며, Servlet 2.3에 등장하였다. 따라서 Spring Framework가 아니어도 Servlet Filter를 사용할 수 있다. 

Interceptor의 실행 위치는 DispatchetServlet 내부에서 실행이 되고 있다.
따라서 Interceptor는 Spring Framework에서 제공하는 API이며, 다양한 기능을 제공하고 있다.
공통적으로 Spring에서는 전후처리기로 많이 사용하고 있다. AOP와 함께 핵심 로직에 영향을 주지 않고 요청을 가로채어 처리하는 Spring의 큰 특징 중 하나로 생각된다.



# Filter

위에서 잠깐 설명 했듯이, (Servlet) Filter는 J2EE 표준 스펙으로 Servlet API 2.3 부터 등장 하였다.
실행 위치는 WAS(Web Application Server) 내의 Application Context에 등록 된 필터가 요청 URL Pattern에 따라 실행 되도록 되어있다. 대표적인 예로는 Spring Security, CORS Filter등이 있다.



### Filter Chain

![filter-chain](/filter-chain.gif)

Filter의 큰 특징으로는 Filter chain이라는 개념이 있다.
실제 Filter는 web.xml에 등록 된 Filter들을 WAS 구동 시에, WAS내의 ApplicationContext내의 Standard Context에 FilterMap이라는 Array에 등록된다. 실행 시에 요청 URL의 형식에 맞는 Pattern을 가진 Filter들로 Filter chain을 구성하게 되어 순차적으로 실행 되게 된다. 

* [Request에 대한 Filter Chain 생성](https://github.com/apache/tomcat/blob/trunk/java/org/apache/catalina/core/ApplicationDispatcher.java#L705)
* [Request URL에 대한 FilterMap 생성](https://github.com/apache/tomcat/blob/trunk/java/org/apache/catalina/core/ApplicationFilterFactory.java#L53)

※ 본 포스팅에서는 WAS에 대한 기준을 Apache Tomcat 8.5를 기준으로 작성하였다.



Filter Chain이 실행되는 순서는 2가지 원칙에 의해 결정된다.

1. url-pattern 매칭은 web.xml 파일에 표기된 순서대로 필터 체인을 형성한다.
2. servlet-name 매칭이 web.xml 파일에 표기된 순서대로 필터 체인을 형성한다.



### 필터 생성

~~~java
package com.example.springstudy.filter;

import javax.servlet.*;
import javax.servlet.annotation.WebFilter;
import java.io.IOException;

@WebFilter
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



### 설정 방법

Filter를 등록하는 방식을 크게 3가지 정도 있다.

1. web.xml 등록 방식
2. Java config 등록 방식 -> FilterRegistration Bean을 정의하여 추가할 Filter를 정의
3. java config 등록 방식 -> AbstractAnnotationConfigDispatcherServletInitializer에서 getServletFilter에 추가



##### 1. web.xml 등록 방식

```xml
<filter>
    <filter-name>testFilter</filter-name>
    <filter-class>com.example.springstudy.filter.TestFilter</filter-class>
</filter>
<filter-mapping>
    <filter-name>testFilter</filter-name>
    <url-pattern>/*</url-pattern>
    <!-- url-pattern 대신 Servlet을 지정할 수도 있다. -->
    <servlet-name>testServlet</servlet-name> 
</filter-mapping>
```



##### 2.  FilterRegistration Bean을 정의하여 추가할 Filter를 정의

~~~java
package com.example.springstudy.config;

import com.example.springstudy.filter.TestFilter;
import org.springframework.boot.web.servlet.FilterRegistrationBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class WebApplicationFilterConfig {

    @Bean
    public FilterRegistrationBean getFilterRegistration() {
        FilterRegistrationBean filterRegistrationBean = new FilterRegistrationBean();
        filterRegistrationBean.setFilter(new TestFilter());
        filterRegistrationBean.addUrlPatterns("/*");
        filterRegistrationBean.setName("Test-Filter");
        filterRegistrationBean.setOrder(1);

        return filterRegistrationBean;
    }
}
~~~



##### 3.  AbstractAnnotationConfigDispatcherServletInitializer에서 getServletFilter에 추가

~~~java
package com.example.springstudy.config;

import org.springframework.web.servlet.support.AbstractAnnotationConfigDispatcherServletInitializer;

import javax.servlet.Filter;

@Configuration
public class WebInitializerConfig extends AbstractAnnotationConfigDispatcherServletInitializer {

    @Override
    protected Filter[] getServletFilters() {
        //추가할 필터 리스트를 추가한다.
        return new Filter[]{new TestFilter()};
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



# Interceptor

### 설정 방법



동작원리는 URLMatcher를 통해 설정한 url-pattern과 매칭되는 인터셉터만

골라서 인터셉터 체인 형식으로 실행







# 참조

http://javacan.tistory.com/entry/58
http://www.leafcats.com/39
https://stackoverflow.com/questions/6560969/how-to-define-servlet-filter-order-of-execution-using-annotations-in-war
https://supawer0728.github.io/2018/04/04/spring-filter-interceptor/