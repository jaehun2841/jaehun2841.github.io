---
title: EHCache 설정방법 (Spring Boot)
catalog: true
Categories:
  - Spring
tags:
  - Spring
  - Spring Boot
  - EHCache
typora-root-url: 2018-11-04-ehcache-config-for-springboot
typora-copy-images-to: 2018-11-04-ehcache-config-for-springboot
date: 2018-11-07 23:38:43
header-img:
---

# EHCache 설정하기

설정 순서는 아래와 같다.

1. Maven Dependency 설정
2. Ehcache.xml 작성 (ehcache 설정파일)
3. @EnableCaching 설정



# Maven Dependency 설정

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
</dependency>
<dependency>
    <groupId>net.sf.ehcache</groupId>
    <artifactId>ehcache</artifactId>
    <version>2.10.6</version>
</dependency>
```

# Ehcache.xml 작성 (ehcache 설정파일)
~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<ehcache xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:noNamespaceSchemaLocation="http://ehcache.org/ehcache.xsd"
         maxBytesLocalHeap="300M" <!-- CacheManager에 의해 관리되는 캐시의 메모리를 300M로 제한 -->
         updateCheck="false">

    <!-- 임시저장 경로를 설정 -->
    <diskStore path="java.io.tmpdir" />
    <!-- 
        Cache에 저장할 레퍼런스의 최대값을 100000으로 지정,
        maxDepthExceededBehavior = "continue" :  초과 된 최대 깊이에 대해 경고하지만 크기가 조정 된 요소를 계속 탐색
        maxDepthExceededBehavior = "abort" : 순회를 중지하고 부분적으로 계산 된 크기를 즉시 반환
    -->
    <sizeOfPolicy maxDepth="100000" maxDepthExceededBehavior="continue"/>

   <!-- default Cache 설정 (반드시 선언해야 하는 Cache) -->
    <defaultCache
            eternal="false"
            timeToIdleSeconds="0"
            timeToLiveSeconds="1200"
            overflowToDisk="false"
            diskPersistent="false"
            diskExpiryThreadIntervalSeconds="120"
            memoryStoreEvictionPolicy="LRU">
    </defaultCache>

    <!-- 사용하고자 하는 캐시 별 설정 -->
    <cache name="LocalCacheData"
           eternal="false"
           timeToIdleSeconds="0"
           timeToLiveSeconds="1200"
           overflowToDisk="false"
           diskPersistent="false"
           diskExpiryThreadIntervalSeconds="120"
           memoryStoreEvictionPolicy="LRU">
    </cache>

   <cache name="AuthMemberList"
           eternal="false"
           timeToIdleSeconds="0"
           timeToLiveSeconds="1200"
           overflowToDisk="false"
           diskPersistent="false"
           diskExpiryThreadIntervalSeconds="120"
           memoryStoreEvictionPolicy="LRU">
    </cache>
</ehcache>
~~~

* defaultCache는 반드시 구현해야 할 캐시 (직접 생성하는 캐시에 대한 기본 설정)
* cache는 하나의 캐시를 사용할 때마다 구현
* name 속성은 캐시의 이름을 지정하며, 코드에서는 이 캐시의 이름을 사용하여 사용할 Cache 인스턴스를 구한다.

| 속성                            | 설명 | Default |
| ------------------------------- | ---- | ------------------------------- |
| name                            | 코드에서 사용할 캐시 name | 필수 |
|  maxEntriesLocalHeap                                   |메모리에 생성 될 최대 캐시 갯수| 0 |
|  maxEntriesLocalDisk                                  |디스크에 생성 될 최대 캐시 갯수| 0 |
| eternal                         | 영속성 캐시 설정 (지워지는 캐시인지?) <br />external = "true"이면, timeToIdleSecond, timeToLiveSeconds 설정이 무시됨 | false |
| timeToIdleSecond                | 해당 초동안 캐시가 호출 되지 않으면 삭제 | 0 |
| timeToLiveSeconds               | 해당 초가 지나면 캐시가 삭제 | 0 |
| overflowToDisk                  | 오버플로우 된 항목에 대해 disk에 저장할 지 여부 | false |
| diskPersistent                  | 캐시를 disk에 저장하여, 서버 로드 시 캐시를 말아 둘지 설정 | false |
| diskExpiryThreadIntervalSeconds | Disk Expiry 스레드의 작업 수행 간격 설정 | 0 |
| memoryStoreEvictionPolicy | 캐시의 객체 수가 maxEntriesLocalHeap에 도달하면, 객체를 추가하고 제거하는 정책 설정<br />LRU : 가장 오랫동안 호출 되지 않은 캐시를 삭제<br />LFU : 호출 빈도가 가장 적은 캐시를 삭제<br />FIFO : First In First Out, 캐시가 생성된 순서대로 가장 오래된 캐시를 삭제 | LRU |

 


#@EnableCaching 설정 

~~~java
@Configuration
@EnableCaching(proxyTargetClass = true, mode = AdviceMode.PROXY)
public class EHCacheConfig {
    @Bean
    public EhCacheManagerFactoryBean ehCacheManagerFactoryBean() {
        EhCacheManagerFactoryBean ehCacheManagerFactoryBean = new EhCacheManagerFactoryBean();
        ehCacheManagerFactoryBean.setConfigLocation(new ClassPathResource("config/ehcache.xml"));
        ehCacheManagerFactoryBean.setShared(true);
        return ehCacheManagerFactoryBean;
    }

    @Bean
    public EhCacheCacheManager ehCacheCacheManager(EhCacheManagerFactoryBean ehCacheManagerFactoryBean) {
        EhCacheCacheManager ehCacheCacheManager = new EhCacheCacheManager();
        ehCacheCacheManager.setCacheManager(ehCacheManagerFactoryBean.getObject());
        return ehCacheCacheManager;
    }
}
~~~

* @EnableCaching Annotation은 `<cache:annotation-driven>`와 마찬가지로 어노테이션 기반 캐시를 사용 할 수 있는 설정
* 내부적으로 Spring AOP를 통해 @Cacheable, @CacheEvict와 같은 어노테이션을 Pointcut으로 이용해서 Cache기능을 구현하는 듯 하다.
* proxyTargetClass : class 기반 프록시를 생성함을 의미 (CGLIB라이브러리에 대한 의존성 필요)
* Mode : 어떤 Advisor 모듈을 선택할지에 대한 설정 
  * PROXY : Spring AOP Proxy방식을 이용
  * ASPECTJ : LTW, CTW기능을 이용한 위빙