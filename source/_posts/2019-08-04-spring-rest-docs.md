---

layout: posts
title: Spring Rest Docs를 이용한 API 문서 만들기
date: 2019-08-04 15:02:21
typora-root-url: 2019-08-04-spring-rest-docs
typora-copy-images-to: 2019-08-04-spring-rest-docs
tags: 
- Spring
- Spring-Rest-Docs
---

Spring Rest API 문서를 자동으로 생성하고자  할 때, 보통 Swagger로 많이 사용하지만
이번에는 Spring Rest Docs를 사용하여 API 문서를 자동으로 작성 할 수 있도록 해봤습니다.

포스팅에 작성된 코드는 <https://github.com/jaehun2841/spring-rest-docs-example> 에서 참고하시길 바랍니다.

# Spring Rest Docs란

Spring Rest Docs는 테스트 코드를 기반으로 자동으로 API문서를 작성할 수 있게 해주는 프레임워크입니다.  
그렇기 때문에 반드시 `Test가 통과되어야 문서가 작성` 된다는 장점이 있습니다.  
(그렇기 때문에 API Spec이 변경되거나 추가/삭제 되는 부분에 대해 항상 테스트 코드를 수정하여야 하며, API 문서가 최신화 될 수 있도록 해줍니다.)  
기본적으로 asciidoc을 사용하여 문서를 작성합니다 . (asciidoc은 마크다운과 비슷하게 html문서를 작성할 수 있는 언어입니다.)  
원하는 경우에는 mark down을 사용할 수도 있지만, 이번 포스트에서는 asciidoc을 이용하여 API 문서를 작성해 보도록 하겠습니다.

# Spring Rest Docs Architecture

![asciidoctor-task](./asciidoctor.png)

