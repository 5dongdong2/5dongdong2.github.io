---
layout: post
title: Integration Test(spring, rest-assured, karate)
date: 2023-04-17
categories: [spring, test]
tags: [java, spring framework, rest assured, karate]
---
## Spring
- spring에서 지원하는 TestRestTemplate을 사용

### test code - spring
- @SpringBootTest를 선언하면 TestRestTemplate의 의존성이 자동주입된다.

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class IntegrationTestControllerTest {

	@Value(value = "${local.server.port}")
	private int port;

	@Autowired
	private TestRestTemplate restTemplate;

	@Test
 	void POST_GET_PUT_DELETE() {
		Long id = 1L;
		MessageBodyDTO requestBody = MessageBodyDTO.builder()
			.description("등록합니다.")
			.build();

		//POST
		restTemplate.postForObject("http://localhost:" + port + "/test/api/post", requestBody, Void.class);

		//GET
		ResponseEntity<MessageBodyDTO> responseAfterCreate = restTemplate.getForEntity(
			StringUtils.replace("http://localhost:" + port + "/test/api/get/{id}", "{id}", id.toString()), MessageBodyDTO.class);

		assertThat(responseAfterCreate.getBody().getId()).isEqualTo(id);
		assertThat(responseAfterCreate.getBody().getDescription()).isEqualTo("등록합니다.");

		//PUT
		MessageBodyDTO requestBodyAfterModifying = MessageBodyDTO.builder()
			.id(id)
			.description("수정합니다.")
			.build();
		restTemplate.put(StringUtils.replace("http://localhost:" + port + "/test/api/put/{id}", "{id}", id.toString()), requestBodyAfterModifying);

		ResponseEntity<MessageBodyDTO> responseAfterModifying = restTemplate.getForEntity(
			StringUtils.replace("http://localhost:" + port + "/test/api/get/{id}", "{id}", id.toString()), MessageBodyDTO.class);

		assertThat(responseAfterModifying.getBody().getId()).isEqualTo(id);
		assertThat(responseAfterModifying.getBody().getDescription()).isEqualTo("수정합니다.");

		//DELETE
		restTemplate.delete(StringUtils.replace("http://localhost:" + port + "/test/api/delete/{id}", "{id}", id.toString()));

		ResponseEntity<MessageBodyDTO> responseAfterDelete = restTemplate.getForEntity(
			StringUtils.replace("http://localhost:" + port + "/test/api/get/{id}", "{id}", id.toString()), MessageBodyDTO.class);

		assertThat(responseAfterDelete.getBody().getDescription()).isNull();
	}
}
```

***
## rest-assured
- spring rest docs와 함께 사용해서 API 문서 작성 가능
- mock test, integration test 가능

> 공식 github: https://github.com/rest-assured/rest-assured/wiki/Usage

### test code - rest assured
```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class IntegrationTestControllerTest {

	@Value(value = "${local.server.port}")
	private int port;

	@Autowired
	private ObjectMapper objectMapper;

	@Test
	void POST_GET_PUT_DELETE() {
		//POST
		given()
			.port(port)
			.contentType(MediaType.APPLICATION_JSON_VALUE)
			.body(MessageBodyDTO.builder()
				.description("등록하겠습니다.")
				.build())
			.post("/test/api/post")
		.then()
			.statusCode(200);

		//GET
		given()
			.port(port)
			.get("/test/api/get/{id}", 1)
		.then()
			.body("id", equalTo(1))
			.body("description", equalTo("등록하겠습니다."));

		//PUT
		given()
			.port(port)
			.contentType(MediaType.APPLICATION_JSON_VALUE)
			.body(MessageBodyDTO.builder()
				.id(1L)
				.description("수정하겠습니다.")
				.build())
			.put("/test/api/put/{id}", 1)
		.then()
			.statusCode(200);

		given()
			.port(port)
			.get("/test/api/get/{id}", 1)
		.then()
			.body("id", equalTo(1))
			.body("description", equalTo("수정하겠습니다."));

		//DELETE
		given()
			.port(port)
			.contentType(MediaType.APPLICATION_JSON_VALUE)
			.delete("/test/api/delete/{id}", 1)
		.then()
			.statusCode(200);

		given()
			.port(port)
			.get("/test/api/get/{id}", 1)
		.then()
			.body("id", equalTo(1))
			.body("description", nullValue());
	}
}
```

***
## karate
- mock test, integration test, e2e test, gatling test 등 높은 확장성
- parallel test 가능
- code coverage 가능
- karate 자체만으로 가능, junit5와 함께 사용가능

> 공식 github: https://github.com/karatelabs/karate#http-basic-authentication-example
> 보기편한 README.md: https://karatelabs.github.io/karate/

### test code - karate
```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.DEFINED_PORT)
public class IntegrationTestControllerRunner {

	@Karate.Test
	Karate testPostGetPutDelete() {
		return Karate.run("post_get_put_delete").relativeTo(getClass());
	}
    
}
```

```karate
Feature: post_get_put_delete

  Scenario:
    * configure url = 'http://localhost:8080'

    Given path '/test/api/post'
    And request { id:  null, description:  '등록합니다.'}
    When method post
    Then status 200

    Given path '/test/api/get/1'
    When method get
    Then status 200
    And assert response.id == 1
    And assert response.description == '등록합니다.'

    Given path '/test/api/put/1'
    And request { id:  1, description:  '수정합니다.'}
    When method put
    Then status 200

    Given path '/test/api/get/1'
    When method get
    Then status 200
    And assert response.id == 1
    And assert response.description == '수정합니다.'

    Given path '/test/api/delete/1'
    When method delete
    Then status 200

    Given path '/test/api/get/1'
    When method get
    Then status 200
    And assert response.id == 1
    And assert response.description == null
```

<details>
    <summary>참고한 글</summary>
    
> build.gradle: https://github.com/karatelabs/karate/wiki/Gradle <br>
> directory and naming convention: https://github.com/karatelabs/karate#naming-conventions <br>
> karate with junit5: https://github.com/karatelabs/karate#junit-5 <br>
> generate report: https://github.com/karatelabs/karate#junit-html-report <br>
> parallel execution: https://github.com/karatelabs/karate#junit-5-parallel-execution <br>
> test report: https://github.com/karatelabs/karate#test-reports <br>
> configuration(karate-config.js): https://github.com/karatelabs/karate#configuration <br>
> syntax guide: https://github.com/karatelabs/karate#syntax-guide <br>
> path prefix: https://github.com/karatelabs/karate#path-prefixes <br>
> core keywords(url, path, request, method, status): https://github.com/karatelabs/karate#core-keywords <br>
> configure: https://github.com/karatelabs/karate#configure <br>
> assertions: https://github.com/karatelabs/karate#payload-assertions <br>
> validation: https://github.com/karatelabs/karate#fuzzy-matching <br>
> karate object: https://github.com/karatelabs/karate#the-karate-object <br>
> http authentication(spring security 가능?): https://github.com/karatelabs/karate#http-basic-authentication-example <br>
> async: https://github.com/karatelabs/karate#async <br>
> hooks: https://github.com/karatelabs/karate#hooks <br>

</details>