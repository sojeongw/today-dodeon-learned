# JPA에서 DTO로 바로 조회

{% tabs %} {% tab title="OrderSimpleApiController.java" %}

```java
@RestController
@RequiredArgsConstructor
public class OrderSimpleApiController {
    private final OrderRepository orderRepository;

    @GetMapping("/api/v4/simple-orders")
    public List<OrderSimpleQueryDto> ordersV4() {
        return orderRepository.findOrderDtos();
    }
}
```

{% endtab %} {% tab title="OrderRepository.java" %}

```java
@Repository
@RequiredArgsConstructor
public class OrderSimpleQueryRepository {
    public List<OrderSimpleQueryDto> findOrderDtos() {
        return em.createQuery(
                // 원하는 필드를 직접 선택해서 쿼리를 짠다.
                "select new jpabook.jpashop.repository.order.simplequery.OrderSimpleQueryDto(o.id, m.name, o.status, o.orderDate, d.address)"
                        + " from Order o"
                        + " join o.member m"
                        + " join o.delivery d", OrderSimpleQueryDto.class)
                .getResultList();
    }
}
```

{% endtab %} {% tab title="OrderSimpleQueryDto.java" %}

```java
@Data
public class OrderSimpleQueryDto {
    private Long orderId;
    private String name;
    private OrderStatus orderStatus;
    private LocalDateTime orderDate;
    private Address address;

    public OrderSimpleQueryDto(Long orderId, String name, OrderStatus orderStatus, LocalDateTime orderDate, Address address) {
        this.orderId = orderId;
        this.name = name;
        this.orderStatus = orderStatus;
        this.orderDate = orderDate;
        this.address = address;
    }
}

```

{% endtab %} {% endtabs %}

![](../../.gitbook/assets/kimyounghan-spring-boot-and-jpa-optimization/02/screenshot%202021-05-30%20오후%206.50.07.png)

V3에서는 이렇게 쿼리가 날아갔다.

![](../../.gitbook/assets/kimyounghan-spring-boot-and-jpa-optimization/02/screenshot%202021-05-30%20오후%206.49.51.png)

수정 후에는 원하는 필드만 지정하면서 select 쿼리가 간단해졌다. 데이터를 적게 퍼올리는 만큼 네트워크 용량도 줄어든다.

```java
@Repository
@RequiredArgsConstructor
public class OrderRepository {
    public List<Order> findAllWithMemberDelivery() {
        return em.createQuery(
                "select o from Order o" +
                        " join fetch o.member m" +
                        " join fetch o.delivery d", Order.class)
                .getResultList();
    }
}
```

V3는 외부의 모습을 건드리지 않은 상태로 내부에서 원하는 것만 fetch join으로 가져왔다. 그만큼 리포지토리 재사용성이 좋고 개발이 단순해진다.

```java
@Repository
@RequiredArgsConstructor
public class OrderSimpleQueryRepository {
    public List<OrderSimpleQueryDto> findOrderDtos() {
        return em.createQuery(
                "select new jpabook.jpashop.repository.order.simplequery.OrderSimpleQueryDto(o.id, m.name, o.status, o.orderDate, d.address)"
                        + " from Order o"
                        + " join o.member m"
                        + " join o.delivery d", OrderSimpleQueryDto.class)
                .getResultList();
    }
}
```

반면 V4는 내가 필요한 것만 직접 쿼리를 짰기 때문에 재사용성이 떨어진다. DTO로 조회하기 때문에 Entity를 수정할 수도 없다. 

사실 대부분의 성능은 from, join에서 먹는다. select를 몇 개 줄인다고 엄청나게 성능이 개선되지 않는다. 실시간으로 사용하는 유저가 많다면 그때 고려해보자.

DTO가 Entity를 위한 respository단에 사용되는 것도 애매하기 때문에 `OrderSimpleQueryRepository`처럼 패키지를 구분해 쿼리용 repository를 따로 파는 게 좋다.

## 쿼리 방식 선택 권장 순서

1. 우선 Entity를 DTO로 변환한다.
2. 필요하면 fetch join으로 성능을 최적화 한다.
    - 대부분의 이슈가 여기서 해결된다.
3. 그래도 안되면 DTO로 직접 조회한다.
4. 최후의 방법은 JPA가 제공하는 네이티브 SQL이나 스프링 JDBC Template으로 직접 SQL을 쓰는 것이다.

