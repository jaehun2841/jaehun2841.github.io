---
title: Spring Cloud Config Monitor 시작하기
catalog: true
Categories:
  - Spring
tags:
  - Spring
typora-root-url: 2022-03-11-spring-cloud-config-monitor
typora-copy-images-to: 2022-03-11-spring-cloud-config-monitor
date: 2022-03-11 03:07:28
subtitle:
header-img:
---

# 들어가며

해당 글에서 사용한 예제의 버전 정보는 아래와 같습니다

- kotlin 1.6.0
- Spring Boot 2.6.3
- Spring Cloud Dependencies 2021.0.1

Spring Cloud Config 관련 글은 총 4개의 글로 작성되었습니다.

- [Spring Cloud Config 시작하기](../2022-03-11-spring-cloud-config)
- [Spring Cloud Bus 시작하기](../2022-03-11-spring-cloud-bus)
- **Spring Cloud Config Monitor 시작하기**
- [Spring Cloud Config 주기적으로 Polling 하기](../2022-03-11-spring-cloud-config-polling)

# Spring Cloud Config Monitor

![](./spring-cloud-monitor.jpg)

- `spring-cloud-config-monitor` 를 dependency에 추가하고 spring cloud bus를 활성화 하면 
`/monitor` endpoint가 활성화 됩니다.
- /monitor 엔드포인트를 통해 갱신된 client에 대한 이벤트를 전파할 수 있습니다.
- 특정 클라이언트에만 이벤트 갱신 요청을 보낼 수 있습니다.

# 설정하기

## 1. build.gradle

```json
dependencies {
    implementation("org.springframework.cloud:spring-cloud-config-server")
    implementation("org.springframework.boot:spring-boot-starter-jdbc")
    implementation("org.springframework.cloud:spring-cloud-config-monitor")
	  implementation("org.springframework.cloud:spring-cloud-starter-bus-kafka")
}
```

- config-server 애플리케이션에 spring-cloud-config-monitor 의존성을 추가합니다.

## 2. application.yml

```yaml
spring:
  cloud:
    bus:
      destination: jaehun-platform.prod.event.config-event.json
    config:
      # 추가된 부분
      monitor:
        endpoint:
          path: /api/v1/remote-configurations 
      enabled: true
      server:
        jdbc:
          sql: select prop_key, prop_value from remote_configurations where application=? and profile=? and label=?
  datasource:
    url: jdbc:mysql://localhost:3306/config-server
    username: <database-username>
    password: <database-password>
    driver-class-name: org.mariadb.jdbc.Driver
  profiles:
    active:
      - jdbc
      - local
  # 추가된 부분
  kafka:
    bootstrap-servers: http://localhost:9092

```

- spring cloud config monitor의 기본 endpoint는 /monitor 입니다.
    - 하지만 다른 엔드포인트와 헷갈리지 않도록 `spring.cloud.config.monitor.endpoint.path`를 추가하여 prefix를 선언하여 사용합니다.
    - 위의 설정대로 하면 monitor 엔드포인트는 `/api/v1/remote-configurations/monitor` 가 됩니다

## monitor 엔드포인트는 어떻게 생겼을까?

![](./monitor-endpoint.png)

- PropertyPathEndpoint라는 클래스에서 endpoint를 제공합니다.
- notifyByPath라는 메서드에서 /monitor에 대한 요청을 처리합니다.

## monitor endpoint를 요청해봅시다.

```bash
curl --location --request POST 'http://config-server/api/v1/remote-configurations/monitor' \
--header 'Content-Type: application/json' \
--data-raw '{}'
```

- 단순하게 config server에 POST 메서드로 `/api/v1/remote-configurations/monitor` 엔드포인트를 요청했습니다.

하지만 kafka topic에 아무것도 퍼블리싱된 이벤트가 없습니다 왜그럴까요?

## notifyByPath 코드에 집중해 봅시다.

![](./notifyByPath.png)

<span style="color:green;background:black"><b>초록색</b></span> → <span style="color:yellow;background:black"><b>노란색</b></span> → <span style="color:red;background:black"><b>빨간색</b></span> 박스 순으로 보겠습니다.

