# 컬렉션 조회 최적화

{% tabs %} {% tab title=".java" %}

```java
@RestController
@RequiredArgsConstructor
public class OrderApiController {

    private final OrderRepository orderRepository;
    private final OrderQueryRepository orderQueryRepository;

    @GetMapping("/api/v5/orders")
    public List<OrderQueryDto> ordersV5() {
        return orderQueryRepository.findAllByDto_optimization();
    }

}
```

{% endtab %} {% tab title=".java" %}

```java
@Repository
@RequiredArgsConstructor
public class OrderQueryRepository {

    private final EntityManager em;
    
    public List<OrderQueryDto> findAllByDto_optimization() {
        // 일단 루트 쿼리로 ToOne 코드를 한 방에 가져온다.
        List<OrderQueryDto> result = findOrders();

        // orderId만 따로 뽑은 다음 쿼리 파라미터로 바로 넘겨버린다.
        List<Long> orderIds = result.stream().map(OrderQueryDto::getOrderId).collect(Collectors.toList());

        // 쿼리는 한 번만 날리고
        List<OrderItemQueryDto> orderItems = em.createQuery(
                "select new jpabook.jpashop.repository.order.query.OrderItemQueryDto(oi.order.id, i.name, oi.orderPrice, oi.count)" +
                        " from OrderItem oi" +
                        " join oi.item i" +
                        // in절로 여러 orderId에 대해 한 번에 가져온다.
                        " where oi.order.id in :orderIds", OrderItemQueryDto.class)
                .setParameter("orderIds", orderIds)
                .getResultList();

        // map으로 변환해 메모리에서 값을 매칭해서 가져온다.
        Map<Long, List<OrderItemQueryDto>> orderItemMap = orderItems.stream().collect(Collectors.groupingBy(OrderItemQueryDto::getOrderId));

        result.forEach(o -> o.setOrderItems(orderItemMap.get(o.getOrderId())));

        return result;
    }
}
```

{% endtab %} {% endtabs %}

in절로 여러 orderId에 대한 쿼리를 한 번에 날린 뒤, map으로 변환한 값을 이용해 메모리 상에서 값을 매칭해 가져온다.

![](../../.gitbook/assets/kimyounghan-spring-boot-and-jpa-optimization/03/screenshot%202021-05-31%20오후%209.31.04.png)

루트 쿼리 뒤에 orderItem을 가져오는 쿼리가 한 번만 실행되었다. 총 2번 호출된 것이다.

- ToOne 관계를 먼저 조회하고 여기서 얻은 식별자 orderId로 ToMany 관계인 OrderItem을 한 번에 조회한다.
- map을 사용해 매칭했기 때문에 성능이 O(1)로 향상되었다.