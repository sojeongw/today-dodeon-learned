# 엔티티 직접 노출

- 컬렉션인 1:N 관계를 조회하고 최적화 한다.

```java
@RestController
@RequiredArgsConstructor
public class OrderApiController {

    private final OrderRepository orderRepository;

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

`hibernate5Module`는 지연 로딩의 경우 null로 출력하기 때문에 이렇게 강제 초기화를 해준다. 물론 양방향 관계는 한 쪽에 `@JsonIgnore`를 꼭 붙여준다.