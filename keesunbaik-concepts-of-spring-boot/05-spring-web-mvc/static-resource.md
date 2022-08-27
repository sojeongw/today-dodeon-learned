# 정적 리소스 지원

## 기본 리소스 위치

- classpath:/static
- classpath:/public
- classpath:/resources/
- classpath:/META-INF/resources

이 4가지는 기본적으로 `/**`으로 매핑된다.

- ex. `/hello.html` => `/static/hello.html`

예를 들면 `hello.html`를 요청할 때 `/static`에 있으면 해당 파일을 보내준다.

### 옵션

- spring.mvc.static-path-pattern
    - 맵핑 설정 변경
- spring.mvc.static-locations
    - 리소스 찾을 위치 변경

## ResourceHttpRequestHandler

- 정적 리소스를 처리해주는 핸들러
- 파일이 변경되지 않으면 Last-Modified 헤더를 보고 304를 내리기도 한다.
    - 요청된 리소스를 재전송할 필요가 없음을 나타낸다.
    - 캐시된 자원으로의 암묵적인 리디렉션이다.

```java

@Component
public class WebConfig implements WebMvcConfigurer {
    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/m/**")
                .addResourceLocations("classpath:/m/")  // 반드시 /로 끝나야 한다.
                .setCachePeriod(20);
    }
}
```

- WebMvcConfigurer의 addRersourceHandlers로 커스터마이징 할 수 있다.
- `/m/`으로 요청이 오면 해당 디렉터리에 있는 파일을 받을 수 있게 된다.