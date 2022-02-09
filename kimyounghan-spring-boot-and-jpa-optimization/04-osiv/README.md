# OSIV와 성능 최적화

- JPA에 엔티티 매니저가 있다면 하이버네이트에는 세션이 있다.
- 하이버네이트
    - Open Session In View
- JPA
    - Open EntityManager In View
    - 관례상 OSIV라고 부른다.

## OSIV ON

- spring.jpa.open-in-view: true

스프링에서는 OSIV가 기본값으로 설정되어있다.

![](../../.gitbook/assets/kimyounghan-spring-boot-and-jpa-optimization/04/screenshot%202021-06-05%20오후%206.15.57.png)

스프링을 실행하면 warn 로그가 하나 남는다. OSIV가 DB 커넥션과 관련된 문제로 보인다.

![](../../.gitbook/assets/kimyounghan-spring-boot-and-jpa-optimization/04/screenshot%202021-06-05%20오후%206.10.31.png)

언제 JPA가 DB 커넥션을 가지고 왔다가 끝을 낼까? JPA의 영속성 컨텍스트는 결국 DB를 1:1로 쓰면서 동작한다.

이때 기본적으로는 트랜잭션을 시작할 때 영속성 컨텍스트가 DB 커넥션을 가져온다. 커넥션을 획득한 후에는 API 응답이 끝날때까지 유지한다. 트랜잭션이 끝나도 지연 로딩으로 프록시 객체를 초기화할 상황이 생기기
때문이다. 따라서 영속성 컨텍스트가 DB 커넥션을 계속 물고 있어야 한다.

따라서 API라면 결과가 유저에게 반환될 때까지, 화면이라면 뷰 템플릿으로 렌더링할 때까지는 유지한다. 더 이상 response를 만들 필요가 없을 때까지 가지고 있는다.

하지만 이 전략엔 치명적인 단점이 있다. 너무 오랫동안 DB 커넥션을 물고 있기 때문에 실시간 트래픽이 중요한 애플리케이션에서는 커넥션이 모자랄 수 있다. 이는 결국 장애로 이어진다.

예를 들어 컨트롤러가 외부 API를 호출하면 호출하는 대기 시간만큼 커넥션 리소스를 반환하지 못하고 유지해야 한다. 외부 API가 3초 걸리면 DB도 그동안 연결되어 있어야 한다. 스레드가 다 찰 때까지 DB
커넥션을 먹고 있을 수 있는 것이다.

장점은 지연 로딩을 적극적으로 활용할 수 있다는 것이다.

## OSIV OFF

- spring.jpa.open-in-view: false

OSIV를 끄면 트랜잭션을 종료할 때 영속성 컨텍스트를 닫고 DB 커넥션을 반환한다. 따라서 커넥션 리소스를 낭비하지 않는다.

![](../../.gitbook/assets/kimyounghan-spring-boot-and-jpa-optimization/04/screenshot%202021-06-05%20오후%207.14.26.png)

```java
// 반환
Long id=memberService.join(member);
```

트랜잭션으로 가져온 데이터를 요청한 지점에서 반환한 다음엔 DB 커넥션을 쓰지 않는다. DB 커넥션이 없으므로 사용자 요청이 많을 경우 유연하게 사용할 수 있다.

지연 로딩을 하려면 영속성 컨텍스트가 살아있어야 한다. 따라서 OSIV를 끄면 모든 지연 로딩을 트랜잭션 안에서 해결해야 한다. 지금까지 구현한 많은 지연 로딩 코드를 트랜잭션 안으로 넣어줘야 하는 것이다.

따라서 트랜잭션이 끝나기 전에 지연 로딩을 강제로 호출해두거나 fetch join을 사용해야 한다.