- 초록색 박스를 보면 우리가 생각하는 RefreshRemoteApplicationEvent를 발행합니다.
    - 이 이벤트가 각 클라이언트에 전파되어 context refresh를 트리거 해줍니다.
    - 하지만 이벤트를 발행하려면 service라는 String 값이 필요한 것 같습니다.
- 노란색 박스를 보면 notification이라는 객체에서 path를 추출하여 service를 생성하고 있습니다.
- 빨간색 박스를 보면 extrator라는 객체를 이용하여 header와 request body 내용을 토대로 notification을 생성합니다.

자 그럼 extractor는 뭘까요?

# PropertyPathNotificationExtractor를 주목해 봅시다.

![](./PropertyPathNotificationExtractor.png)

- 주석에는 request와 headers를 통해 notifiation을 제공한다고 되어 있네요

![](./PropertyPathNotificationExtractor2.png)

- notifyByPath에 디버그를 걸어보면 extrator는 CompositePropertyPathNotification이라는 타입의 객체 입니다.
- 클래스명에서 유추할 수 있듯이 PropertyPathNotificationExtractor에 대한 구현체가 List 형태로 있습니다.
- 보아하니 github, gitlab, gitee, bit bucket등에 대한 다양한 webhook 요청에 대해 처리하는 구현체로 보입니다.

우리는 직접 endpoint를 호출하려고 하기 때문에 2번째 객체인 SimplePropertyPathNotificationExtractor를 통해 path를 지정해 보겠습니다.

## SimplePropertyPathNotificationExtractor 살펴보기

![](./SimplePropertyPathNotificationExtractor.png)

- SimplePropertyPathNotificationExtractor는 request body에 있는 path라는 프로퍼티를 읽습니다.
- path에 대한 타입이 String 인 경우에는 단일 값만 `PropertyPathNotification` 으로 리턴합니다.
- path에 대한 타입이 Collection인 경우에는 Collection을 담아 `PropertyPathNotification` 으로 리턴합니다.

## 다시 요청하기

```bash
curl --location --request POST 'http://config-server/api/v1/remote-configurations/monitor' \
--header 'Content-Type: application/json' \
--data-raw '{
    "path" : [
        "**"
    ]
}'
```

path를 `**` 으로 요청하여 모든 클라이언트에게 이벤트를 전파 시켜 보겠습니다.

```bash
[
    {
        "type": "RefreshRemoteApplicationEvent",
        "timestamp": 1646660036164,
        "originService": "jaehun-microservice-admin:0:config-server",
        "destinationService": "**",
        "id": "11fd441b-1840-44c0-be7a-f63acc0a55e5"
    },
    {
        "type": "AckRemoteApplicationEvent",
        "timestamp": 1646660037787,
        "originService": "jaehun-microservice-admin:0:config-server",
        "destinationService": "**",
        "id": "6d11a58d-ed98-4725-b388-9854239f65fe",
        "ackId": "11fd441b-1840-44c0-be7a-f63acc0a55e5",
        "ackDestinationService": "**",
        "event": "org.springframework.cloud.bus.event.RefreshRemoteApplicationEvent"
    },
    {
        "type": "AckRemoteApplicationEvent",
        "timestamp": 1646660038649,
        "originService": "jaehun-microservice-router:8082:client1",
        "destinationService": "**",
        "id": "861376bf-c221-4222-9f96-7c03bd546cb7",
        "ackId": "11fd441b-1840-44c0-be7a-f63acc0a55e5",
        "ackDestinationService": "**",
        "event": "org.springframework.cloud.bus.event.RefreshRemoteApplicationEvent"
    },
    {
        "type": "AckRemoteApplicationEvent",
        "timestamp": 1646660038652,
        "originService": "jaehun-microservice-router:8083:client2",
        "destinationService": "**",
        "id": "e416ee3e-bfc3-4caa-9135-dd6736f10e87",
        "ackId": "11fd441b-1840-44c0-be7a-f63acc0a55e5",
        "ackDestinationService": "**",
        "event": "org.springframework.cloud.bus.event.RefreshRemoteApplicationEvent"
    }
]
```

