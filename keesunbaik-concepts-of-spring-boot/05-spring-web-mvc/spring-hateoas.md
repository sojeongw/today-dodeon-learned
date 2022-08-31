# Spring HATEOAS

- HATEOAS를 지원하는 스프링 기능
- spring-boot-starter-hateoas 의존성을 추가한다.
    - ObjectMapper와 LinkDiscovers를 제공한다.

## HATEOAS

- Hypermedia As The Engine Of Application State
- 서버
    - 요청된 정보를 제공할 때 리소스와 연관된 링크 정보까지 같이 제공한다
- 클라이언트
    - 연관된 링크 정보를 바탕으로 리소스에 접근한다.

### 연관된 링크 정보

- Relation
- Hypertext Reference

book 조회 링크가 있다면 relation에 book이 있고 그 HREF는 /book이 된다. book을 요청하면 book이라는 relation의 HREF를 요청한다.

## ObjectMapper

- spring.jackson.*
- json으로 변환할 때 사용한다.
- starter 의존성에 이미 들어 있기 때문에 hateoas가 없어도 이미 등록되어 있어 사용하면 된다.

## 예제

{% tabs %} {% tab title="SampleController.java" %}

```java

@RestController
public class SampleController {

    @GetMapping("/hateoas")
    public EntityModel<Hello> hello() {
        Hello hello = new Hello();
        hello.setPrefix("Hey, ");
        hello.setName("keesun");

        // 링크 추가
        EntityModel<Hello> helloResource = EntityModel.of(hello);
        helloResource.add(linkTo(methodOn(SampleController.class).hello()).withSelfRel());

        return helloResource;
    }
}
```

{% endtab %} {% tab title="SampleControllerTest.java" %}

```java

@WebMvcTest(SampleController.class)
class SampleControllerTest {

    @Autowired
    MockMvc mockMvc;

    @Test
    void hello() throws Exception {
        mockMvc.perform(get("/hateoas"))
                .andDo(print())
                .andExpect(status().isOk())
                // 링크 정보 확인
                .andExpect(jsonPath("$._links.self").exists());
    }
}
```

{% endtab %} {% endtabs %}

```json
{
  "prefix": "Hey, ",
  "name": "keesun",
  "_links": {
    "self": {
      "href": "http://localhost/hateoas"
    }
  }
}
```

- 결과에 링크 정보가 추가되는 걸 볼 수 있다.