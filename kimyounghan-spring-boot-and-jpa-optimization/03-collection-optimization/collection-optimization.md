# 컬렉션 조회 최적화

- 이전에 발생한 N+1 문제를 해결해보자.

{% tabs %} {% tab title="OrderApiController.java" %}

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

{% endtab %} {% tab title="OrderQueryRepository.java" %}

```java

@Repository
@RequiredArgsConstructor
public class OrderQueryRepository {
    public List<OrderQueryDto> findAllByDto_optimization() {
        // 일단 루트 쿼리로 ToOne 코드를 한 방에 가져온다.
        List<OrderQueryDto> result = findOrders();

        // orderId만 따로 뽑은 다음 쿼리 파라미터로 바로 넘겨버린다.
        List<Long> orderIds = result.stream().map(OrderQueryDto::getOrderId).collect(Collectors.toList());

        // 쿼리는 한 번만 날린다.
        List<OrderItemQueryDto> orderItems = em.createQuery(
                        "select new jpabook.jpashop.repository.order.query.OrderItemQueryDto(oi.order.id, i.name, oi.orderPrice, oi.count)" +
                                " from OrderItem oi" +
                                " join oi.item i" +
                                // in절로 여러 orderId에 대한 order_item을 한 번에 가져온다.
                                " where oi.order.id in :orderIds", OrderItemQueryDto.class)
                .setParameter("orderIds", orderIds)
                .getResultList();

        // 메모리상에서 map으로 변환 후 값을 매칭해서 가져온다.
        Map<Long, List<OrderItemQueryDto>> orderItemMap = orderItems.stream().collect(Collectors.groupingBy(OrderItemQueryDto::getOrderId));

        result.forEach(o -> o.setOrderItems(orderItemMap.get(o.getOrderId())));

        return result;
    }
}
```

{% endtab %} {% endtabs %}

```sql
 select order0_.order_id   as col_0_0_,
        member1_.name      as col_1_0_,
        order0_.order_date as col_2_0_,
        order0_.status     as col_3_0_,
        delivery2_.city    as col_4_0_,
        delivery2_.street  as col_4_1_,
        delivery2_.zipcode as col_4_2_
 from orders order0_
          inner join
      member member1_ on order0_.member_id = member1_.member_id
          inner join
      delivery delivery2_ on order0_.delivery_id = delivery2_.delivery_id

select orderitem0_.order_id    as col_0_0_,
       item1_.name             as col_1_0_,
       orderitem0_.order_price as col_2_0_,
       orderitem0_.count       as col_3_0_
from order_item orderitem0_
         inner join
     item item1_ on orderitem0_.item_id = item1_.item_id
-- in 절로 한 번에 가져온다.
where orderitem0_.order_id in (
                               ?, ?
    )
```

- in절로 여러 orderId에 대한 쿼리를 한 번에 날린 뒤, 메모리 상에서 값을 매칭해 가져온다.
    - map을 사용해 매칭했기 때문에 성능이 O(1)로 향상되었다.
- 쿼리는 총 2번 나간다.
    - 루트 쿼리 뒤에 orderItem을 가져오는 쿼리가 한 번만 실행된다.
    - ToOne 관계를 먼저 조회하고 여기서 얻은 식별자 orderId로 ToMany 관계인 OrderItem을 한 번에 조회한다.