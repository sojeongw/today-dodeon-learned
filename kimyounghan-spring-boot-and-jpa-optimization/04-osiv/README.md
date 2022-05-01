# OSIV와 성능 최적화

- Open Session In View
    - JPA에 EntityManager가 있다면 하이버네이트에는 세션이 있다.
- JPA는 Open EntityManager In View라고 하지만 관례상 OSIV라고 부른다.

```properties
spring.jpa.open-in-view:true
```

스프링에서는 OSIV가 기본적으로 켜져있다.

![](../../.gitbook/assets/kimyounghan-spring-boot-and-jpa-optimization/04/screenshot%202021-06-05%20오후%206.15.57.png)

스프링을 실행하면 warn 로그가 하나 남는다. DB 커넥션과 관련된 문제로 보인다.

## OSIV ON

![](../../.gitbook/assets/kimyounghan-spring-boot-and-jpa-optimization/04/screenshot%202021-06-05%20오후%206.10.31.png)

- DB 트랜잭션을 시작할 때 JPA 영속성 컨텍스트가 DB 커넥션을 가져온다.
- OSIV가 켜져있으면 @Transactional 메서드를 벗어나도 커넥션을 계속 유지한다.
    - API 응답이 나가고 화면이 렌더링 될 때까지 영속성 컨텍스트를 물고 있는다.
- 그래서 View Template이나 Controller 단에서도 지연 로딩으로 데이터를 가져올 수 있었던 것이다.
    - 지연 로딩은 영속성 컨텍스트가 살아 있어야 가능하다.
- 너무 오랫동안 DB 커넥션을 사용하기 때문에 실시간 트래픽이 중요한 애플리케이션에서는 커넥션이 모자랄 수 있다.
    - 결국 장애로 이어진다.
- 로직 상에서 외부 API를 호출한다면 이걸 처리하는 시간만큼 커넥션 리소스를 반환하지 못한다.
- 지연 로딩을 적극 활용할 수 있다는 장점이 있다.

## OSIV OFF

```properties
spring.jpa.open-in-view:false
```

- 트랜잭션을 종료할 때 영속성 컨텍스트를 닫고 DB 커넥션을 반환한다.
- 따라서 커넥션 리소스를 낭비하지 않는다.

![](../../.gitbook/assets/kimyounghan-spring-boot-and-jpa-optimization/04/screenshot%202021-06-05%20오후%207.14.26.png)

- 트랜잭션으로 가져온 데이터를 요청한 지점에서 반환한 다음엔 DB 커넥션을 쓰지 않는다.
    - 사용자 요청이 많을 경우 유연하게 사용할 수 있다.
- 모든 지연 로딩을 트랜잭션 안에서 해결해야 한다.
    - 지연 로딩을 하려면 영속성 컨텍스트가 살아있어야 한다.
    - 지금까지 구현한 많은 지연 로딩 코드를 트랜잭션 안으로 넣어줘야 한다.
    - 따라서 트랜잭션이 끝나기 전에 지연 로딩을 강제로 호출해두거나 fetch join을 사용해야 한다.

### 예제

```java

@RestController
@RequiredArgsConstructor
public class OrderApiController {

    @GetMapping("/api/v1/orders")
    public List<Order> ordersV1() {
        List<Order> all = orderRepository.findAll(new OrderSearch());

        // 지연 로딩을 시도한다.
        for (Order order : all) {
            order.getMember().getName();
            order.getDelivery().getAddress();
            List<OrderItem> orderItems = order.getOrderItems();
            orderItems.forEach(o -> o.getItem().getName());
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

- OSIV를 false로 바꾸면 프록시를 초기화 하지 못해 에러가 발생한다.
    - for문에 있는 지연 로딩 로직을 service 안으로 옮겨서 해결한다.

## 실무 활용 팁

- 고객 서비스 기반의 트래픽이 많은 실시간 API
    - OSIV를 끈다.
- ADMIN처럼 커넥션을 많이 사용하지 않는 곳
    - OSIV를 켠다.

## 커맨드와 쿼리 분리

- 실무에서 OSIV를 끈 상태로 복잡성을 관리하려면 커맨드와 쿼리를 분리한다.
- 성능 문제는 주로 조회에서 발생한다.
- 비즈니스 로직
    - 정책적인 것이라 잘 변경되지 않는다.
    - 특정 Entity 몇 개를 등록하거나 수정하는 것이 전부라서 성능이 크게 문제되진 않는다.
- 화면에 뿌리기 위한 조회 API
    - 자주 바뀌고 라이프사이클이 빠르다.
    - 복잡한 화면을 출력해야 하므로 성능 최적화하가 중요하다.
- 이 둘 사이의 라이프 사이클이 다르기 때문에 명확하게 분리하는 것이 좋다.

### 예제

![](../../.gitbook/assets/kimyounghan-spring-boot-and-jpa-optimization/04/screenshot%202021-06-05%20오후%207.34.57.png)

- OrderService
    - 핵심 비즈니스 로직만 담는다.
- OrderQueryService
    - 화면이나 API에 맞춘 서비스로 구현한다.
    - 주로 읽기 전용 트랜잭션을 사용한다.
- 보통 서비스 계층에서 트랜잭션을 유지하기 때문에, 두 서비스 모두 트랜잭션을 유지하면서 지연 로딩을 사용할 수 있다.