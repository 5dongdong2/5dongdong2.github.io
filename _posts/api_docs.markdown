# API docs

#### test 작성 방법
only web layer test -> @WebMvcTest
- web layer unit test로 Spring rest docs 사용 가능

integration test -> @SpringBootTest(rest assured)
- Spring rest docs 사용 가능
- 통합테스트를 작성해야 API 문서 작성됨

karate
- spring rest docs 가능 여부 확인필요

***
## Spring Rest docs
test code를 기반으로 api docs 생성

> 공식문서: https://docs.spring.io/spring-restdocs/docs/current/reference/htmlsingle/<br>
> https://docs.spring.io/spring-restdocs/docs/1.0.0.BUILD-SNAPSHOT/reference/html5/<br>
> intellij에서 미리보기: https://jojoldu.tistory.com/299<br>

### 적용

> pathVariable 사용 시 : https://blog.naver.com/PostView.naver?blogId=qjawnswkd&logNo=222339541874&categoryNo=41&parentCategoryNo=0&viewDate=&currentPage=1&postListTopCurrentPage=1&from=postView


> Spring REST Docs uses Spring MVC’s test framework, Spring WebFlux’s WebTestClient, or REST Assured to make requests to the service that you are documenting. It then produces documentation snippets for the request and the resulting response.
> https://docs.spring.io/spring-restdocs/docs/current/reference/htmlsingle/#getting-started-documentation-snippets

#### build.gradle

```groovy
plugins {
    ...
    id 'org.asciidoctor.jvm.convert' version '3.3.2' //version이 달라질 수 있음
    ...
}

ext {
    set('snippetsDir', file("build/generated-snippets"))
}

dependencies {
    ...
    testImplementation 'org.springframework.restdocs:spring-restdocs-mockmvc'
    ...
}

test {
    ...
    outputs.dir snippetsDir
    ...
}

asciidoctor {
    dependsOn test
    inputs.dir snippetsDir
}

task copyApiDocs(type: Copy) {
    copy {
        from asciidoctor.outputDir
        into 'src/main/resources/static/docs'
    }
}

bootJar {
    dependsOn asciidoctor, copyApiDocs
}

build {
    dependsOn asciidoctor, copyApiDocs
}

task apiDocsClean(type: Delete) {
    delete file('src/main/resources/static/docs')
}
clean.dependsOn apiDocsClean
```

#### src/docs/asciidoc/api-guide.adoc

```adoc
= Spring REST Docs
:toc: left
:toclevels: 2
:sectlinks:

ifndef::snippets[]
:snippets: ./build/generated-snippets
endif::[]

[[resources-get]]
== GET

[[resources-get-test]]
=== GET

==== HTTP request

include::{snippets}/조회/http-request.adoc[]

==== HTTP response

include::{snippets}/조회/http-response.adoc[]

[[resources-post]]
== POST

include::{snippets}/put/http-request.adoc[]

[[resources-post-test]]
=== POST

include::{snippets}/조회/http-response.adoc[]

//https://docs.asciidoctor.org/asciidoctor/latest/
//https://narusas.github.io/2018/03/21/Asciidoc-basic.html
```

### 출력 예시
> https://techblog.woowahan.com/2597/

***
## Swagger
springfox-swagger는 2020년부터 업데이트가 되지 않고있다.

https://happy-jjang-a.tistory.com/164

### 적용
build.gradle #
```groovy
dependencies {
    ...
    //swagger의 구현체 springfox
    implementation "io.springfox:springfox-boot-starter:3.0.0"
    ...
}
```
application.yml #
```yml
spring:
  mvc:
    pathmatch:
      matching-strategy: ant_path_matcher
```