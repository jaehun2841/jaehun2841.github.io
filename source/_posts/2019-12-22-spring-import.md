---
title: Custom @Enable annotation 만들어보기
catalog: true
Categories:
  - Spring
  - Spring Configuration
tags:
  - Spring
  - Spring Configuration
typora-root-url: 2019-12-22-spring-import
typora-copy-images-to: 2019-12-22-spring-import
date: 2019-12-22 16:04:10
subtitle: Import를 사용하면서 ImportAware, ImportSelector, ImportBeanDefinitionRegistrar를 사용해 봅시다.
---



# Multi 모듈 프로젝트에서 발생하는 일

Maven, Gradle을 이용해서 멀티 모듈을 구성하다 보면 각 모듈 별로 중복된 Bean을 계속 생성 코드를 만드는 경우가 있습니다.  
(ex: Datasource, TransactionManager 등등)  



> 예시 : Custom Module에서만 사용하는 JPA 환경 구성입니다.  
> 일반적으로 하나의 DataSource를 사용하신다면, Spring AutoConfiguration을 이용하면 편리하게 구성하실 수 있습니다.  
> 하지만 실무에서는 DataSource를 2개 이상 구성해서 쓰는 경우도 있기에, 직접 경험한 예시를 사용하게 되었습니다.

```java
@EntityScan(basePackages = {"com.your.packages.module1"})
@EnableJpaRepositories(basePackages = {"com.your.packages.module1"})
@Configuration
class Module_1_Configuration {
  
  @Bean
  @ConfigurationProperties(prefix = "spring.custom-module.datasource")
  public HikariConfig hikariConfig() {
    return new HikariConfig();
  }
  
  @Bean
  @FlywayDatasource
  public DataSource dataSource(HikariConfig hikariConfig) {
    return new HikariDataSource(hikariConfig);
  }
  
  @Bean
  public LocalContainerEntityManagerFactoryBean entityManagerFactory(
    EntityManagerFactoryBuilder builder,
    DataSource dataSource
  ){
    return builder.dataSource(dataSource)
      .packages("com.your.packages.module1")
      .build();
  }
   
  @Bean
  public JpaTransactionManager transactionManager(EntityManagerFactory entityManagerFactory) {
    return new JpaTransactionManager(entityManagerFactory);
  }
}
```

```java
@EntityScan(basePackages = {"com.your.packages.module2"})
@EnableJpaRepositories(basePackages = {"com.your.packages.module2"})
@Configuration
class Module_2_Configuration {
  
  @Bean
  @ConfigurationProperties(prefix = "spring.custom-module.datasource")
  public HikariConfig hikariConfig() {
    return new HikariConfig();
  }
  
  @Bean
  public DataSource dataSource(HikariConfig hikariConfig) {
    return new HikariDataSource(hikariConfig);
  }
  
  @Bean
  public LocalContainerEntityManagerFactoryBean entityManagerFactory(
    EntityManagerFactoryBuilder builder,
    DataSource dataSource
  ){
    return builder.dataSource(dataSource)
      .packages("com.your.packages.module2")
      .build();
  }
   
  @Bean
  public JpaTransactionManager transactionManager(EntityManagerFactory entityManagerFactory) {
    return new JpaTransactionManager(entityManagerFactory);
  }
}
```



위의 두 Configuation 코드는 99% 같은 코드입니다.  
하지만 왜 같은 코드 두개를 짰을까요?  

첫 번째는 packages 설정이 달랐습니다.

* packages("com.your.packages.module1")
* packages("com.your.packages.module2")

두 번째는 EntityScan, EnableJpaRepositories의 basePackages 설정이 달랐습니다.

* basePackages = {"com.your.packages.module1"}
* basePackages = {"com.your.packages.module2"}

세 번째는

* module1에서는 DataSource Bean에 `@FlywayDataSource` annotation이 있고
* module2에서는 DataSource Bean에 `@FlywayDataSource` annotation이 없습니다.

