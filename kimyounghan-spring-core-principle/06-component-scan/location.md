# 탐색 위치와 기본 스캔 대상

```java

@Configuration
@ComponentScan(
    // 스캔할 기준이 되는 위치를 설정할 수 있다.
    basePackages = "hello.core"
)
public class AutoAppConfig {

}
```

모든 자바 클래스를 다 컴포넌트 스캔하면 시간이 오래 걸린다. 그래서 꼭 필요한 위치부터 탐색하도록 시작 위치를 지정할 수 있다.

만약 지정하지 않으면 `ComponentScan`이 붙은 설정 정보 클래스의 패키지가 시작 위치가 된다. 기본적으로는 패키지 위치를 지정하지 않고 설정 정보 클래스를 프로젝트
최상단에 두는 것을 권장한다.

```text
com.hello
com.hello.service
com.hello.repository
```

이런 구조라면 프로젝트 시작 루트인 `com.hello`에 메인 설정 정보를 둬서 `@ComponentScan` 애너테이션을 붙이고 `basePackages` 지정은 생략한다.
이렇게 하면 `com.hello`를 포함한 하위는 모두 컴포넌트 스캔의 대상이 된다.

프로젝트 메인 설정 정보는 프로젝트를 대표하는 정보이기 때문에 시작 루트에 두는 것이 좋다.

스프링 부트를 사용하면 스트링 부트의 대표 시작 정보인 `@SpringBootApplication`을 프로젝트 루트에 두는 것이 관례다. 이 애너테이션
안에 `@ComponentScan`이 들어있다.

## 컴포넌트 스캔 기본 대상

컴포넌트 스캔은 `@Component`뿐만 아니라 아래의 애너테이션도 대상에 포함한다.

- @Component
    - 컴포넌트 스캔에서 사용
- @Controller
    - 스프링 MVC 컨트롤러에서 사용
    - 스프링 MVC 컨트롤러로 인식한다.
- @Service
    - 스프링 비즈니스 로직에서 사용
    - 특별한 처리를 하는 것은 아니다. 개발자들이 '핵심 비즈니스 로직이 여기에 있겠구나' 라고 비즈니스 계층을 인식하는데 도움이 된다.
- @Repository
    - 스프링 데이터 접근 계층에서 사용
    - 스프링 데이터 접근 계층으로 인식하고 데이터 계층의 예외를 스프링의 추상화된 예외로 변환해준다.
    - DB 종류가 바뀌면 예외 자체도 바뀌어버리기 때문에 예외를 받는 서비스 계층에서 변경 사항이 생기지 않도록 추상화된 예외를 사용하는 것이다.
- @Configuration
    - 스프링 설정 정보에서 사용
    - 스프링 빈이 싱글턴을 유지하도록 추가 처리를 한다.

```java

@Component
public @interface Controller {

}

@Component
public @interface Service {

}

@Component
public @interface Configuration {

}
```

실제로 해당 애너테이션을 제공하는 클래스 소스 코드를 보면 `@Component`를 포함하고 있다.

애너테이션에는 상속 관계라는 것이 없다. 이렇게 애너테이션을 붙여줬다고 해서 언어적으로 인식하는 것은 아니다. 애너테이션에 애너테이션을 붙였다고 연동되는 기능이 전혀 없다. 이런 것은 자바가 아니라 모두 스프링이 지원하는 기능이다.

`userDefaultFilters` 옵션은 기본으로 켜져있는데, 이 옵션을 끄면 기본 스캔 대상들이 제외된다. 그냥 이런 옵션이 있다는 것만 알고 넘어가자.