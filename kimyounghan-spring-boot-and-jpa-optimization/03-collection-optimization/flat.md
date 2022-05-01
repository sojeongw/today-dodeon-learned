# DTO 직접 조회: 플랫 데이터 최적화

- 이번엔 한 방 쿼리를 보내보자.

## join

{% tabs %} {% tab title="OrderApiController.java" %}

```java

@RestController
@RequiredArgsConstructor
public class OrderApiController {

    @GetMapping("/api/v6/orders")
    public List<OrderFlatDto> ordersV6() {
        List<OrderFlatDto> flats = orderQueryRepository.findAllByDto_flat();

        return flats;
    }
}
```

{% endtab %} {% tab title="OrderQueryRepository.java" %}

```java

@Repository
@RequiredArgsConstructor
public class OrderQueryRepository {

    public List<OrderFlatDto> findAllByDto_flat() {
        // 모든 데이터를 join 한다.
        return em.createQuery("select new jpabook.jpashop.repository.order.query.OrderFlatDto(o.id, m.name, o.orderDate, o.status, d.address, i.name, oi.orderPrice, oi.count)" +
                        " from Order o" +
                        " join o.member m" +
                        " join o.delivery d" +
                        " join o.orderItems oi" +
                        " join oi.item i", OrderFlatDto.class)
                .getResultList();
    }
}
```

{% endtab %} {% tab title="OrderFlatDto.java" %}

```java

@Data
public class OrderFlatDto {

    private Long orderId;
    private String name;
    private LocalDateTime orderDate;
    private Address address;
    private OrderStatus orderStatus;

    private String itemName;
    private int orderPrice;
    private int count;

    public OrderFlatDto(Long orderId, String name, LocalDateTime orderDate, OrderStatus orderStatus, Address address, String itemName, int orderPrice, int count) {
        this.orderId = orderId;
        this.name = name;
        this.orderDate = orderDate;
        this.address = address;
        this.orderStatus = orderStatus;
        this.itemName = itemName;
        this.orderPrice = orderPrice;
        this.count = count;
    }
}
```

{% endtab %} {% endtabs %}

```json
[
  {
    "orderId": 4,
    "name": "userA",
    "orderDate": "2022-05-01T15:53:40.001454",
    "address": {
      "city": "서울",
      "street": "1",
      "zipcode": "1111"
    },
    "orderStatus": "ORDER",
    "itemName": "JPA1 BOOK",
    "orderPrice": 10000,
    "count": 1
  },
  {
    "orderId": 4,
    "name": "userA",
    "orderDate": "2022-05-01T15:53:40.001454",
    "address": {
      "city": "서울",
      "street": "1",
      "zipcode": "1111"
    },
    "orderStatus": "ORDER",
    "itemName": "JPA2 BOOK",
    "orderPrice": 20000,
    "count": 2
  },
  {
    "orderId": 11,
    "name": "userB",
    "orderDate": "2022-05-01T15:53:40.033561",
    "address": {
      "city": "진주",
      "street": "2",
      "zipcode": "2222"
    },
    "orderStatus": "ORDER",
    "itemName": "SPRING1 BOOK",
    "orderPrice": 20000,
    "count": 3
  },
  {
    "orderId": 11,
    "name": "userB",
    "orderDate": "2022-05-01T15:53:40.033561",
    "address": {
      "city": "진주",
      "street": "2",
      "zipcode": "2222"
    },
    "orderStatus": "ORDER",
    "itemName": "SPRING2 BOOK",
    "orderPrice": 40000,
    "count": 4
  }
]
```

- 이렇게 모든 데이터를 join 해서 조회하면 중복 데이터를 반환한다.
- order가 아니라 orderItem이 기준이 되면서 페이징이 불가하다.

```sql
select order0_.order_id         as col_0_0_,
       member1_.name            as col_1_0_,
       order0_.order_date       as col_2_0_,
       order0_.status           as col_3_0_,
       delivery2_.city          as col_4_0_,
       delivery2_.street        as col_4_1_,
       delivery2_.zipcode       as col_4_2_,
       item4_.name              as col_5_0_,
       orderitems3_.order_price as col_6_0_,
       orderitems3_.count       as col_7_0_
from orders order0_
         inner join
     member member1_ on order0_.member_id = member1_.member_id
         inner join
     delivery delivery2_ on order0_.delivery_id = delivery2_.delivery_id
         inner join
     order_item orderitems3_ on order0_.order_id = orderitems3_.order_id
         inner join
     item item4_ on orderitems3_.item_id = item4_.item_id
```

- 하지만 쿼리가 총 한 번만 나간다는 장점이 있다.

## 중복 제거

{% tabs %} {% tab title="OrderApiController.java" %}

