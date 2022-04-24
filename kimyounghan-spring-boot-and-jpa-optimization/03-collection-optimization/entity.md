# Entity 직접 노출

- 컬렉션인 1:N 관계를 조회하고 최적화 한다.
    - 1:N을 join 하면 데이터가 뻥튀기되어 최적화 하기가 힘들다.

```java

@RestController
@RequiredArgsConstructor
public class OrderApiController {

    private final OrderRepository orderRepository;

    @GetMapping("/api/v1/orders")
    public List<Order> ordersV1() {
        List<Order> all = orderRepository.findAllByString(new OrderSearch());

        for (Order order : all) {
            order.getMember().getName(); // Lazy 강제 초기화
            order.getDelivery().getAddress(); // Lazy 강제 초기화
            List<OrderItem> orderItems = order.getOrderItems();
            
            // orderItem의 Item을 초기화 한다.
            orderItems.forEach(o -> o.getItem().getName()); // Lazy 강제 초기화
        }

        return all;
    }
}

```

- Order - OrderItem, OrderItem - Item 관계를 가져온다.
- 지연 로딩으로 설정한 연관 관계는 강제 초기화 한다.
    - `hibernate5Module`는 지연 로딩 필드를 null로 출력한다.
- 양방향 관계는 한 쪽에 `@JsonIgnore`를 꼭 붙여준다.
- Entity를 직접 노출하기 때문에 이 방법은 지양한다.