* Test Case를 수행하면 산출물이 <document-name>.adoc 파일로 /build/generate-snippets 디렉토리에 생성됩니다. (default path)
* /src/docs/asciidoc 디렉토리에 /build/generate-snippets에 있는 adoc 파일을 include하여 문서를 생성할 수 있습니다.
  * /build/generate-snippets/\*.adoc 파일들은 API Request, Response에 대한 명세들만 있는 파일이고
  * /src/docs/asciidoc/*.adoc 파일들이 실제 사용자에게 html파일로 변환되어 제공되는 API 문서 파일입니다.
  * 따라서 /src/docs/asciidoc/\*.adoc 에 API 문서를 작성하고,   
    필요한 API Request, Response Spec은 자동 생성된 /build/generate-snippets/*.adoc 파일들을 이용해 표현해줍니다.
  * 이렇게 하면 향후 API Spec이 변경되더라도, 문서를 수정하지 않아도 되는 장점이 있습니다.
* 이렇게 생성된 asciidoc 문서는 AsciidoctorTask를 통해 html 문서로 processing 되어 /build/asciidoc/html5 하위에 html문서로 생성됩니다.
  * html 문서가 생성되는 기준은 /src/docs/asciidoc/*.adoc 파일을 기준으로 생성됩니다.



## 예시
### /src/docs/asciidoc 디렉토리 내에 html로 제공될 문서를 asciidoc으로 작성
![docuement-templates](./src-template.png)
### Test Case의 산출물 
* /build/generated-snippets 디렉토리 하위에 생성
* Request, Response Spec에 대한 정보를 생성
* /src/docs/asciidoc/*.adoc 파일에서 include해서 사용 
![snippets](./snippets.png)
### /src/docs/asciidoc 디렉토리 adoc파일을 기반으로 생성된 html 파일
![processed-html](./processed-html.png)

자세한 부분은 아래 예시를 따라하면서 보시면 좋을 것 같습니다.

# Spring Rest Docs 시작하기

## 개발 스펙

* Spring Boot 2.1.6.RELEASE
* Junit 5
* Kotlin 1.3.41 (저희 팀에서 코틀린만 써서 코틀린이 편하네요)
* Gradle 5.4.1
* Asciidoctor 1.5.9.2

## Test Library

Spring Rest Docs는 3가지 테스트 라이브러리를 지원합니다.

* MockMvc (@WebMvcTest)
* WebTestClient (Mono / Flux, @WebTestClient)
* Rest Assured (IntegrationTest, @SpringBootTest)

API 문서를 작성할 때 Spring Mvc를 사용하는 환경이라면 가장 가볍게 돌릴 수 있는게 MockMvc를 사용하는 것이라 생각합니다.  
그래서 이번 포스팅에서는 MockMvc를 사용한 예제로만 진행하도록 하겠습니다.

## Gradle 설정

### asciidoctor plugin 설정

```groovy
buildscript {
    ext {
        kotlinVersion = '1.3.41'
        springBootVersion = "2.1.6.RELEASE"
    }

    repositories {
        mavenCentral()
    }
    dependencies {
        classpath "org.springframework.boot:spring-boot-gradle-plugin:$springBootVersion"
        classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlinVersion"
    }
}

plugins {
    id 'org.asciidoctor.convert' version '1.5.9.2'
    id 'org.springframework.boot' version '2.1.6.RELEASE'
    id "io.spring.dependency-management" version "1.0.5.RELEASE"
}

apply plugin: 'io.spring.dependency-management'
apply plugin: 'kotlin'
apply plugin: "groovy"
apply plugin: "org.springframework.boot"
apply plugin: "io.spring.dependency-management"
apply plugin: "kotlin-kapt"

group = 'com.example'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '1.8'

repositories {
    mavenCentral()
}

ext {
    set('snippetsDir', file("build/generated-snippets"))
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    testImplementation("org.springframework.boot:spring-boot-starter-test") {
        exclude group: "junit", module: "junit"
    }
    testImplementation "org.junit.jupiter:junit-jupiter-api"
    testImplementation "org.junit.jupiter:junit-jupiter-params"
    testRuntimeOnly "org.junit.jupiter:junit-jupiter-engine"
    testImplementation 'org.springframework.restdocs:spring-restdocs-mockmvc'
    testImplementation "com.nhaarman:mockito-kotlin:1.6.0"
    implementation "org.jetbrains.kotlin:kotlin-stdlib-jdk8:$kotlinVersion"
}

kapt {
    useBuildCache = true
    correctErrorTypes = true
}

test {
    useJUnitPlatform()
    outputs.dir snippetsDir
}

asciidoctor {
    inputs.dir snippetsDir
    dependsOn test //test Task 이후에 실행될 수 있도록 Dependency를 설정
}

asciidoctor.doFirst {
    println "=====start asciidoctor"
    //asciidoctor 실행전 기존에 생성된 API 문서 삭제
    delete file('src/main/resources/static/docs')
}
asciidoctor.doLast {
    println "=====finish asciidoctor"
}

task copyDocument(type: Copy) {
    dependsOn asciidoctor
    from file("build/asciidoc/html5")
    // resources/static/docs 로 복사하여 서버가 돌아가고 있을때 /docs/index.html 로 접속하면 볼수 있음
    into file("src/main/resources/static/docs")  
}

build {
    dependsOn copyDocument
}

bootJar {
    archiveName = 'app.jar'
    dependsOn asciidoctor
    //실제 배포 시, BOOT-INF/classes가 classpath가 됩니다.
    from ("${asciidoctor.outputDir}/html5") {
        into "BOOT-INF/classes/static/docs"
    }
}

compileKotlin {
    kotlinOptions {
        freeCompilerArgs = ["-Xjsr305=strict"]
        jvmTarget = "1.8"
    }
}

compileTestKotlin {
    kotlinOptions {
        freeCompilerArgs = ["-Xjsr305=strict"]
        jvmTarget = "1.8"
    }
}

jar.enabled = false
bootJar.enabled = true
bootJar.mainClassName = 'com.example.restdocs.RestdocsApplication'

```

* `testImplementation 'org.springframework.restdocs:spring-restdocs-mockmvc'` : mockMvc 테스트를 통해 API adoc 파일을 생성해주도록 하는 라이브러리 입니다.
* `id 'org.asciidoctor.convert' version '1.5.9.2'` : asciidoc 파일을 html 파일로 processing 해주는 플러그인 입니다.
* `asciidoctor` :  asciidoc 파일을 html 파일로 processing 해주는 Gradle Task를 정의합니다.



## User API 코드 작성

## UserController.kt

```kotlin
@Suppress("SpringJavaInjectionPointsAutowiringInspection")
@RestController
@RequestMapping("/user")
open class UserController(
        val userService: UserService
) {

  @GetMapping("/{userId}")
  fun getUser(@PathVariable(value = "userId") userId: Long): Response<User> {
    val searchUser = userService.search(userId)
    return Response.success(searchUser)
  }

  @PostMapping
  fun createUser(@RequestBody userDto: UserDto): Response<User> {
    val createUser = userService.create(User(name = userDto.name, address = userDto.address, age = userDto.age))
    return Response.success(createUser)
  }


  @PutMapping("/{userId}")
  fun updateUser(@PathVariable("userId") userId: Long,
                 @RequestBody userDto: UserDto): Response<User> {
    val updateUser = userService.update(User(id = userId, name = userDto.name, address = userDto.address, age = userDto.age))
    return Response.success(updateUser)
  }

  @DeleteMapping("/{userId}")
  fun deleteUser(@PathVariable(value = "userId") userId: Long): Response<Unit> {
    userService.delete(userId)
    return Response.success()
  }

  @PostMapping("/{userId}/role/{roleId}")
  fun grantRole(@PathVariable(value = "userId") userId: Long,
                @PathVariable(value = "roleId") roleId: Long): Response<Unit> {
     userService.grantRole(userId = userId, roleId = roleId)
    return Response.success()
  }
}
```

## UserService.kt

```kotlin
interface UserService {

  fun search(userId: Long): User?
  fun create(user: User): User?
  fun update(user: User): User?
  fun delete(userId: Long)
  fun grantRole(userId: Long, roleId: Long): User?
}
```

## Response.kt

```kotlin
data class Response<T>(
        val code: Int,
        val message: String,
        val data: T?,
        val error: T?
) {

  companion object {
    fun success(): Response<Unit> = success(null)
    fun <T> success(data: T?): Response<T> = Response(200, "OK", data, null)
    fun <T> error(error: T?): Response<T> = Response(500, "Server Error", null, error)
  }
}
```

## User.kt

```kotlin
data class User(
        val id: Long? = null,
        val name: String,
        val age: Int,
        val address: String,
        var roles: MutableList<Role> = mutableListOf()
)
```



## Test 코드 작성

```kotlin
@ExtendWith(RestDocumentationExtension::class, SpringExtension::class) // (1)
@WebMvcTest(UserController::class, secure = false) // (2)
@AutoConfigureRestDocs // (3)
class UserControllerTest {

  @Autowired
  lateinit var mockMvc: MockMvc // (4)

  @MockBean
  lateinit var userService: UserService // (5)

  @Test
  fun `user search api docs`() {

    //given
    given(userService.search((eq(1L))))
            .willReturn(User(1, "배달이", 30, "서울특별시 송파구 올림픽로 295")) // (6)

    //when
    val resultActions = mockMvc.perform(
            RestDocumentationRequestBuilders.get("/user/{userId}", 1)
                    .header("x-api-key", "API-KEY")
                    .accept(MediaType.APPLICATION_JSON_UTF8)
    ).andDo(MockMvcResultHandlers.print()) // (7)

    //then
    resultActions
            .andExpect(status().isOk) // (8)
            .andDo(
                    document(
                            "user-search", // (9)
                            getDocumentRequest(), // (10)
                            getDocumentResponse(), // (11)
                            requestHeaders(*header()),  // (12)
                      			pathParameters(userIdPathParameter()),  // (13)
                            responseFields(*common())  // (14)
                                    .andWithPrefix("data.", *user()) 
                                    .andWithPrefix("data.roles[].", *role())
                    )
            )
  }

  {...} //중략..
  
  private fun getUserDto(): String {
    return """
      {
        "name": "배달이",
        "age": 30,
        "address": "서울특별시 송파구 올림픽로 295"
      }
    """.trimIndent()
  }

  private fun header(): Array<HeaderDescriptor> {
    return arrayOf(headerWithName("x-api-key").description("Api Key"))
  }

  private fun userIdPathParameter(): ParameterDescriptor {
    return parameterWithName("userId").description("USER ID")
  }

  private fun roleIdPathParameter(): ParameterDescriptor {
    return parameterWithName("roleId").description("ROLE ID")
  }

  private fun common(): Array<FieldDescriptor> {
    return arrayOf(
            fieldWithPath("code").type(JsonFieldType.NUMBER).description("응답 코드"),
            fieldWithPath("message").type(JsonFieldType.STRING).description("응답 메세지"),
            subsectionWithPath("error").type(JsonFieldType.OBJECT).description("에러 Data").optional(),
            subsectionWithPath("data").type(JsonFieldType.OBJECT).description("응답 Data").optional()
    )
  }

  private fun user(): Array<FieldDescriptor> {
    return arrayOf(
            fieldWithPath("id").type(JsonFieldType.NUMBER).description("User ID"),
            fieldWithPath("name").type(JsonFieldType.STRING).description("이름"),
            fieldWithPath("age").type(JsonFieldType.NUMBER).description("나이"),
            fieldWithPath("address").type(JsonFieldType.STRING).description("주소")
    )
  }

  private fun role(): Array<FieldDescriptor> {
    return arrayOf(
            fieldWithPath("id").type(JsonFieldType.NUMBER).description("Role ID").optional(),
            fieldWithPath("name").type(JsonFieldType.STRING).description("Role명").optional()
    )
  }
}
```

```kotlin
object RestApiDocumentUtils {

  fun getDocumentRequest(): OperationRequestPreprocessor { // (10)
    return Preprocessors.preprocessRequest(
            modifyUris()
              .scheme("http")
              .host("user.api.com")
              .removePort(),
            Preprocessors.prettyPrint()
    )
  }

  fun getDocumentResponse(): OperationResponsePreprocessor { // (11)
    return Preprocessors.preprocessResponse(Preprocessors.prettyPrint())
  }
}
```



### 코드 설명

1. Junit5에서 Spring Rest Docs를 사용할 때, RestDocumentationExtension::class, SpringExtension::class Extension 두개를 사용합니다.  
  Junit 4 에서 RunWith와 같은 기능입니다.
2. @WebMvcTest annotation을 사용하여, mockMvc를 사용할 수 있는 환경을 설정합니다.  
  1. 이 예제에서는 UserController에 대한 테스트와 API문서를 작성하므로, controller를 UserController로 지정합니다.
  2. spring security를 사용하는 경우, `secure=false` 옵션을 통해 Spring Security 사용 안함으로 설정할 수 있습니다.
3. Spring Rest Docs에 대한 Auto Configuration을 설정합니다.
4. mockMvc를 사용하기 위해 MockMvc bean을 Autowiring 해줍니다.
5. UserService에 대한 Mocking을 위해 @MockBean annotation을 통해 Test Context에 bean으로 등록해줍니다.
6. UserService.search() function에 대한 stubbing을 해줍니다. (호출 시, return 되는 값 지정)
7. mockMvc를 이용하여, `GET /user/1` API를 호출 합니다.
8. API 호출 결과에 대해 간단한 status체크 정도로 테스트 항목을 추가했습니다.
9. andDo function으로 Asciidoc을 생성하도록 설정합니다.  
  `user-search`는 테스트가 수행 된 후, adoc 파일이 생성될 디렉토리 이름입니다.  
  `/build/generate-snippets/user-search/*.adoc` path에 adoc 파일이 생성됩니다.
10. DocumentRequest에 대한 설정을 추가합니다.
  1. modifyUris()를 통해 adoc 파일에 어떤 도메인으로 API를 호출 할 지, 정의할 수 있습니다.
    (Default는 `http://localhost:8080` 입니다.)
  2. Request Json을 이쁘장하게 출력하도록 해줍니다
11. DocumentResponse에 대한 설정을 추가합니다.
   1. Response Json을 이쁘장하게 출력하도록 해줍니다
12. requestHeader 정보를 추가합니다.
13. path parameter 정보를 추가합니다.
14. responseFields 정보를 추가합니다.



## Document 생성하기

asciidoctor를 이용하면 6개의 snippet 파일을 기본적으로 생성해줍니다.

### 예시

* `<output-directory>/<document-name>/curl-request.adoc` 
```
[source,bash]
----
$ curl 'http://user.api.com/user/1' -i -X GET \
    -H 'Accept: application/json;charset=UTF-8' \
    -H 'x-api-key: API-KEY'
----
```
* `<output-directory>/<document-name>/http-request.adoc`
```
[source,http,options="nowrap"]
----
GET /user/1 HTTP/1.1
Accept: application/json;charset=UTF-8
Host: user.api.com
x-api-key: API-KEY

----
```
* `<output-directory>/<document-name>/http-response.adoc`
```
[source,http,options="nowrap"]
----
GET /user/1 HTTP/1.1
Accept: application/json;charset=UTF-8
Host: user.api.com
x-api-key: API-KEY

----
```
* `<output-directory>/<document-name>/httpie-request.adoc`
```
[source,bash]
----
$ http GET 'http://user.api.com/user/1' \
    'Accept:application/json;charset=UTF-8' \
    'x-api-key:API-KEY'
----
```
* `<output-directory>/<document-name>/request-body.adoc`
```
[source,http,options="nowrap"]
----
{
  "name": "홍길동"
}
----
```
* `<output-directory>/<document-name>/response-body.adoc`
```
[source,options="nowrap"]
----
{
  "code" : 200,
  "message" : "OK",
  "data" : {
    "id" : 1,
    "name" : "배달이",
    "age" : 30,
    "address" : "서울특별시 송파구 올림픽로 295",
    "roles" : [ ]
  },
  "error" : null
}
----
```



Optional하게 생성되는 snippet도 존재합니다.

* `<output-directory>/<document-name>/path-parameters.adoc.adoc` : Path Parameters에 대한 정보를 표로 나타냅니다.
* `<output-directory>/<document-name>/request-headers.adoc.adoc` : request header에 대한 정보를 표로 나타냅니다.
* `<output-directory>/<document-name>/request-fieldadoc` : request fields에 대한 정보를 표로 나타냅니다.
* `<output-directory>/<document-name>/response-fields.adoc` : response fields에 대한 정보를 표로 나타냅니다.

### API Document 생성하기
```
ifndef::snippets[]
:snippets: ../../../build/generated-snippets
endif::[]
:doctype: book
:icons: font
:source-highlighter: highlightjs
:toc: left
:toclevels: 4
:sectlinks:
:site-url: /build/asciidoc/html5/


== Request

=== [Request URL]
....
GET /user/{userId}
Content-Type: application/json;charset=UTF-8
....

=== [Request Headers]
include::{snippets}/user-search/request-headers.adoc[]

=== [Request Path Parameters]

include::{snippets}/user-search/path-parameters.adoc[]

=== [Request HTTP Example]

include::{snippets}/user-search/http-request.adoc[]

== Response


=== [Response Fields]

include::{snippets}/user-search/response-fields.adoc[]

=== [Response HTTP Example]

include::{snippets}/user-search/http-response.adoc[]
```
* /src/docs/asciidoc 디렉토리를 생성하고 user-search.adoc 파일을 생성합니다.
* 이 파일은 html로 변환될 adoc 파일입니다.
* snippet에 대한 path를 지정하고 `include::` 를 통해 adoc 파일을 include하여 processing 할 수 있습니다.
* include 된 파일은 include한 부분에 html 태그로 직접 삽입됩니다.
* processing 된 html 파일은 `/build/docs/html5` 디렉토리에 생성됩니다 (default)
* 부가적으로 API 문서에 대한 내용을 작성하여 API 문서를 만들 수 있습니다. (ex: API 설명, 담당자, 주의사항)

# HTML 문서 Serving 해보기

Spring Rest Docs로 만들어진 API 문서의 최종 형태는 HTML 파일로 제공됩니다.  
그렇기 때문에 Spring Static Resource Handler를 이용하여 간단하게 html 파일을 Serving하여 웹 브라우져를 통해 사용자에게 API문서를 제공 할 수 있습니다.  

SpringBoot의 WebMvcConfigure를 상속하여 Configuration을 추가합니다.

```kotlin
package com.example.restdocs.configuration

import org.springframework.context.annotation.Configuration
import org.springframework.web.servlet.config.annotation.ResourceHandlerRegistry
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer

@Configuration
open class WebMvcConfig: WebMvcConfigurer {

  override fun addResourceHandlers(registry: ResourceHandlerRegistry) {
    registry.addResourceHandler("/docs/**").addResourceLocations("classpath:/static/docs/")
  }
}
```

위와 같이 설정을 추가하게 되면  
 `localhost:8080/docs/index.html`  과 같이 `/docs/**` 패턴으로 유입되는 url에 대해  
classpath 하위의 /static/docs/ 디렉토리 아래의 파일을 찾아 반환하도록 해줍니다.

![](./static-resource-path.png)

# 여러가지 Function 사용해보기

## PathParameters

* API의 Path Parameter에 대한 Snippet을 생성합니다. 
* /build/generated-snippets/\<document-name>/path-parameters.adoc 인 파일명으로 생성됩니다.
* pathParameter(parameterWithName("key 명칭").description("key가 의미 하는 내용")) 으로 코드를 작성합니다.

```kotlin
resultActions
            .andExpect(status().isOk)
            .andDo(
                    document(
                            "user-update",
                            getDocumentRequest(),
                            getDocumentResponse(),
                            requestHeaders(*header()),
                            pathParameters(parameterWithName("userId").description("유저 ID")),
                            responseFields(*common())
                                    .andWithPrefix("data.", *user())
                                    .andWithPrefix("data.roles[].", *role())
                    )
            )
```



## requestHeader

* API의 Request Header에 대한 Snippet을 생성합니다.
* /build/generated-snippets/\<document-name>/request-headers.adoc 인 파일명으로 생성됩니다.
* requestHeader(headerWithName("header-key명").description("header-key가 의미하는 내용"))

```kotlin
  resultActions
            .andExpect(status().isOk)
            .andDo(
                    document(
                            "user-grant-role",
                            getDocumentRequest(),
                            getDocumentResponse(),
                            requestHeaders(headerWithName("api-auth-key").description("API 인증 키")),
                            pathParameters(userIdPathParameter(), roleIdPathParameter()),
                            responseFields(*common())
                                    .andWithPrefix("data.", *user())
                                    .andWithPrefix("data.roles[].", *role())
                    )
            )
```



## fieldWithPath

* API의 Request, Response Snippet을 구성하는 요소를 정의하는 가장 기본적인 function입니다.
* fieldWithPath("key") 형태로 API Request, Response 내의 요소를 정의 할 수 있습니다.
* Attributes
  * description("내용") : Request, Response 내의 요소의 내용을 정의합니다.
  * type(JsonFieldType.TYPE) : Request, Response 내의 요소의 타입을 정의합니다.
  * optional() : Request, Response 내의 요소의 필수값 여부를 정의합니다. (optional()를 선언하면 필수값 아님)

```kotlin
resultActions
            .andExpect(status().isOk)
            .andDo(
                    document(
                            "user-grant-role",
                            getDocumentRequest(),
                            getDocumentResponse(),
                            requestHeaders(*header()),
                            pathParameters(userIdPathParameter(), roleIdPathParameter()),
                            responseFields(
                            fieldWithPath("code").type(JsonFieldType.NUMBER).description("응답 코드"),
            fieldWithPath("message").type(JsonFieldType.STRING).description("응답 메세지"),
            subsectionWithPath("error").type(JsonFieldType.OBJECT).description("에러 Data").optional(),
            subsectionWithPath("data").type(JsonFieldType.OBJECT).description("응답 Data").optional()
                            )
                                    .andWithPrefix("data.", *user())
                                    .andWithPrefix("data.roles[].", *role())
                    )
            )
```



## subSectionWithPath

* API Request, Response Snippet을 구성하는 요소를 정의하는 function입니다.
* 요소의 하위를 선언하고 싶지 않거나, 가변적인 Request, Response가 오는 경우 사용하기 좋습니다.
* subSectionWithPath는 root key가 존재하는 지만 체크하고 하위 key에 대한 체크는 하지 않습니다.

```Json
{
  "message": "Parameter validation error",
  "code": 4000,
  "errors": [
    {
      "field": "receivers[0].phone",
      "value": "010469090",
      "errorMessage": "유효하지 않은 수신자 번호입니다."
    }
  ]
}
```

위와 같은 응답이 오는 경우

```kotlin
resultActions
            .andExpect(status().isOk)
            .andDo(
                    document(
                            "user-grant-role",
                            getDocumentRequest(),
                            getDocumentResponse(),
                            requestHeaders(*header()),
                            pathParameters(userIdPathParameter(), roleIdPathParameter()),
                            responseFields(*common())
                                    .and(fieldWithPath("errors").type(JsonFieldType.ARRAY).description("에러 Data")) //에러 발생!!
                    )
            )
```

아래와 같은 에러가 발생하게 됩니다.  
에러 내용을 요약하자면, payload에 있는 정보를 문서화 하지 않았다는 에러입니다.  
즉, errors 하위에 field, value, errorMessage 정보를 문서화 하지 않았기 때문에 발생하는 에러입니다.

```json
org.springframework.restdocs.snippet.SnippetException: The following parts of the payload were not documented:
{
  "errors": [
    {
      "field": "receivers[0].phone",
      "value": "010469090",
      "errorMessage": "유효하지 않은 수신자 번호입니다."
    }
  ]
}
```

하지만 subSectionWithPath를 사용하게 되면 errors 하위에 대한 정보는 문서화 하지 않겠다!를 의미하므로  
에러가 발생하지 않고, errors 까지만 문서화 됩니다.

```kotlin
resultActions
            .andExpect(status().isOk)
            .andDo(
                    document(
                            "user-grant-role",
                            getDocumentRequest(),
                            getDocumentResponse(),
                            requestHeaders(*header()),
                            pathParameters(userIdPathParameter(), roleIdPathParameter()),
                            responseFields(*common())
                                    .and(subSectionWithPath("errors").type(JsonFieldType.ARRAY).description("에러 Data")) 
                    )
            )
```



## responseField().and()

* response에 대한 Field정보가 많은 경우 가독성을 해칠 수 있고, Field Snippet에 대한 재사용성이 떨어질 수 있습니다.
* 이럴때 and를 사용하면 좋습니다.
* requestField(), responseField() 함수 뒤에 and() 함수를 사용하여 FieldDescriptor에 대한 Concatination이 가능합니다.

```json
{
  "code": 200,
  "message": "OK",
  "id": 1,
  "name": "배달이",
  "age": 30,
  "address": "서울특별시 송파구 올림픽로 295"
}
```

위와 같이 모든 요소가 1레벨인 Json이 Response로 오는 경우   
code, message 는 모든 API Response에 대한 공통 요소 입니다.  
id, name, age, address는 User API Response에만 응답오는 요소라고 할 떄,  
두 요소에 대한 관리를 별도로 하여 재사용성을 높일 수 있습니다.



아래와 같이 and를 사용하여 Response에 대한 FieldDesciptor를 concatination 하여 구성할 수 있습니다. 

```kotlin
resultActions
            .andExpect(status().isOk)
            .andDo(
                    document(
                            "user-update",
                            getDocumentRequest(),
                            getDocumentResponse(),
                            requestHeaders(*header()),
                            pathParameters(userIdPathParameter()),
                            responseFields(*common())
                                    .and(*user())
                    )
            )
```



## responseField().andWithPrefix()

- and()의 사용예제와 동일합니다.
- 하지만, andWithPrefix는 key 앞에 prefix를 붙여 depth를 표현할 수 있도록 해줍니다.

```json
{
  "code": 200,
  "message": "OK",
  "data" : {
    "id": 1,
  	"name": "배달이",
  	"age": 30,
  	"address": "서울특별시 송파구 올림픽로 295"
  }
}
```

위와 같이 data에 대한 키가 2레벨인 Json이 Response로 오는 경우   
아래와 같이 and를 사용하여 Response에 대한 FieldDesciptor를 concatination 하여 구성할 수 있습니다. 

```kotlin
resultActions
            .andExpect(status().isOk)
            .andDo(
                    document(
                            "user-update",
                            getDocumentRequest(),
                            getDocumentResponse(),
                            requestHeaders(*header()),
                            pathParameters(userIdPathParameter()),
                            responseFields(*common())
                                    .andWithPrefix("data.", *user())
                    )
            )
```



## beneathPath

* API를 사용하다 보면 특정 Field의 명세만 별도로 추출하여 문서화 하는 needs가 있을 수 있습니다.
* beneathPath("data-path") function을 사용하면 특정 data-path에 대한 field를 별도의 snippet으로 생성해 줍니다.

```json
{
  "code": 200,
  "message": "OK",
  "data" : {
    "id": 1,
  	"name": "배달이",
  	"age": 30,
  	"address": "서울특별시 송파구 올림픽로 295"
  }
}
```

```kotlin
resultActions
            .andExpect(status().isOk)
            .andDo(
                    document(
                            "user-search",
                            getDocumentRequest(),
                            getDocumentResponse(),
                            requestHeaders(*header()),
                            pathParameters(userIdPathParameter()),
                            responseFields(
                                    beneathPath("data").withSubsectionId("user"),
                                    *user(),
                                    subsectionWithPath("roles").description("User Role")
                                    )
```
* data 필드 하위의 내용을 문서화 해줍니다.
* beneathPath("data") : data 필드 하위의 내용이라는 Path 지정
* .withSubsectionId("user") : response-fields-user.adoc이라고 SubSectionId로 response-fields.adoc 파일이 별도로 만들어 집니다.



# Spring Rest Docs 추가 기능

## Custom 필드 삽입하기

* Spring Rest Docs에서 기본적으로 제공하는 path, description, optional, type 이외에 다른 메타정보를 추가하고 싶을 때 Custom 필드를 만들어 사용할 수 있습니다.
* AbstractDescriptor에서 제공하는 attributes라는 Map을 이용합니다.
* 저는 비고, 최대길이라는 필드를 표현하기 위해 Kotlin Extension Function을 이용하여 기능을 추가해보았습니다.
* 기본적으로 AbstractDescriptor의 attributes라는 Map에 key, value라는 내용을 넣어주면 됩니다.  
  (Java에서는 이 방식을 사용하세요)

```kotlin
fun <T : AbstractDescriptor<T>> AbstractDescriptor<T>.remarks(remarks: String): T {
  return this.attributes(key("remarks").value(remarks))
}

fun <T : AbstractDescriptor<T>> AbstractDescriptor<T>.maxLength(length: Int): T {
  return this.attributes(key("maxLength").value(length))
}
```

```kotlin
resultActions
            .andExpect(status().isOk)
            .andDo(
                    document(
                            "user-update",
                            getDocumentRequest(),
                            getDocumentResponse(),
                            requestHeaders(*header()),
                            pathParameters(
                              userIdPathParameter()
                              .maxLength(10)
                              .remarks("User ID가 없는 경우는 먼저 생성하세요")),
                            responseFields(*common())
                                    .andWithPrefix("data.", *user())
                                    .andWithPrefix("data.roles[].", *role())
                    )
            )
```

위의 예제에서는 일단 ParameterDescriptor에 maxLength와 remark 속성을 추가 했습니다.  
ParameterDescriptor 뿐만 아니라, AbstractDescriptor를 구현하는 모든 Descriptor에 적용이 가능합니다.  
(FieldDescriptor, SubsectionDescriptor, HeaderDescriptor등...)

이렇게 추가한 속성을 snippet에 출력해보도록 하겠습니다.  
위의 예시는 path parameter에 추가한 속성이므로, path-parameters snippet을 override하여 출력해 줄 수 있습니다.  

### path-parameters snippet override

* /test/org/springframework/restdocs/templates 하위에 path-parameters.snippet 파일을 새로 등록합니다.
* snippet 파일은 mustache 문법을 사용합니다.
* 여기에 제가 추가한 최대길이, 비고 컬럼을 추가하였습니다.
* mustache 문법 중 `#maxLength` 문법은 maxLength 속성이 존재하는가? 라는 if문과 비슷한 문법입니다.
* 반대되는 문법으로 `^maxLength` 문법은 maxLength 속성이 존재하지 않는가? 라는 문법입니다.

```snippet
.+{{path}}+
|===
|Parameter|Description|최대길이|비고

{{#parameters}}
|{{#tableCellContent}}`+{{name}}+`{{/tableCellContent}}
|{{#tableCellContent}}{{description}}{{/tableCellContent}}
|{{#tableCellContent}}{{#maxLength}}{{maxLength}}{{/maxLength}}{{/tableCellContent}}
|{{#tableCellContent}}{{#remarks}}{{remarks}}{{/remarks}}{{/tableCellContent}}
{{/parameters}}
|===
```

### 결과

![add-path-parameter-custom-fields](./add-path-parameter-custom-fields.png)



## Custom Snippet 생성하기

API 문서를 작성하다 보면, 특정 API와 관계없는 응답코드나, Status 값에 대한 명세를 정의하고 문서화 해야 할 요구사항이 있을 수 있습니다.  
그런 경우 Custom Snippet을 생성하여 문서화하면, 별도로 응답코드나 Status가 추가되거나 변경되어도 항상 문서를 최신화 할 수 있습니다.



생성해야 할 것은 4가지 입니다.

1. Test 용 Controller
2. Custom Snippet Template 파일
3. ResponseCodeSnippet 클래스 (AbstractFieldsSnippet 상속)
4. Test Code 



### Test용 Controller

MockMvc 기반에서 생성되는 문서이다 보니 사용하지 않더라도, Test용 Controller가 하나 필요합니다.  
실제 코드에 영향이 없도록 `/test` 디렉토리 하위에 Controller를 만들었습니다.

```kotlin
@RestController
class ResponseCodeController {

  @GetMapping("/response-code")
  fun getResponseCode(): Response<Unit> {
    return Response.success()
  }
}
```



### Custom Snippet Template 파일

```
{{title}}
|===
|Code|Message
{{#fields}}
|{{#tableCellContent}}`+{{path}}+`{{/tableCellContent}}
|{{#tableCellContent}}{{description}}{{/tableCellContent}}
{{/fields}}
|===
```

* title을 변수로 받아서 출력할 예정입니다.
* 표에 출력할 필드는 code값과 description입니다.



### ResponseCodeSnippet 클래스

```kotlin
class ResponseCodeSnippet(
        name: String, 
        descriptors: MutableList<FieldDescriptor>,
        attributes: Map<String, Any>,
        ignoreUndocumentedFields: Boolean
) : AbstractFieldsSnippet(name, descriptors, attributes, ignoreUndocumentedFields) {

  override fun getContentType(operation: Operation): MediaType? {
    return operation.response.headers.contentType
  }

  override fun getContent(operation: Operation): ByteArray {
    return operation.response.content
  }
}
```

* name: 생성할 snippet 템플릿의 prefix입니다. (실제 파일은 {name}-fields.snippet 파일로 생성됩니다.)

![AbstractFieldSnippet-Constructor](./AbstractFieldSnippet-Constructor.png)

* descriptors : snippet에 출력될 field 정보들입니다. (예제에서는 ResponseCode 하나하나가 Row가 됩니다.)
* attributes: snippet에 출력할 추가적인 속성입니다.



### Test Code

```kotlin
@ExtendWith(RestDocumentationExtension::class, SpringExtension::class)
@WebMvcTest(ResponseCodeController::class, secure = false)
@AutoConfigureRestDocs
class ResponseCodeDocs {

  @Autowired
  lateinit var mockMvc: MockMvc

  @Test
  fun `build response code snippet`() {

    mockMvc.perform(get("/response-code")
            .accept(MediaType.APPLICATION_JSON)
    ).andDo(MockMvcResultHandlers.print())
            .andExpect(status().isOk)
            .andDo(
                    document(
                            "common",
                            responseCodeFields(
                                    "response-code", //{name}-fields.snippet 이라는 파일명으로 생성
                                    Attributes.attributes(Attributes.key("title").value("공통 응답 코드")),
                                    *convertResponseCodeToFieldDescriptor(ResponseCode.values())
                            )
                    )
            )
  }

  fun responseCodeFields(
          name: String,
          attributes: Map<String, Any>,
          vararg descriptors: FieldDescriptor
  ): ResponseCodeSnippet {
    return ResponseCodeSnippet(name, mutableListOf(*descriptors), attributes, true)
  }

  private fun convertResponseCodeToFieldDescriptor(enumTypes: Array<ResponseCode>): Array<FieldDescriptor> {
    return Arrays.stream(enumTypes)
            .map {
              fieldWithPath(it.code.toString()).type(JsonFieldType.NUMBER).description(it.message).optional()
            }
            .collect(Collectors.toList())
            .toTypedArray()
  }
}
```



### 결과

![custom-snippet](./custom-snippet.png)



### 추가

위의 예시에서는 Custom Field Snippet으로 예시를 들었지만
* AbstractParametersSnippet
* AbstractBodySnippet
* AbstractHeadersSnippet

위의 AbstractSnippet이 더 있는 걸로 보아 다른 Custom Snippet도 생성할 수 있는 것 같습니다.  
필요할 때마다 한번씩 써보면 좋겠죠?


# Reference

* https://docs.spring.io/spring-restdocs/docs/current/reference/html5/#introduction
* https://asciidoctor.org/docs/asciidoctor-gradle-plugin/
* https://asciidoctor.org/docs/asciidoc-syntax-quick-reference/
* http://woowabros.github.io/experience/2018/12/28/spring-rest-docs.html