# 스프링 예제 프로젝트 PetClinic

## 프로젝트 세팅

```shell script
./mvnw package   # 프로젝트 빌드
java -jar target/*.jar    # 프로젝트 실행
```

Maven의 `package`라는 명령어를 실행하면 빌드해서 패키지 파일을 만든다. 설정 파일에 아무것도 하지 않았으면 기본적으로 `jar`로 생성된다.

## 또 다른 실행 방법

스프링 부트 기반이기 때문에 `src/main/java`에 있는 `PetClinicApplication`을 실행하면 된다. 웹이지만 내장 톰캣이 있기 때문에 자바 애플리케이션으로 설정 없이 실행할 수 있다.

이때 반드시 패키징을 먼저 해놔야 한다. 패키징 하는 과정에서 프론트 관련 라이브러리를 생성하는 플러그인이 동작하기 때문이다. 그래서 패키징을 하지 않으면 화면이 깨져 나온다.

## 프로젝트 구조

일반적인 메이븐 프로젝트 구조로 되어있다.

```text
src/main/java
src/main/resources
src/test/java
src/test/resources
```

## 스프링 부트

스프링 부트를 기반으로 프로젝트를 하면 아래와 같이 사용한다.

```text
스프링 부트
스프링 데이터 JPA
DB: HSQLDB
뷰: 타임리프
캐시: EHCache
```

## 코드의 흐름을 아는 법
### 로그 분석

application.properties에서 다음의 코드를 추가한다.

```properties
# Logging
logging.level.org.springframework=INFO
logging.level.org.springframework.web=DEBUG     # 이 부분을 활성화 한다
# logging.level.org.springframework.context.annotation=TRACE
```

그럼 이런 로그가 찍힌다.

```text
2020-01-17 22:39:34.859  INFO 62253 --- [(1)-172.30.1.18] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring DispatcherServlet 'dispatcherServlet'
2020-01-17 22:39:34.860  INFO 62253 --- [(1)-172.30.1.18] o.s.web.servlet.DispatcherServlet        : Initializing Servlet 'dispatcherServlet'
2020-01-17 22:39:34.860 DEBUG 62253 --- [(1)-172.30.1.18] o.s.web.servlet.DispatcherServlet        : Detected StandardServletMultipartResolver
2020-01-17 22:39:34.868 DEBUG 62253 --- [(1)-172.30.1.18] o.s.web.servlet.DispatcherServlet        : enableLoggingRequestDetails='false': request parameters and headers will be masked to prevent unsafe logging of potentially sensitive data
2020-01-17 22:39:34.869  INFO 62253 --- [(1)-172.30.1.18] o.s.web.servlet.DispatcherServlet        : Completed initialization in 9 ms
2020-01-17 22:39:39.283 DEBUG 62253 --- [nio-8080-exec-1] o.s.web.servlet.DispatcherServlet        : GET "/", parameters={}
2020-01-17 22:39:39.289 DEBUG 62253 --- [nio-8080-exec-1] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped to org.springframework.samples.petclinic.system.WelcomeController#welcome()
2020-01-17 22:39:39.310 DEBUG 62253 --- [nio-8080-exec-1] o.s.w.s.v.ContentNegotiatingViewResolver : Selected 'text/html' given [text/html, application/xhtml+xml, image/webp, image/apng, application/signed-exchange;v=b3, application/xml;q=0.9, */*;q=0.8]
2020-01-17 22:39:39.572 DEBUG 62253 --- [nio-8080-exec-1] o.s.web.servlet.DispatcherServlet        : Completed 200 OK
2020-01-17 22:39:40.980 DEBUG 62253 --- [nio-8080-exec-2] o.s.web.servlet.DispatcherServlet        : GET "/owners/find", parameters={}
2020-01-17 22:39:40.982 DEBUG 62253 --- [nio-8080-exec-2] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped to org.springframework.samples.petclinic.owner.OwnerController#initFindForm(Map)
2020-01-17 22:39:41.000 DEBUG 62253 --- [nio-8080-exec-2] o.s.w.s.v.ContentNegotiatingViewResolver : Selected 'text/html' given [text/html, application/xhtml+xml, image/webp, image/apng, application/signed-exchange;v=b3, application/xml;q=0.9, */*;q=0.8]
2020-01-17 22:39:41.043 DEBUG 62253 --- [nio-8080-exec-2] o.s.web.servlet.DispatcherServlet        : Completed 200 OK
```

DispatcherServlet 로그는 요청한 맨 처음에만 나타난다. 서블릿에 대한 자세한 내용은 Spring Framework MVC 강좌를 참고하면 된다.

요청을 하면 서블릿이 컨트롤러에서 매핑(OwnerController)된 내용을 보고 해당 메소드(initFindForm)를 찾아 뷰를 리턴하는 내용을 볼 수 있다.

### 디버그 모드

자바 코드에서 빨간 원으로 체크해두고 디버그 모드로 실행하면 해당 부분을 지날 때 코드에 딱 걸리게 된다.