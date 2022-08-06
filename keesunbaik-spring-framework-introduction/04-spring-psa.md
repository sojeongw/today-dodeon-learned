# PSA(Portable Service Abstraction)

PSA는 잘 만든 인터페이스를 의미한다. 이게 무슨 의미일까? 

## 스프링 웹 MVC

예로, 우리는 서블릿 애플리케이션을 만들고 있음에도 서블릿을 전혀 쓰지 않는다. 

```java
public class TestServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) {
        ...
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) {
        ...
    }
}
```

이렇게 서블릿을 만들고 매핑하지 않는다는 것이다.

```java
// 요청을 처리하는 빈으로 등록된다.
@Controller
class OwnerController {
    // 아래와 정확히 매핑될 때만 처리한다.
    @GetMapping("/owners/new")
	public String initCreationForm(Map<String, Object> model) {
		...
	}

	@PostMapping("/owners/new")
    public String processCreationForm(@Valid Owner owner, BindingResult result) {
		...
	}
}
```

단지 이렇게 간단한 애너테이션만으로 요청을 처리한다. 스프링 뒷단에서 여러 기능을 제공하고 있기 때문이다. 서비스 추상화는 이렇듯 편의성을 제공하기 위한 용도로 사용된다. 

또한, 코드를 거의 바꾸지 않고도 마음대로 기술을 바꿔가며 쓸 수 있게 해준다. 예를 들어, 스프링 부트는 톰캣 기반으로 돌아가지만 WebFlux를 쓰기 위해 네티 기반으로 바꿀 수도 있다.

## 스프링 트랜잭션

`setAutoCommit(false)` 후에 `commit()`이 실행될 때까지 모든 SQL이 적용하도록 만드는 일련의 과정을 `@Transactional` 애너테이션이 처리해준다.

JPA를 쓰든, Hibernate를 쓰든 코드 변경 없이 자유롭게 옮겨다닐 수도 있다. 그래서 Portable한 서비스 추상화인 것이다.

**Reference**

[Service Abstraction](https://en.wikipedia.org/wiki/Service_abstraction)