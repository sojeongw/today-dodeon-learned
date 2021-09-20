# 요청 매핑 핸들러 어댑터

![](../../.gitbook/assets/kimyounghan-spring-mvc/06/screenshot%202022-03-01%20오후%209.44.34.png)

HTTP 메시지 컨버터는 4번에서 handler를 호출하는 RequestMappingHandlerAdapter(요청 매핑 핸들러 어댑터)와 관련있다.

## RequestMappingHandlerAdapter 동작 방식

![](../../.gitbook/assets/kimyounghan-spring-mvc/06/screenshot%202022-03-01%20오후%209.48.14.png)

### ArgumentResolver

```java

@Slf4j
@Controller
public class RequestBodyStringController {

    @PostMapping
    public void requestBodyString(HttpServletRequest request, HttpServletResponse response) {
        ...
    }

    @PostMapping
    public HttpEntity<String> requestBodyStringV3(HttpEntity<String> httpEntity) {
        ...
    }

    @ResponseBody
    @PostMapping
    public String requestBodyStringV4(@RequestBody String messageBody) {
        ...
    }
}
```

- 컨트롤러 메서드에는 파라미터, 애너테이션 등등 많은 정보가 있다. 이 정보를 ArgumentResolver가 처리해준다.
- RequestMappingHandlerAdapter는 이 ArgumentResolver를 호출해 컨트롤러(핸들러)에게 필요한 파라미터 값을 생성한다.
- 파라미터 값이 모두 준비되면 컨트롤러를 호출하면서 값을 넘겨준다.

![](../../.gitbook/assets/kimyounghan-spring-mvc/06/screenshot%202022-03-01%20오후%209.56.57.png)

- ArguemntResolver는 정확히 말하면 HandlerMethodArgumentResolver라고 한다.
- supportsParameter()
    - 파라미터 정보를 보고 이 값을 지원하는지 확인한다.
- resolverArgument()
    - 지원한다면 객체를 만들어서 반환한다.
    - 리턴 타입을 보면 Object인 것을 알 수 있다.
- 원한다면 이 인터페이스를 직접 확장해서 커스텀할 수 있다.

**Reference**

[가능한 파라미터 목록](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-arguments)

### ReturnValueHandler

![](../../.gitbook/assets/kimyounghan-spring-mvc/06/screenshot%202022-03-01%20오후%2010.00.29.png)

- HandlerMethodReturnValueHandler를 줄여서 부르는 명칭
- ReturnValueHandler 덕분에 컨트롤러에서 String, ModelAndView, @ResponseBody 등 다양한 값을 반환할 수 있다.

**Reference**

[가능한 응답 값 목록](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-return-types)

## HTTP 메시지 컨버터

![](../../.gitbook/assets/kimyounghan-spring-mvc/06/screenshot%202022-03-01%20오후%2010.04.53.png)

- 메시지 컨버터는 ArgumentResolver와 ReturnValueHandler에 있다.
- @RequestBody, @ResponseBody
    - RequestResponseBodyMethodProcessor라는 ArgumentResolver가 처리한다.
- HttpEntity
    - HttpEntityMethodProcessor라는 ArgumentResolver가 처리한다.

### 요청

- @RequestBody, HttpEntity 등을 처리하는 ArgumentResolver가 있다.
- 이 ArgumentResolver가 HTTP 메시지 컨버터를 이용해 필요한 객체를 생성한다.

### 응답

- @ResponseBody, HttpEntity 등을 처리하는 ReturnValueHandler가 있다.
- 이 ArgumentResolver가 HTTP 메시지 컨버터를 호출해 응답 결과를 만든다.

![](../../.gitbook/assets/kimyounghan-spring-mvc/06/screenshot%202022-03-01%20오후%2010.15.57.png)

- ArgumentResolver에서 메시지 컨버터를 호출하는 걸 볼 수 있다.

## 확장

- HandlerMethodArgumentResolver
- HandlerMethodReturnValueHandler
- HttpMessageConverter

스프링은 이 세 가지를 모두 인터페이스로 제공하기 때문에 필요하면 언제든 확장할 수 있다. 스프링이 대부분을 지원하기 때문에 확장할 일은 거의 없긴 하다.

```java
class Example {

    @Bean
    public WebMvcConfigurer webMvcConfigurer() {
        return new WebMvcConfigurer() {
            @Override
            public void addArgumentResolvers(List<HandlerMethodArgumentResolver> resolvers) {
              ...
            }

            @Override
            public void extendMessageConverters(List<HttpMessageConverter<?>> converters) {
            ...
            }
        };
    }
}
```

만약 필요하다면 WebMvcConfigurer를 상속받아서 스프링 빈으로 등록하면 된다.