코드가 거의 같더라도, 약간의 설정이 다르다는 이유로 거의 같은 두 개의 Configuration을 사용했습니다.



# 리팩토링1 - @EnableCustomDataSource

Spring을 사용하다 보면 @Enable~ 하는 Annotation을 자주 보았을 것이고, 실제로도 많이 사용해 보셨을 겁니다.  

* EnableCaching
* EnableTransactionManagement
* EnableJpaRepositories
* EnableConfigurationProperties
* EnableAsync

이 처럼 많은 Enable Annotation들을 Spring에서 사용하고 있고, 실제로 개발자가 해야할 귀찮은   
Configuration이나 BeanPostProcessing 같은 처리들을 손쉽게 하도록 도와줍니다.  

그렇기 때문에 이번 리팩토링도 Spring의 이런 부분을 벤치마킹하여 `@EnableCustomDataSource` 를 한번 만들어 보겠습니다.   

최종적인 모습은 아래와 같은 코드로 깔끔하게 리팩토링을 하는 그림이면 좋겠네요

```java
@EnableCustomDataSource(basePackage = {"com.your.packages.module1"})
@Configuration
class Module1Configuration {
}
```


# 리팩토링2 - 다른 @Enable~ Annotation 내부는 어떻게 생겼을까?

![EnableAsync](./EnableAsync.png)

![EnableConfigurationProperties](./EnableConfigurationProperties.png)

![EnableJpaRepositories](./EnableJpaRepositories.png)



혹시 3개의 Annotation의 공통점이 보이시나요?  
모두 `@Import` Annotation을 통해 어떤 class를 import하고 있습니다.  
대부분의 Enable의 Annotation은 위와 같이 Configuration 클래스를 Import하는 형식으로 많이 작성되고 있습니다.  
(이외에는 SpringAutoConfiguration으로 작동하는 것들도 있는 것 같습니다.)



# 리팩토링3 - @EnableCustomDataSource 생성

위 에서 많은 개발자들이 만든 @Enable Annotation 생성 방식을 모방하여 만들어 보겠습니다.

```java
@Target(ElementType.Type)
@Retention(RetentionPolicy.RUNTIME)
@Import(??.class)
public @interface EnableCustomDataSource {
}
```

일단 @EnableCustomDataSource를 만들었습니다.  
근데 @Import Annotation에 지정할 Configuration Class가 없어서.. 어떻게 해야 할지 잘 모르겠네요.  
위에는 2개의 Configuration이 있는데 말이죠



# 리팩토링 4 - ImportSelector를 이용한 선택적 Configuration 사용

spring-context 라이브러리에서는 @Import Annotation processing에 대한 3가지 Interface를 지원합니다.

* ImportSelector
* ImportBeanDefinitionRegistrar
* ImportAware

먼저 ImportSelector를 사용하여 선택적으로 Configuration을 사용해 보겠습니다.
ImportSelector는 Annotation Attribute의 값에 따라 Import할 Configuration을 지정할 수 있습니다.  

그렇기 때문에 @EnableCustomDataSource의 Attribute를 수정해보았습니다.  
```java
@Target(ElementType.Type)
@Retention(RetentionPolicy.RUNTIME)
@Import(CustomDataSourceConfigurationSelector.class)
public @interface EnableCustomDataSource {
  Module module();
}
```
```java
enum Module { ONE, TWO }
```


ImportSelector를 구현한 CustomDataSourceConfigurationSelector 코드를 구성하였습니다.  
```java
class CustomDataSourceConfigurationSelector implements ImportSelector {
  
  @Override
  public String[] selectImports(AnnotationMetadata importingClassMetadata) {
    
    //find annotation attribute (module)
    Map<String, Object> metaData = importingClassMetadata.getAnnotationAttributes(EnableCustomDataSource.class.getName());
    AnnotationAttributes attributes = AnnotationAttributes.fromMap(metaData);
    Module module = attributes.getEnum("module");
    
    //determine configuration class
    String configurationClass = null;
    switch (module) {
        case ONE:
          configurationClass = Module_1_Configuration.class.getName();
          break;
        case TWO:
          configurationClass = Module_2_Configuration.class.getName();
          break;
    }
    
    return new String[]{configurationClass};
  }
}
```


