---
title: Spring에서 Client IP구하기
catalog: true
date: 2018-08-10 22:24:23
subtitle:
header-img:
tags: Spring
typora-root-url: ./2018-08-10-httprequest-client-ip
typora-copy-images-to: ./2018-08-10-httprequest-client-ip
---



# HttpServletRequest에서 IP 구하기

회사 업무 중에 Controller에서 Request를 보낸 IP에 대한 정보를 저장해야 하는 요청 사항이 있었다. 처음에는 Javascript를 통해 Client IP를 조회한 다음에 파라미터로 보내야 하나? 하고 생각 했지만, 적용해야 하는 부분도 많았고, IP를 조회 하는 중복 코드가 다수 발생 할 것이라 생각이 들었다. 

생각을 해보니, HttpServletRequest내의 Header정보 중에는 분명 IP정보도 잊겠지? 라는 생각으로 찾아보았고, 아니나 다를까 IP정보를 조회할 수 있는 방법이 있었다.

HttpServletRequest에서 IP를 구하는 소스는 아래와 같다.

이전 포스트에서 잠깐 다뤘던 내용을 그대로 퍼왔다 ([Spring Argument Resolver](https://jaehun2841.github.io/2018/08/10/2018-08-10-spring-argument-resolver/))

~~~java
    /**
     * Method parameter에 대한 Argument Resovler로직 처리
     * @param methodParameter
     * @param modelAndViewContainer
     * @param nativeWebRequest
     * @param webDataBinderFactory
     * @return
     * @throws Exception
     */
    @Override
    public Object resolveArgument(MethodParameter methodParameter, ModelAndViewContainer modelAndViewContainer,NativeWebRequest nativeWebRequest, WebDataBinderFactory webDataBinderFactory) throws Exception {

        HttpServletRequest request = (HttpServletRequest) nativeWebRequest.getNativeRequest();

        String clientIp = request.getHeader("X-Forwarded-For");
        if (StringUtils.isEmpty(clientIp)|| "unknown".equalsIgnoreCase(clientIp)) {
            //Proxy 서버인 경우
            clientIp = request.getHeader("Proxy-Client-IP");
        }
        if (StringUtils.isEmpty(clientIp) || "unknown".equalsIgnoreCase(clientIp)) {
            //Weblogic 서버인 경우
            clientIp = request.getHeader("WL-Proxy-Client-IP");
        }
        if (StringUtils.isEmpty(clientIp) || "unknown".equalsIgnoreCase(clientIp)) {
            clientIp = request.getHeader("HTTP_CLIENT_IP");
        }
        if (StringUtils.isEmpty(clientIp) || "unknown".equalsIgnoreCase(clientIp)) {
            clientIp = request.getHeader("HTTP_X_FORWARDED_FOR");
        }
        if (StringUtils.isEmpty(clientIp) || "unknown".equalsIgnoreCase(clientIp)) {
            clientIp = request.getRemoteAddr();
        }

        return clientIp;
    }
~~~



보통은 HttpServletRequest.getRemoteAddr()를 하면은 서비스를 요청한 Client의 IP정보가 나온다. 하지만 서버 구성이 L4 스위치, 프록시 서버, 로드밸런서등이 구성 되어 있으면 다른 Header를 통해 ip를 접근해야 하는 경우도 생긴다.
구글링을 해보니 위의 코드 정도면은 예상 되는 Case를 Filtering 하여 Client IP를 구할 수 있다고 한다.





# IPv6 형식으로 나오는 IP를 IPv4로 변환

위의 코드를 적용하여 Client IP가 제대로 나오는지 테스트를 해보았다.

![ipv6](./ipv6.png)

 

예상과는 다르게 IPv6 형태의 IP주소가 나오는 것을 볼 수 있었다. 


 IPv6로 나오는 IP를 IPv4형식으로 나오게 하기 위해서는 WAS의 VM옵션을 추가해야 한다.

1. Tomcat인 경우
   1. $CATALINA_HOME\bin\catalina.bat(.sh) 을 찾는다.
   2. `:noJuliConfig, :noJuliManager` 를 검색한다. 
   3. JAVA_OPTS에 `-Djava.net.preferIPv4Stack=true` 를 추가한다.

~~~shell
if not "%LOGGING_CONFIG%" == "" goto noJuliConfig
set LOGGING_CONFIG=-Dnop
if not exist "%CATALINA_BASE%\conf\logging.properties" goto noJuliConfig
set LOGGING_CONFIG=-Djava.util.logging.config.file="%CATALINA_BASE%\conf\logging.properties"
:noJuliConfig
set JAVA_OPTS=%JAVA_OPTS% %LOGGING_CONFIG% -Djava.net.preferIPv4Stack=true

if not "%LOGGING_MANAGER%" == "" goto noJuliManager
set LOGGING_MANAGER=-Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager
:noJuliManager
set JAVA_OPTS=%JAVA_OPTS% %LOGGING_MANAGER% -Djava.net.preferIPv4Stack=true
~~~



2. 개발 환경에서 VM 속성 추가 하기
   1. 사용 중인 IDE에서 VM Options에 `-Djava.net.preferIPv4Stack=true` 를 추가한다.

![ide setting](./ide setting.png)



확인 결과

![ipv4](./ipv4.png)



127.0.0.1로 정상적인 IP가 나오는 것을 확인 하였다.



# 참고

http://all-record.tistory.com/168
http://www.leafcats.com/35
http://ooz.co.kr/138