# 뷰 리졸버

{% tabs %} {% tab title="OldController.java" %}

```java

@Component("/springmvc/old-controller")
public class OldController implements Controller {
    @Override
    public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception {
        System.out.println("OldController.handleRequest");

        // 논리 이름으로 호출한다.
        return new ModelAndView("new-form");
    }
}
```

{% endtab %} {% endtabs %}

view를 사용하도록 코드를 추가했지만 컨트롤러를 호출하면 에러 페이지가 노출된다.

## InternalResourceViewResolver

{% tabs %} {% tab title="application.properties" %}

```properties
logging.level.org.apache.coyote.http11=debug
spring.mvc.view.prefix=/WEB-INF/view
spring.mvc.view.suffix=.jsp
```

{% endtab %} {% endtabs %}

prefix, suffix를 설정하면 정상적으로 뷰가 출력된다.

- 스프링 부트는 InternalResourceViewResolver로 뷰 리졸버를 자동 등록한다.
- application.properties에 등록한 prefix, suffix 설정으로 등록한다.

```java
public class MainApplication {
    @Bean
    ViewResolver internalResourceViewResolver() {
        return new InternalResourceViewResolver("/WEB-INF/views/", ".jsp");
    }
}
```

내부적으로는 이렇게 자동으로 만들어서 동작한다고 보면 된다.

![](../../.gitbook/assets/kimyounghan-spring-mvc/05/screenshot%202022-02-13%20오전%2011.40.25.png)

- ViewResolver
    - 1순위: BeanNameViewResolver
        - 빈 이름으로 뷰를 찾아서 반환한다.
        - ex. 엑셀 파일 생성 기능
        - 우리는 new-form이라는 스프링 빈이 있는 게 아니므로 해당하지 않는다.
    - 2순위: InternalResourceViewResolver
        - JSP를 처리할 수 있는 뷰를 반환한다.

## 실행 과정

1. 핸들러 어댑터 호출
    1. 핸들러 어댑터를 통해 new-form이라는 논리 뷰 이름을 획득한다.
2. ViewResolver 호출
    1. new-form이라는 뷰 이름으로 viewResolver를 순서대로 호출한다.
    2. BeanNameViewResolver는 new-form이라는 스프링 빈을 찾아야 하는데 없다.
    3. InternalResourceViewResolver가 호출된다.
3. InternalResourceViewResolver
    1. InternalResourceView를 반환한다.
        - 뷰가 인터페이스화 되어 있어서 InternalResourceView를 반환하게 된다.
4. InternalResourceView
    1. forward()가 있어서 JSP 등에 사용한다.
5. view.render()
    1. view.render()가 호출되고 InternalResourceView는 forward()를 사용해 JSP를 실행한다.

## 참고

- InternalResourceViewResolver는 JSTL 라이브러리를 사용하고 있으면 InternalResourceView를 상속받은 JstlView를 반환한다.
    - JSTL 사용 시 약간의 부가 기능이 추가된다.
- 다른 뷰는 실제 뷰를 렌더링하지만 JSP는 forward()를 통해 JSP로 이동해야 렌더링이 된다.
    - JSP를 제외한 나머지 뷰 템플릿은 forward() 없이 바로 렌더링 된다.
- 타임리프 뷰 템플릿을 사용하면 ThymeleafViewResolver를 등록해야 한다.
    - 스프링 부트가 자동화해준다.