Module1의 CustomDataSourceConfiguration에서 Module.ONE이라고 설정을 정의해주면   
Module_1_Configuration에 대한 설정을 Import할 수 있습니다.
```java
@Configuration
@EnableCustomDataSource(module = Module.ONE)
class CustomDataSourceConfiguration {
}
```



# 리팩토링 5 - ImportBeanDefinitionRegistrar를 이용한 직접 Bean 생성하기 

ImportBeanDefinitionRegistrar를 이용하면 직접 beanRegistry에 Bean을 등록할 수 있습니다.
> 하지만 추천하는 방식은 아닙니다.  
> 실제로 Bean이 등록되는 순서를 제어하기 어렵기 때문에 원하는대로 실행이 되지 않을 수 있습니다.  
> 토비의 스프링 ver2에서도 가급적이면 사용을 하지 않을 것을 권장하고 있습니다. 

```java
public class CustomDataSourceBeanDefinitionRegistrar implements ImportBeanDefinitionRegistrar {

    private static final String BEAN_NAME = "customDataSourceConfiguration"
    @Override
    public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
      Map<String, Object> metaData = importingClassMetadata.getAnnotationAttributes(EnableCustomDataSource.class.getName());
      AnnotationAttributes attributes = AnnotationAttributes.fromMap(metaData);
      EnableCustomDataSource.Module module = attributes.getEnum("module");

      AbstractBeanDefinition beanDefinition = null;
      switch (module) {
          case ONE:
              beanDefinition = BeanDefinitionBuilder
                      .rootBeanDefinition(Module_1_Configuration.class)
                      .getBeanDefinition();
              break;
          case TWO:
              beanDefinition = BeanDefinitionBuilder
                      .rootBeanDefinition(Module_2_Configuration.class)
                      .getBeanDefinition();
              break;
      }
      registry.registerBeanDefinition(BEAN_NAME, beanDefinition);
    }
}
```

```java
@Target(ElementType.Type)
@Retention(RetentionPolicy.RUNTIME)
@Import(CustomDataSourceBeanDefinitionRegistrar.class)
public @interface EnableCustomDataSource {
  Module module();
}
```



# 리팩토링 6 - ImportAware를 이용한 Annoataion attribute 주입

마지막으로 ImportAware입니다.

ImportAware interface는 @Import하는 Configuration class에 @Import 메타 annoatation을 사용하는 annoatation의 attribute를 사용할 수 있도록 해줍니다.

```java
@Target(ElementType.Type)
@Retention(RetentionPolicy.RUNTIME)
@Import(CustomDataSourceConfiguration.class)
public @interface EnableCustomDataSource {
}
```



```java
@Configuration
public class CustomDataSourceConfiguration implements ImportAware {
  @Bean
  @ConfigurationProperties(prefix = "spring.custom-module.datasource")
  public HikariConfig hikariConfig() {
    return new HikariConfig();
  }
  
  @Bean
  public DataSource dataSource(HikariConfig hikariConfig) {
    return new HikariDataSource(hikariConfig);
  }
  
  @Bean
  public LocalContainerEntityManagerFactoryBean entityManagerFactory(
    EntityManagerFactoryBuilder builder,
    DataSource dataSource
  ){
    return builder.dataSource(dataSource)
      .packages("com.your.packages.module2")
      .build();
  }
   
  @Bean
  public JpaTransactionManager transactionManager(EntityManagerFactory entityManagerFactory) {
    return new JpaTransactionManager(entityManagerFactory);
  }
  
  @Override
  public void setImportMetadata(annoatationMetadata: AnnotationMetadata) {
    //use @EnableCustomDataSource annotation attribute
  }
}
```