```java

@RestController
@RequiredArgsConstructor
public class OrderApiController {
    @GetMapping("/api/v1/orders")
    public List<Order> ordersV1() {
        List<Order> all = orderRepository.findAll(new OrderSearch());

        for (Order order : all) {
            order.getMember().getName(); // Lazy 강제 초기화
            order.getDelivery().getAddress(); // Lazy 강제 초기화
            List<OrderItem> orderItems = order.getOrderItems();
            orderItems.forEach(o -> o.getItem().getName()); // Lazy 강제 초기화
        }

        return all;
    }
}
```

```yaml
spring:
  jpa:
    open-in-view: false
```

![](../../.gitbook/assets/kimyounghan-spring-boot-and-jpa-optimization/04/screenshot%202021-06-05%20오후%207.24.11.png)

![](../../.gitbook/assets/kimyounghan-spring-boot-and-jpa-optimization/04/screenshot%202021-06-05%20오후%207.25.45.png)

false로 바꾸면 v1을 호출했을 때 500 에러가 나는 것을 볼 수 있다. 이를 해결하려면 for문에 있는 지연 로딩 로직을 서비스 안으로 옮겨야 한다.

## 커맨드와 쿼리 분리

실무에서 OSIV를 끈 상태로 복잡성을 관리하려면 커맨드와 쿼리를 분리하는 것이 좋다.

보통 비즈니스 로직은 특정 엔티티 몇 개를 등록하거나 수정하는 것이므로 성능이 크게 문제되진 않는다. 하지만 복잡한 화면을 출력하기 위한 쿼리는 화면에 맞춰 성능을 최적화하는 것이 중요하다. 

개발하다 보면 핵심적인 비즈니스 로직은 얼마 안되고 복잡하고 무거운 쿼리가 많다. 그럼 유지보수성이 떨어진다. 

화면에 뿌리기 위한 조회 API들은 자주 바뀌고 라이프사이클이 빠르다. 하지만 정책이 있는 비즈니스 로직은 잘 변경되지 않는다. 이 둘 사이의 라이프 사이클이 다르기 때문에 크고 복잡한 애플리케이션을 개발한다면 이 둘의 관심사를 명확하게 분리하는 것이 좋다.

따라서 OrderService엔 핵심 비즈니스 로직만 담고 OrderQueryService엔 화면이나 API에 맞춘 서비스로 구현한다. 후자는 주로 읽기 전용 트랜잭션을 사용한다.

보통 서비스 계층에서 트랜잭션을 유지하기 때문에, 두 서비스 모두 트랜잭션을 유지하면서 지연 로딩을 사용할 수 있다.

![](../../.gitbook/assets/kimyounghan-spring-boot-and-jpa-optimization/04/screenshot%202021-06-05%20오후%207.34.57.png)

아키텍처를 고려해 일반 비즈니스 서비스와는 분리해서 만든다.

{% tabs %} {% tab title="OrderQueryService.java" %}

```java
@Service
@RequiredArgsConstructor
@Transactional(readOnly = true)
public class OrderQueryService {

    private final OrderRepository orderRepository;

    @GetMapping("/api/v3/orders")
    public List<OrderDto> ordersV3() {
        List<Order> orders = orderRepository.findAllWithItem();

        return orders.stream()
                .map(OrderDto::new)
                .collect(toList());
    }
}
```

{% endtab %} {% tab title="OrderApiController.java" %}

```java

@RestController
@RequiredArgsConstructor
public class OrderApiController {
  private final OrderQueryService orderQueryService;

  @GetMapping("/api/v3/orders")
  public List<OrderDto> ordersV3() {

    return orderQueryService.ordersV3();
  }
}
```

{% endtab %} {% endtabs %}

데이터는 쿼리 서비스에서 모두 정리해서 컨트롤러에 반환한다.

## 실무 활용

- 고객 서비스 기반의 트래픽이 많은 실시간 API는 OSIV를 끈다.
- ADMIN처럼 커넥션을 많이 사용하지 않는 곳에서는 OSIV를 켠다.