```java

@RestController
@RequiredArgsConstructor
public class OrderApiController {

    @GetMapping("/api/v6/orders")
    public List<OrderQueryDto> ordersV6() {
        List<OrderFlatDto> flats = orderQueryRepository.findAllByDto_flat();

        return flats.stream()
                // orderId 기준으로 묶는다.
                .collect(groupingBy(o -> new OrderQueryDto(o.getOrderId(), o.getName(), o.getOrderDate(), o.getOrderStatus(), o.getAddress()),
                        mapping(o -> new OrderItemQueryDto(o.getOrderId(), o.getItemName(), o.getOrderPrice(), o.getCount()), toList())
                )).entrySet().stream()
                .map(e -> new OrderQueryDto(e.getKey().getOrderId(), e.getKey().getName(), e.getKey().getOrderDate(), e.getKey().getOrderStatus(), e.getKey().getAddress(), e.getValue()))
                .collect(toList());
    }
}
```

{% endtab %} {% tab title="OrderQueryDto.java" %}

```java

@Data
// groupingBy할 때 묶는 기준이 뭔지 알려줘야 한다.
@EqualsAndHashCode(of = "orderId")
public class OrderQueryDto {
    private Long orderId;
    private String name;
    private LocalDateTime orderDate;
    private OrderStatus orderStatus;
    private Address address;
    private List<OrderItemQueryDto> orderItems;

    public OrderQueryDto(Long orderId, String name, LocalDateTime orderDate, OrderStatus orderStatus, Address address) {
        this.orderId = orderId;
        this.name = name;
        this.orderDate = orderDate;
        this.orderStatus = orderStatus;
        this.address = address;
    }

    public OrderQueryDto(Long orderId, String name, LocalDateTime orderDate, OrderStatus orderStatus, Address address, List<OrderItemQueryDto> orderItems) {
        this.orderId = orderId;
        this.name = name;
        this.orderDate = orderDate;
        this.orderStatus = orderStatus;
        this.address = address;
        this.orderItems = orderItems;
    }
}
```

{% endtab %} {% endtabs %}

```sql
 select order0_.order_id         as col_0_0_,
        member1_.name            as col_1_0_,
        order0_.order_date       as col_2_0_,
        order0_.status           as col_3_0_,
        delivery2_.city          as col_4_0_,
        delivery2_.street        as col_4_1_,
        delivery2_.zipcode       as col_4_2_,
        item4_.name              as col_5_0_,
        orderitems3_.order_price as col_6_0_,
        orderitems3_.count       as col_7_0_
 from orders order0_
          inner join
      member member1_ on order0_.member_id = member1_.member_id
          inner join
      delivery delivery2_ on order0_.delivery_id = delivery2_.delivery_id
          inner join
      order_item orderitems3_ on order0_.order_id = orderitems3_.order_id
          inner join
      item item4_ on orderitems3_.item_id = item4_.item_id
```

```json
[
  {
    "orderId": 11,
    "name": "userB",
    "orderDate": "2022-05-01T16:07:35.06548",
    "orderStatus": "ORDER",
    "address": {
      "city": "진주",
      "street": "2",
      "zipcode": "2222"
    },
    "orderItems": [
      {
        "orderId": 11,
        "itemName": "SPRING1 BOOK",
        "orderPrice": 20000,
        "count": 3
      },
      {
        "orderId": 11,
        "itemName": "SPRING2 BOOK",
        "orderPrice": 40000,
        "count": 4
      }
    ]
  },
  {
    "orderId": 4,
    "name": "userA",
    "orderDate": "2022-05-01T16:07:35.019514",
    "orderStatus": "ORDER",
    "address": {
      "city": "서울",
      "street": "1",
      "zipcode": "1111"
    },
    "orderItems": [
      {
        "orderId": 4,
        "itemName": "JPA1 BOOK",
        "orderPrice": 10000,
        "count": 1
      },
      {
        "orderId": 4,
        "itemName": "JPA2 BOOK",
        "orderPrice": 20000,
        "count": 2
      }
    ]
  }
]
```

- API 스펙을 OrderQueryDto로 맞춰야 한다면 노가다로 중복을 제거해 변환할 수 있다.
- `@EqualsAndHashCode(of = "orderId")`

## 단점

- 쿼리는 한 번이지만 join으로 인해 DB에서 애플리케이션에 전달하는 데이터가 중복된다.
    - 상황에 따라 V5보다 더 느릴 수도 있다.
- 애플리케이션에서 해야 할 추가 작업이 크다.
    - 분해하는 추가 작업이 필요하다.
- 페이징이 불가능하다.
    - 데이터가 중복되기 때문에 정확한 페이징 결과가 나오지 않는다.
    - 2개만 페이지 하면 중복 데이터 2개만 나오게 된다.