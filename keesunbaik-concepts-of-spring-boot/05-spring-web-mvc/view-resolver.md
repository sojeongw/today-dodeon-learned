# ViewResolver

- HttpMessageConvertersAutoConfiguration
- 뷰 리졸버 설정을 제공한다.
- accept header에서 원하는 타입에 따라 응답 본문을 다르게 만든다.
    - 제공하지 않는 경우 format이라는 매개 변수를 사용해 원하는 타입을 넘겨준다.

{% tabs %} {% tab title=".java" %}

```java

@WebMvcTest(UserController.class)
public class UserControllerTest {

    @Test
    void createUser_XML() throws Exception {
        String userJson = "{\"username\":\"keesun\",\"password\":1234}";

        mockMvc.perform(post("/users/create")
                        .contentType(MediaType.APPLICATION_JSON)
                        .accept(MediaType.APPLICATION_XML)
                        .content(userJson))
                .andExpect(status().isOk())
                .andExpect(xpath("/User/username").string("keesun"))
                .andExpect(xpath("/User/password").string("1234"));

    }
}
```

{% endtab %} {% tab title="pom.xml" %}

```xml

<dependency>
    <groupId>com.fasterxml.jackson.dataformat</groupId>
    <artifactId>jackson-dataformat-xml</artifactId>
    <version>2.9.6</version>
</dependency>

```

{% endtab %} {% endtabs %}

- 다른 내용은 변경없이 accept만 바꿨는데 그 형태로 응답이 오는 걸 확인할 수 있다.
    - 이때 xml 컨버팅 하는 의존성 추가가 필요하다.