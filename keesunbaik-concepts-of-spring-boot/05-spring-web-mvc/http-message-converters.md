# HttpMessageConverters

```java

@RestController
public class UserController {

    @PostMapping("/user")
    public @ResponseBody User create(@RequestBody User user) {
        return user;
    }
}
```

```text
{“username”:”keesun”, “password”:”123”} <-> User
```

- 스프링 프레임워크에서 제공하는 인터페이스
- http 요청 본문을 객체로, 객체를 http 응답 본문으로 변환할 때 사용한다.
- @RequestBody, @ResponseBody와 사용된다.
- @ResponseBody는 @RestController가 붙어있으면 생략할 수 있다.
    - 없는데 달아주지 않으면 뷰를 찾으려고 시도한다.

```java

@WebMvcTest(UserController.class)
public class UserControllerTest {

    @Test
    void createUser_JSON() throws Exception {
        String userJson = "{\"username\":\"keesun\",\"password\":1234}";
        
        mockMvc.perform(post("/users/create")
                        .contentType(MediaType.APPLICATION_JSON)
                        .accept(MediaType.APPLICATION_JSON)
                        .content(userJson))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.username", is(equalTo("keesun"))))
                .andExpect(jsonPath("$.password", is(equalTo("1234"))));

    }
}
```

- 요청할 때 json으로 들어온다는 것을 확인할 수 있다.