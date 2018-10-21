---
layout: posts
title: "@Import와 @ImportResource Annotation"
date: 2018-10-21 18:53:38
subtitle: xml을 Java Config로 변경하기
catalog: true
Categories:
  - Spring
tags:
  - Spring
  - Java-Config
typora-root-url: ./2018-10-21-java-config-import
typora-copy-images-to: ./2018-10-21-java-config-import
---



# 들어가며

회사 프로젝트 설정은 대부분이 xml설정으로 되어있다.
아무도 그 xml설정을 건드리지 않고 있는데, 내가 차츰차츰 Java-Config로 바꾸자고 주장하고 있었고 
최근에 일부 xml Config를 Java Config로 전환하는 작업을 진행 하였다.
그 때 사용한 @Import, @ImportResource Annotation에 대한 기록을 남겨보고자 한다.



# @Import

@Import Annotation은 기본적으로 Java 파일에 대한 Import를 위해 사용한다.
사용하는 개념은 기존에 xml 파일을 import하는.. <import /> 구분과 동일하게 사용된다.

예를 들어 Cache에 대한 Datasource를 설정하는 부분이 있다고 가정을 하면..



**RedisClusterConfig**

~~~java
@Configuration
@Import(value = RedisShardsConfig.class) // Redis Shard정보에 대한 Config이다.
public class RedisClusterConfig {

	@Autowired
	@Qualifier("redisConfig")
	private Properties redisConfig;

	@Bean
	public GenericObjectPoolConfig jedisPoolConfig() {
		JedisPoolConfig poolConfig = new JedisPoolConfig();
	    poolConfig.setMaxTotal(redisConfig.getProperty("redis.cluster.connectionCount"));
		poolConfig.setMaxIdle(commonConfig.getProperty("redis.cluster.connectionCount"));
		poolConfig.setMinIdle(commonConfig.getProperty("redis.cluster.connectionCount"));
		poolConfig.setNumTestsPerEvictionRun(2);
		poolConfig.setTestOnBorrow(true);
		poolConfig.setTestOnReturn(false);
		poolConfig.setTestWhileIdle(true);
		poolConfig.setTimeBetweenEvictionRunsMillis(300000);
		return poolConfig;
	}
    
    //따로 @Autowired 하지 않고, 변수명 = bean이름으로 자동으로 매칭될 수 있게 설정하였다.
	@Bean
	public ShardedJedisPool masterShardedPool(GenericObjectPoolConfig jedisPoolConfig, List<JedisShardInfo> redisMasterShards) {
		return new ShardedJedisPool(jedisPoolConfig, redisMasterShards, Hashing.MURMUR_HASH);
	}

	@Bean
	public ShardedJedisPool slaveShardedPool(GenericObjectPoolConfig jedisPoolConfig, List<JedisShardInfo> redisSlaveShards) {
		return new ShardedJedisPool(jedisPoolConfig, redisSlaveShards, Hashing.MURMUR_HASH);
	}
}
~~~



**RedisShardsConfig.java**

~~~java
@Configuration
public class RedisShardsConfig {

    @Autowired
	@Qualifier("redisConfig")
	private Properties redisConfig;
    
	@Bean
	public List<JedisShardInfo> redisMasterShards() {
        
        String url = redisConfig.getProperty("redis.cluster.url");
        //Integer 형변환 생략함...(길어져서 안보임)
        Integer masterPort1 = redisConfig.getProperty("redis.cluster.master.port1");
        Integer masterPort2 = redisConfig.getProperty("redis.cluster.master.port2");
        Integer masterPort3 = redisConfig.getProperty("redis.cluster.master.port3");
        
        String masterShardKey1 = redisConfig.getProperty("redis.cluster.master.key1");
        String masterShardKey2 = redisConfig.getProperty("redis.cluster.master.key2");
        String masterShardKey3 = redisConfig.getProperty("redis.cluster.master.key3");
        
		return new ArrayList<JedisShardInfo>() {
			{
				add(new JedisShardInfo(url, masterPort1, masterShardKey1));
				add(new JedisShardInfo(url, masterPort2, masterShardKey2));
				add(new JedisShardInfo(url, masterPort3, masterShardKey3));
			}
		};
	}

	@Bean
	public List<JedisShardInfo> redisSlaveShards() {
        
        String url = redisConfig.getProperty("redis.cluster.url");
        //Integer 형변환 생략함...(길어져서 안보임)
        Integer slavePort1 = redisConfig.getProperty("redis.cluster.slave.port1");
        Integer slavePort2 = redisConfig.getProperty("redis.cluster.slave.port2");
        Integer slavePort3 = redisConfig.getProperty("redis.cluster.slave.port3");
        
        String slaveShardKey1 = redisConfig.getProperty("redis.cluster.slave.key1");
        String slaveShardKey2 = redisConfig.getProperty("redis.cluster.slave.key2");
        String slaveShardKey3 = redisConfig.getProperty("redis.cluster.slave.key3");
        
		return new ArrayList<JedisShardInfo>() {
			{
				add(new JedisShardInfo(url, slavePort1, slaveShardKey1));
				add(new JedisShardInfo(url, slavePort2, slaveShardKey2));
				add(new JedisShardInfo(url, slavePort3, slaveShardKey3));
			}
		};
	}
}
~~~



RedisClusterConfig.java에서는 실제 Redis Cluster Bean을 생성하는 코드이다.
RedisShardsConfig.java에서는 Shard Node에 대한 설정을 하는 부분이다. 
이렇게 설정을 두개로 분리하여 @Import를 Annotation을 사용 하여 상속보다는 유동적으로 Config 설정을 할 수 있다.
이 설정은 기본 Spring Configuration 설정 보다는 Test Case작성 시에 큰 힘을 발휘 할 것이라고 생각된다.
xml을 로드해서 전체 Bean을 올리는게 아닌, 필요한 Bean 설정만 Import해서 올릴 수 있기 때문이다.



# @ImportResource

모든 설정을 JavaConfig로 변경하면 좋겠지만, 시간 관계상이나 복잡한것들은 나중에 하려고 xml파일 형태로 남겨둔 것들이 있다. (예를들면 Datasource나… 외부 연동 Bean들...)

그런 configuration들을 import해주기 위해 @ImportResource Annotation을 사용하였다.
사용법은 간단하다. @ImportResource Annotation value에 배열 형태로 선언만 해주면 되기 때문이다.

예시는 아래와 같다.



~~~java
@Configuration
@ImportResource(value = {
		"classpath:applicationContextForExternalMember.xml",    //External-Member
		"classpath*:applicationContextForExternalAPI.xml",      //External-API
		"classpath*:applicationContextForExternalLogger.xml"    //External-Logger
})
public class ExternalConfig {
}
~~~

 위와같이 설정하면, @Configuration Annotation에 의해 Component-scan 되는 시점에 해당 파일들이 import되어 Configuration 설정이 적용된다.