위와 같이 @EnableCustomDataSource annotation의 attribute를 사용할 수 있는 메서드를 제공합니다.

그렇게 되면, 애초에 2개의 파일로 분리되었던 

* Module_1_Configuration
* Module_2_Configuration

이 두가지 파일을 하나로 합칠 수 있을 것 같습니다.

두개의 파일을 하나로 합쳐서 CustomDataSourceConfiguration이라는 class를 만들고  
@EnableCustomDataSource annotation에서 @Import 해보겠습니다.



## basePackages attribute 추가

@EnableCustomDataSource annotation의 attribute를 조금 변경해 보겠습니다.  
여태까지 사용한 module attribute는 이제 의미가 없을 것 같습니다.  
파일을 하나로 합치기로 했기 때문이죠

```java
@Target(ElementType.Type)
@Retention(RetentionPolicy.RUNTIME)
@Import(CustomDataSourceConfiguration.class)
public @interface EnableCustomDataSource {
  String[] basePackages() default {};
}
```



Module1Configuration class에 드디어 우리가 원하던 대로 @EnableCustomDataSource를 사용해보게 되었습니다.  
이게 껍데기는 완성되었고 내부 로직만 잘 구성하면 될듯 합니다.

```java
@Configuration
@EnableCustomDataSource(basePackages = {"com.your.package.module1"})
class Module1Configuration {
}
```



 ## ImportAware 메서드 코드 구성

```java
@Configuration
public class CustomDataSourceConfiguration implements ImportAware {
  
  private String[] basePackages;
  
  @Bean
  @ConfigurationProperties(prefix = "spring.custom-module.datasource")
  public HikariConfig hikariConfig() {
    return new HikariConfig();
  }
  
  @Bean
  public DataSource dataSource(HikariConfig hikariConfig) {
    return new HikariDataSource(hikariConfig);
  }
  
  @Bean
  public LocalContainerEntityManagerFactoryBean entityManagerFactory(
    EntityManagerFactoryBuilder builder,
    DataSource dataSource
  ){
    return builder.dataSource(dataSource)
      .packages(this.basePackages)
      .build();
  }
   
  @Bean
  public JpaTransactionManager transactionManager(EntityManagerFactory entityManagerFactory) {
    return new JpaTransactionManager(entityManagerFactory);
  }
  
  @Override
  public void setImportMetadata(annoatationMetadata: AnnotationMetadata) {
    //use @EnableCustomDataSource annotation attribute
    Map<String, Object> metaData = importingClassMetadata.getAnnotationAttributes(EnableCustomDataSource.class.getName());
    AnnotationAttributes attributes = AnnotationAttributes.fromMap(metaData);
    
    String[] basePackages = attributes.get("basePackages");
    this.basePackages = basePackages; // initialize member variable basePackages
  }
}
```

CustomDataSourceConfiguration을 @Import를 통해 Import하게 되면 @Bean 메서드 보다 setImportMetadata가 먼저 실행됩니다.  
(이유는 ImportAwareBeanPostProcessor에 의해 @Configuration Bean 초기화 이전에 setImportMetadata 메서드가 실행되기 때문입니다.) 

> 주의: ImportAware를 구현한 Configuration class에는 반드시 @Configuration Annotation을 붙여야 합니다.
> 그래야 Spring 내부적으로 ConfigurationClassPargser가 작동할 때 scan 되어 나중에 ImportAwareBeanPostProcessor에 의해 Post Processing이 될 수 있습니다.



# 리팩토링 7 - 아직 끝나지 않았다.

아직 끝나지 않았습니다.