- config-server가 /monitor 엔드포인트를 통해 RefreshRemoteApplicationEvent를 kafka에 퍼블리싱 했습니다.
- 위에서 부터 순서대로 config-server, client1, client2에서 이벤트를 수신했다고 ack를 퍼블리싱 합니다.
- 확인 하니 모든 클라이언트에서 context refresh가 발생함을 확인했습니다.

# 특정 클라이언트에만 이벤트 발행하기

하나의 설정이 변경되었을 때 모든 클라이언트에 대해 context refresh가 발생한다면 상당히 비효율적 입니다.
변경이 발생한 프로퍼티의 대상 클라이언트에만 context refresh가 발생하도록 해보겠습니다.

만약 변경이 발생한 프로퍼티의 application이 jaehun-microservice-router 인 경우 아래와 같이 path 부분에 파라미터를 설정합니다.

```bash
curl --location --request POST 'http://config-server/api/v1/remote-configurations/monitor' \
--header 'Content-Type: application/json' \
--data-raw '{
    "path" : [
        "jaehun-microservice-router:**"
    ]
}'
```

```json

  {
      "type": "RefreshRemoteApplicationEvent",
      "timestamp": 1646661367607,
      "originService": "jaehun-microservice-admin:0:config-server",
      "destinationService": "jaehun:microservice-router:**",
      "id": "9517ae59-00d6-4911-945d-0ca34f84c2e9"
  }
  {
      "type": "RefreshRemoteApplicationEvent",
      "timestamp": 1646661367611,
      "originService": "jaehun-microservice-admin:0:config-server",
      "destinationService": "jaehun-microservice:router:**",
      "id": "e5db510b-9b49-4aa5-b94e-935ba4658b40"
  }
  {
      "type": "RefreshRemoteApplicationEvent",
      "timestamp": 1646661367612,
      "originService": "jaehun-microservice-admin:0:config-server",
      "destinationService": "jaehun-platform-router:**",
      "id": "14c184ec-5700-491e-88b0-8f6282657473"
  }
  {
      "type": "AckRemoteApplicationEvent",
      "timestamp": 1646661367828,
      "originService": "jaehun-microservice-router:8082:client1",
      "destinationService": "**",
      "id": "846110db-fd9c-4c70-8b11-9a4f89a018f4",
      "ackId": "14c184ec-5700-491e-88b0-8f6282657473",
      "ackDestinationService": "jaehun-microservice-router:**",
      "event": "org.springframework.cloud.bus.event.RefreshRemoteApplicationEvent"
  }
  {
      "type": "AckRemoteApplicationEvent",
      "timestamp": 1646661367848,
      "originService": "jaehun-microservice-router:8083:client2",
      "destinationService": "**",
      "id": "e942b045-a126-4c57-b7e8-54bf327d307f",
      "ackId": "14c184ec-5700-491e-88b0-8f6282657473",
      "ackDestinationService": "jaehun-microservice-router:**",
      "event": "org.springframework.cloud.bus.event.RefreshRemoteApplicationEvent"
  }

```

- RefreshRemoteApplicationEvent가 3번 발생하였습니다.
    - jaehun:microservice-router:**
    - jaehun-microservice:router:**
    - jaehun-microservice-router:**
- 이유는 위의 노란색 박스 코드에서 guessServiceName이라는 함수 실행 시 대시(-)를 구분자로 하여 모든 케이스에 대한 서비스를 추출하기 때문입니다.
    - 따라서 가급적 application name에 대시를 넣지 않는게 좋습니다.

ack 이벤트를 보겠습니다.

- ack 응답을 준 애플리케이션은 2개 입니다.
    - originServicer가 `jaehun-microservice-router` 로 시작하는 애플리케이션 2개 입니다.
- 따라서 전체 클라이언트가 아닌 jaehun-microservice-router 애플리케이션에만 context refresh가 일어나는 것을 확인할 수 있습니다.

# 참고

- [https://docs.spring.io/spring-cloud-config/docs/3.0.3/reference/html/](https://docs.spring.io/spring-cloud-config/docs/3.0.3/reference/html/)