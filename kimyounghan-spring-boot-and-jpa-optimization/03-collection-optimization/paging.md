# 페이징과 한계 돌파

- 컬렉션을 fetch join 하면 페이징이 불가능하다.
    - 1:N 조인이 발생하므로 데이터가 에측하지 못한 방향으로 뻥튀기된다.
    - 1:N에서 1을 기준으로 페이징 하는게 목적인데 데이터는 N을 기준으로 row가 생성된다.
    - 즉, order를 페이징하고 싶은데 orderItem이 기준이 되어버린다.
- 하이버네이트는 모든 DB 데이터를 읽어온 뒤 메모리에서 페이징하는 위험한 상황이 벌어질 수 있다.

## 해결 방법

1. OneToOne, ManyToOne 관계를 모두 fetch join 한다.
    - ToOne 관계는 row 수를 증가시키지 않으므로 페이징 쿼리에 영향을 주지 않는다.
    - ex) order의 member, delivery
2. 컬렉션은 지연 로딩으로 조회한다.
    - ex) order의 orderItem
3. 지연 로딩 성능이 최적화를 위해 `hibernate.default_batch_fetch_size`와 `@BatchSize`를 적용한다.
    - 컬렉션이나 프록시 객체를 설정한 size만큼 in 쿼리로 조회한다.
    - `hibernate.default_batch_fetch_size`: 글로벌 설정
    - `@BatchSize`: 개별 최적화

{% tabs %} {% tab title="OrderRepository.java" %}

```java

@Repository
@RequiredArgsConstructor
public class OrderRepository {

    public List<Order> findAllWithMemberDelivery(int offset, int limit) {
        return em.createQuery(
                "select o from Order o" +
                        " join fetch o.member m" +
                        " join fetch o.delivery d", Order.class)
                .setFirstResult(offset)
                .setMaxResults(limit)
                .getResultList();
    }
}
```

{% endtab %} {% tab title="OrderApiController.java" %}

```java

@RestController
@RequiredArgsConstructor
public class OrderApiController {

    @GetMapping("/api/v3.1/orders")
    public List<OrderDto> ordersV3_page(
            @RequestParam(value = "offset", defaultValue = "0") int offset,
            @RequestParam(value = "limit", defaultValue = "100") int limit) {
        List<Order> orders = orderRepository.findAllWithMemberDelivery(offset, limit);

        return orders.stream().map(OrderDto::new).collect(Collectors.toList());
    }
}
```

{% endtab %} {% tab title="application.yaml" %}

```yaml
spring:
  jpa:
    properties:
      hibernate:
        default_batch_fetch_size: 1000
```

{% endtab %} {% endtabs %}

![](../../.gitbook/assets/kimyounghan-spring-boot-and-jpa-optimization/03/screenshot%202021-05-30%20오후%208.39.25.png)

원래는 order 2개에서 orderItems가 있고 그 orderItems에 item이 2개씩 있어 N+1이 발생한다. 이젠 쿼리 수가 간단해졌다.

맨 마지막에 보면 유저 A와 B의 orderItems를 한 번에 가져왔다. 원래는 유저 A 2개, 유저 B 2개 총 4개의 쿼리가 나갔었다.

## 장점

- 쿼리 호출 수가 1+N에서 1+1로 최적화된다.
- fetch join 보다 DB 데이터 전송량이 최적화된다.
    - order와 orderItem을 조인하면 order가 orderItem만큼 중복되어 조회된다.
    - 하지만 이렇게 하면 각각 조회하기 때문에 전송할 중복 데이터가 사라진다.
- fetch join과 비교해 쿼리 호출 수가 약간 증가하지만 DB 데이터 전송량이 감소한다.
- 컬렉션 fetch join은 페이징이 불가능하지만 이 방법은 페이징이 가능하다.

ToOne 관계는 fetch join 해도 페이징에 영향을 주지 않는다. 따라서 ToOne 관계는 fetch join으로 쿼리 수를 줄이고, 나머지는 `default_batch_fetch_size`로 최적화 한다.

![](../../.gitbook/assets/kimyounghan-spring-boot-and-jpa-optimization/03/screenshot%202021-05-30%20오후%208.57.50.png)

V3에서는 컬렉션 때문에 중복 데이터가 조회되었다.

![](../../.gitbook/assets/kimyounghan-spring-boot-and-jpa-optimization/03/screenshot%202021-05-30%20오후%208.58.58.png)

V3.1은 반환 데이터는 똑같지만 쿼리 결과를 보면 최적화가 되어서 나온다. 데이터를 몇 천개씩 퍼올릴 때 사용하면 좋다.

## default_batch_fetch_size

- 적어놓은 개수만큼 미리 땡겨온다.
    - in 절은 PK를 가지고 빠른 속도로 조회해온다.
- 설정 값은 in 쿼리에 들어갈 조건의 개수와 같다.
    - 데이터가 1000개이고 100으로 설정해놨다면 쿼리가 10번 나간다.

실무에서 웬만하면 이 설정을 켜두고 있는 게 좋다.

### 적정 값

값은 100~1000 사이를 권장한다. DB에 따라 in절 파라미터를 1000으로 제한하기도 한다.

1000으로 잡으면 한번에 1000개를 불러오므로 DB에 순간 부하가 증가한다. 값이 적으면 부하를 낮추는 대신 잘라가면서 가니까 속도가 느리다.

```java
orders.stream().map(OrderDto::new).collect(Collectors.toList());
```

애플리케이션은 100이든 1000이든 결국 데이터를 가져올 때 위처럼 loop를 돌면서 전체 데이터를 로딩한다. 따라서 WAS 입장에서는 메모리 사용량이 같다. 1000이 쿼리를 덜 날려도 되니 성능상 가장 좋지만 결국 DB든 애플리케이션이든 순간 부하를 견딜 수 있는 값으로 한다.

## @BatchSize

- 개별로 설정하고 싶다면 컬렉션은 컬렉션 필드에, Entity는 Entity 클래스에 `@BatchSize`를 적용한다.