Configuration class를 두 개로 나눈 두번째 이유 @EntityScan, @EnableJpaRepositories의 basePackages에 대한 설정이 다르기 때문입니다.  
이 문제를 해결하기 위해서는 Spring에서 제공하는 @AliasFor annoataion을 사용해 보았습니다.

![AliasFor](./AliasFor.png)

@AliasFor Annotation은 Spring 4.2에서 추가된 Annotation입니다. 

* AliasFor을 이용해 Attribute의 다른 속성에 값을 바인딩 할 수 있습니다.  
  위의 예시처럼 value로 들어온 값을 attribute의 값에 바인딩
* Annotation의 메타 Annotation의 attribute에 값을 바인딩 할 수 있습니다.
  **(우리는 이 기능을 사용해 보도록 할 것입니다.)**

> 주의: Spring framework 5.2 하위버전에서 AliasFor이 Array에 대해서는 잘 적용이 안되는 이슈가 있었습니다.  
> 이 이슈는 Spring framework 5.2, Springboot 2.2.0 이후에는 잘 적용이 됩니다.  
> 혹시나 적용이 안되시는 분은 Spring version up을 권장드립니다.



## @EnableCustomDataSource 수정

```java
@Target(ElementType.Type)
@Retention(RetentionPolicy.RUNTIME)
@EntityScan // meta annotation으로 추가
@EnableJpaRepositories // meta annotation으로 추가
@Import(CustomDataSourceConfiguration.class)
public @interface EnableCustomDataSource {
  @AliasFor(annotation = EntityScan.class, attribute = "basePackages")
  String[] jpaEntityBasePackages() default {};
  @AliasFor(annotation = EnableJpaRepositories.class, attribute = "basePackages")
  String[] jpaRepositoriesBasePackages() default {};
}
```

@AliasFor annotation은 @Repeatable meta annotation이 없기 때문에 하나의 메서드에 중첩해서 사용할 수 없습니다.  
그렇기 때문에 @EntityScan에 대한 basePackages를 설정할 수 있는 메서드와  
@EnableJpaRepositories에 대한 basePacakges를 설정할 수 있는 메서드로 분리 하였습니다.



위와 같이 설정해 주면  
jpaEntityBasePackages -> `EntityScan.basePackages`로 바인딩 됩니다.  
jpaRepositoriesBasePackages -> `EnableJpaRepositories.basePackages`로 바인딩 됩니다.  
물론 EnableCustomDataSource의 attribute로도 사용할 수 있습니다.



## CustomDataSourceConfiguration 수정

```java
@Configuration
public class CustomDataSourceConfiguration implements ImportAware {
  
  private String[] basePackages;
  
  ...
  
  @Override
  public void setImportMetadata(annoatationMetadata: AnnotationMetadata) {
    //use @EnableCustomDataSource annotation attribute
    Map<String, Object> metaData = importingClassMetadata.getAnnotationAttributes(EnableCustomDataSource.class.getName());
    AnnotationAttributes attributes = AnnotationAttributes.fromMap(metaData);
    
    String[] basePackages = attributes.get("jpaEntityBasePackages");
    this.basePackages = basePackages; // initialize member variable basePackages
  }
}
```

attribute가 변경되어 attribute명을 변경하였습니다.





# 리팩토링 8 - FlywayDataSource는 어째요?

Flyway는 Database 자동 구성에 도움을 주는 라이브러리입니다.  
(저장된 sql을 versioning하고 내부적으로 version관리를 하여 자동으로 Database를 초기화 하도록 해줍니다.)

이런 FlywayDatasource는 보통 하나의 모듈에서만 초기화 하게됩니다.  
보통 하나의 DataSource만 사용하는 상황에서는 Flyway를 적용하는 대상도 하나이기 때문에 문제가 안되지만  
지금과 같은 다중 DataSource를 사용하는 상황에서는 제약이 생길 수 있습니다.

