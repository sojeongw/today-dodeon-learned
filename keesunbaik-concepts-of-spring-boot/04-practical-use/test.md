# 테스트

- spring-boot-starter-test
    - 의존성 추가 필요

## @SpringBootTest

```java

@RunWith(SpringRunner.class)
@AutoConfigureMockMvc
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.MOCK)
class SampleControllerTest {

    @Autowired
    MockMvc mockMvc;

    @Test
    void hello() throws Exception {
        mockMvc.perform(get("/hello"))
                .andExpect(status().isOk())
                .andExpect(content().string("hello keesun"))
                .andDo(print());
    }
}
``` 

- mock
    - 기본값
    - 내장 톰캣 구현 안함
    - mock up 된 DispatcherServlet을 사용한다.
    - mockMvc를 사용해 테스트를 진행한다.

## RANDOM_PORT, DEFINED_PORT

```java

@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class SampleControllerTest {

    @Autowired
    TestRestTemplate testRestTemplate;

    @Test
    void hello() throws Exception {
        String result = testRestTemplate.getForObject("/", String.class);
        assertThat(result).isEqualTo("hello keesun");
    }
}
```

- 내장 톰캣 사용함

```java

@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class SampleControllerTest {

    @Autowired
    TestRestTemplate testRestTemplate;

    @MockBean
    SampleService sampleService;

    @Test
    void hello() throws Exception {
        when(sampleService.getName()).thenReturn("whiteship");

        String result = testRestTemplate.getForObject("/hello", String.class);
        assertThat(result).isEqualTo("hello whiteship");
    }
}
```

- @MockBean
    - ApplicationContext에 있는 Bean을 Mock 객체로 교체한다.
    - 모든 @Test마다 자동으로 리셋된다.
- SampleController를 MockBean으로 교체하면 MockSmapleService를 쓰게 된다.
- SampleService까지 가지 않고 모킹해서 사용할 수 있는 장점이 있다.

```java

@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class SampleControllerTest {

    @Autowired
    WebTestClient webTestClient;

    @MockBean
    SampleService sampleService;

    @Test
    void hello() throws Exception {
        when(sampleService.getName()).thenReturn("whiteship");

        webTestClient.get().uri("/hello").exchange().expectStatus().isOk()
                .expectBody(String.class).isEqualTo("hello whiteship");
    }
}
```

- RestClient
    - synchronous이기 때문에 요청을 하나 보내면 끝나야 다음 요청을 보낸다.
- WebTestClient
    - asynchronous라서 응답이 오면 우리에게 콜백을 준다.
    - webflux가 있어야 한다.

## NONE

- 서블릿 환경 제공 안함

## 슬라이스 테스트

- @SpringBootTest는 모든 빈을 다 찾아 등록하고 필요한걸 찾아 테스트한다.
- 이때 레이어별로 잘라서 테스트할 수 있다.
- 각 레이어만 등록되기 때문에 다른 것들은 모킹이 필요하다.

### JsonTest

```java

@JsonTest
class SampleControllerTest {

    @Autowired
    JacksonTester<SampleDomain> jacksonTester;

}
```

- 테스트로 나가는 값이 어떤 형태의 json인지 확인한다.
- json 테스트용이므로 빈이 많이 등록되지 않아 테스트가 빨리 돈다.

### @WebMvcTest

```java

@WebMvcTest(SampleController.class)
class SampleControllerTest {
    @MockBean
    SampleService sampleService;

    @Autowired
    MockMvc mockMvc;

    @Test
    void hello() throws Exception {
        when(sampleService.getName()).thenReturn("whiteship");

        mockMvc.perform(get("/hello"))
                .andExpect(status().isOk())
                .andExpect(content().string("hello whiteship"))
                .andDo(print());
    }
}
```

- 컨트롤러 하나만 등록한다.
- 서비스가 아예 빈으로 등록되지 않는다.
    - 웹 계층 아래의 의존성은 다 끊기기 때문에 MockBean으로 등록해줘야 한다.

### @WebFluxTest

### @DataJpaTest