그래서 @FlywayDataSource annotation을 지정하여 이 DataSource에 Flyway를 적용할 지 말지 결정할 수 있습니다.  
여기서는 Springboot에서 제공하는 @ConditionalOnProperty를 이용하여 Bean 생성을 해보도록 하겠습니다.

![ConditionalOnProperty](./ConditionalOnProperty.png)

* **prefix**: 사용하고자 하는 property의 prefix입니다.
* **name**: Condition에 유효한 property인지 테스트할 property명
* **havingValue**: prefix + name property의 값이 어떤 value일 경우 조건식이 참이 될지 결정하는 값입니다.
* **matchIfMissing**: property를 찾을 수 없는 경우 조건식의 default 값을 정의합니다.



## property 추가

```properties
spring.custom.datasource.flyway.enable = true
```

이와 같이 custom property를 추가해보겠습니다. (custom datasource에 대한 flyway를 쓸지 말지에 대한 설정입니다.)



## CustomDataSourceConfiguration 수정

```java
@Configuration
public class CustomDataSourceConfiguration implements ImportAware {
  @Bean
  @ConfigurationProperties(prefix = "spring.custom-module.datasource")
  public HikariConfig hikariConfig() {
    return new HikariConfig();
  }
  
  @ConditionalOnProperty(
    prefix = "spring.custom.datasource.flyway", 
    name = ["enable"],
    havingValue = "true", 
    matchIfMissing = false
  )
  @FlywayDataSource
  @Bean("dataSource")
  public DataSource dataSourceUsingFlyway(HikariConfig hikariConfig) {
    return new HikariDataSource(hikariConfig);
  }
  
  @ConditionalOnProperty(
    prefix = "spring.custom.datasource.flyway", 
    name = ["enable"],
    havingValue = "false", 
    matchIfMissing = true
  )
  @Bean("dataSource")
  public DataSource dataSource(HikariConfig hikariConfig) {
    return new HikariDataSource(hikariConfig);
  }
  
  @Bean
  public LocalContainerEntityManagerFactoryBean entityManagerFactory(
  	EntityManagerFactoryBuilder builder,
    DataSource dataSource
  ){
    return builder.dataSource(dataSource)
      .packages("com.your.packages.module2")
      .build();
  }
   
  @Bean
  public JpaTransactionManager transactionManager(EntityManagerFactory entityManagerFactory) {
    return new JpaTransactionManager(entityManagerFactory);
  }
  
  @Override
  public void setImportMetadata(annoatationMetadata: AnnotationMetadata) {
    //use @EnableCustomDataSource annotation attribute
    Map<String, Object> metaData = importingClassMetadata.getAnnotationAttributes(EnableCustomDataSource.class.getName());
    AnnotationAttributes attributes = AnnotationAttributes.fromMap(metaData);
    
    String[] basePackages = attributes.get("jpaEntityBasePackages");
    this.basePackages = basePackages; // initialize member variable basePackages
  }
}
```

이와 같이 CustomDataSourceConfiguration에서 DataSource Bean을 생성하는 코드를 두개로 하고  
`spring.custom.datasource.flyway.enable` 값에 따라 자동으로 Bean을 생성하도록 코드를 구성하였습니다.  

FlywayDataSource를 사용하는 module에만 `spring.custom.datasource.flyway.enable` 를 추가하여 Bean이 생성되도록 하였습니다.  
FlywayDataSource를 사용하지 않는 module에는 `spring.custom.datasource.flyway.enable` 를 설정하지 않고 사용하여  
일반적인 DataSource만 사용하도록 구성하였습니다.



# 참고

* 토비의 스프링 Ver.2
* [Spring Conditional annotation](<https://medium.com/@circlee7/spring-conditional-annotation-e288ccf6b536>)
* [Spring AliasFor](https://docs.spring.io/spring/docs/4.2.0.RC2_to_4.2.0.RC3/Spring%20Framework%204.2.0.RC3/org/springframework/core/annotation/AliasFor.html)


