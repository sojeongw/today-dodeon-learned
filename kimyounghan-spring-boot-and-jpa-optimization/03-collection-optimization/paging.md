# 페이징과 한계 돌파

- 컬렉션을 fetch join 하면 페이징이 불가능하다.
    - 1:N 조인이 발생하므로 데이터가 에측하지 못한 방향으로 뻥튀기된다.
    - 1:N에서 1을 기준으로 페이징 하는게 목적인데 데이터는 N을 기준으로 row가 생성된다.
    - 즉, order를 페이징하고 싶은데 orderItem이 기준이 되어버린다.
- 하이버네이트는 모든 DB 데이터를 읽어온 뒤 메모리에서 페이징하는 위험한 상황이 벌어질 수 있다.

## 해결 방법

- OneToOne, ManyToOne 관계를 모두 fetch join 한다.
    - ToOne 관계는 row 수를 증가시키지 않으므로 페이징 쿼리에 영향을 주지 않는다.
    - ex) order의 member, delivery
- 컬렉션은 지연 로딩으로 조회한다.
    - fetch join은 사용하지 않는다.
    - ex) order의 orderItem
- 지연 로딩 성능 최적화를 위해 `hibernate.default_batch_fetch_size`와 `@BatchSize`를 적용한다.
    - 컬렉션이나 프록시 객체를 설정한 size만큼 in 쿼리로 조회한다.
    - `hibernate.default_batch_fetch_size`
        - 글로벌로 설정할 때 사용
    - `@BatchSize`
        - 개별로 최적화 할 때 사용

## before

{% tabs %} {% tab title="OrderRepository.java" %}

```java

@Repository
@RequiredArgsConstructor
public class OrderRepository {

    public List<Order> findAllWithMemberDelivery(int offset, int limit) {
        // ToOne 관계는 fetch join으로 가져온다.
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
    public List<OrderDto> ordersV3_page() {
        List<Order> orders = orderRepository.findAllWithMemberDelivery();
        return orders.stream().map(OrderDto::new).collect(Collectors.toList());
    }
}
```

{% endtab %} {% endtabs %}

```sql
select order0_.order_id       as order_id1_6_0_,
       member1_.member_id     as member_i1_4_1_,
       delivery2_.delivery_id as delivery1_2_2_,
       order0_.delivery_id    as delivery4_6_0_,
       order0_.member_id      as member_i5_6_0_,
       order0_.order_date     as order_da2_6_0_,
       order0_.status         as status3_6_0_,
       member1_.city          as city2_4_1_,
       member1_.street        as street3_4_1_,
       member1_.zipcode       as zipcode4_4_1_,
       member1_.name          as name5_4_1_,
       delivery2_.city        as city2_2_2_,
       delivery2_.street      as street3_2_2_,
       delivery2_.zipcode     as zipcode4_2_2_,
       delivery2_.status      as status5_2_2_
from orders order0_
         inner join
     member member1_ on order0_.member_id = member1_.member_id
         inner join
     delivery delivery2_ on order0_.delivery_id = delivery2_.delivery_id

select orderitems0_.order_id      as order_id5_5_0_,
       orderitems0_.order_item_id as order_it1_5_0_,
       orderitems0_.order_item_id as order_it1_5_1_,
       orderitems0_.count         as count2_5_1_,
       orderitems0_.item_id       as item_id4_5_1_,
       orderitems0_.order_id      as order_id5_5_1_,
       orderitems0_.order_price   as order_pr3_5_1_
from order_item orderitems0_
where orderitems0_.order_id = ?

select item0_.item_id        as item_id2_3_0_,
       item0_.name           as name3_3_0_,
       item0_.price          as price4_3_0_,
       item0_.stock_quantity as stock_qu5_3_0_,
       item0_.artist         as artist6_3_0_,
       item0_.etc            as etc7_3_0_,
       item0_.author         as author8_3_0_,
       item0_.isbn           as isbn9_3_0_,
       item0_.actor          as actor10_3_0_,
       item0_.director       as directo11_3_0_,
       item0_.dtype          as dtype1_3_0_
from item item0_
where item0_.item_id = ?

select item0_.item_id        as item_id2_3_0_,
       item0_.name           as name3_3_0_,
       item0_.price          as price4_3_0_,
       item0_.stock_quantity as stock_qu5_3_0_,
       item0_.artist         as artist6_3_0_,
       item0_.etc            as etc7_3_0_,
       item0_.author         as author8_3_0_,
       item0_.isbn           as isbn9_3_0_,
       item0_.actor          as actor10_3_0_,
       item0_.director       as directo11_3_0_,
       item0_.dtype          as dtype1_3_0_
from item item0_
where item0_.item_id = ?
    ...반복
```

- 위 로그는 order 결과 하나 당 나가는 쿼리다.
    - order 조회 후 order_item을 쿼리한다.
    - order_item에 item이 2개 있으므로 다시 2번 쿼리 한다.
- 다른 order에 대해서도 똑같이 order_item 1번, item 2번 쿼리한다.
- order가 100개라면 더 많은 쿼리가 나가게 될 것이다.

## after

{% tabs %} {% tab title="OrderApiController.java" %}

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

{% endtab %} {% tab title="OrderRepository.java" %}

```java

@Repository
public class OrderRepository {

    public List<Order> findAllWithMemberDelivery(int offset, int limit) {
        return em.createQuery(
                        "select o from Order o" +
                                " join fetch o.member m" +
                                " join fetch o.delivery d", Order.class)
                // 페이징을 적용한다.
                .setFirstResult(offset)
                .setMaxResults(limit)
                .getResultList();
    }
}
```

{% endtab %} {% tab title="application.yaml" %}

```yaml
spring:
  jpa:
    properties:
      hibernate:
        # 미리 in 절로 땡겨 올 데이터 개수
        default_batch_fetch_size: 1000
```

{% endtab %} {% endtabs %}

```sql
select order0_.order_id       as order_id1_6_0_,
       member1_.member_id     as member_i1_4_1_,
       delivery2_.delivery_id as delivery1_2_2_,
       order0_.delivery_id    as delivery4_6_0_,
       order0_.member_id      as member_i5_6_0_,
       order0_.order_date     as order_da2_6_0_,
       order0_.status         as status3_6_0_,
       member1_.city          as city2_4_1_,
       member1_.street        as street3_4_1_,
       member1_.zipcode       as zipcode4_4_1_,
       member1_.name          as name5_4_1_,
       delivery2_.city        as city2_2_2_,
       delivery2_.street      as street3_2_2_,
       delivery2_.zipcode     as zipcode4_2_2_,
       delivery2_.status      as status5_2_2_
from orders order0_
         inner join
     member member1_ on order0_.member_id = member1_.member_id
         inner join
     --     페이징이 적용된다.
         delivery delivery2_ on order0_.delivery_id = delivery2_.delivery_id limit ?
offset ?

select orderitems0_.order_id      as order_id5_5_1_,
       orderitems0_.order_item_id as order_it1_5_1_,
       orderitems0_.order_item_id as order_it1_5_0_,
       orderitems0_.count         as count2_5_0_,
       orderitems0_.item_id       as item_id4_5_0_,
       orderitems0_.order_id      as order_id5_5_0_,
       orderitems0_.order_price   as order_pr3_5_0_
from order_item orderitems0_
-- in 절로 땡겨온다.
where orderitems0_.order_id in (
                                ?, ?
    )

select item0_.item_id        as item_id2_3_0_,
       item0_.name           as name3_3_0_,
       item0_.price          as price4_3_0_,
       item0_.stock_quantity as stock_qu5_3_0_,
       item0_.artist         as artist6_3_0_,
       item0_.etc            as etc7_3_0_,
       item0_.author         as author8_3_0_,
       item0_.isbn           as isbn9_3_0_,
       item0_.actor          as actor10_3_0_,
       item0_.director       as directo11_3_0_,
       item0_.dtype          as dtype1_3_0_
from item item0_
-- in 절로 땡겨온다.
where item0_.item_id in (
                         ?, ?
    )
```

- 페이징이 잘 적용되었다.
- default_batch_fetch_size
    - order 2개와 그 아래의 데이터를 가져오는 데 쿼리가 3개만 나갔다.
    - 이전에는 order 마다 item 쿼리 2개씩 총 4개가 나갔는데 확 줄었다.
    - pk 기준으로 in 절을 날리기 때문에 쿼리 최적화로 빠르게 가져온다.
    - fetch_size를 100으로 정했는데 데이터가 1000개면 쿼리는 10개가 나간다.

## 비교

### V3

![](../../.gitbook/assets/kimyounghan-spring-boot-and-jpa-optimization/03/screenshot%202021-05-30%20오후%208.57.50.png)

- 진짜 한 방 쿼리로 모든 걸 가져온다.
- 컬렉션 때문에 중복 데이터가 많아져 부하 이슈가 있다.

### V3.1

![](../../.gitbook/assets/kimyounghan-spring-boot-and-jpa-optimization/03/screenshot%202021-05-30%20오후%208.58.58.png)

- 중복 없이 최적화 되어서 나온다.
- 데이터를 몇 천개씩 퍼올릴 때 사용하면 좋다.

## 장점

- 쿼리 호출 수가 1+N에서 1+1로 최적화된다.
- fetch join과 비교해 쿼리 호출 수가 약간 증가하지만 중복이 제거되어 DB 데이터 전송량이 감소한다.
- 컬렉션 fetch join은 페이징이 불가능하지만 이 방법은 페이징이 가능하다.
    - ToOne 관계는 fetch join 해도 페이징에 영향을 주지 않는다.
    - 따라서 ToOne 관계는 fetch join으로 쿼리 수를 줄이고, 나머지는 `default_batch_fetch_size`로 최적화 한다.

## default_batch_fetch_size

- 적어놓은 개수만큼 미리 땡겨온다.
    - in 절은 PK를 가지고 빠른 속도로 조회해온다.
- 설정 값은 in 쿼리에 들어갈 조건의 개수와 같다.
    - 데이터가 1000개이고 100으로 설정해놨다면 쿼리가 10번 나간다.

실무에서 웬만하면 이 설정을 켜두고 있는 게 좋다.

### 적정 값

- 1000개 이상은 부하로 오류가 발생하므로 사용하지 않는다.
    - DB에 따라 in절 파라미터를 1000으로 제한하기도 한다.
- 100~1000 사이를 권장한다.
    - 값이 적으면 부하를 낮추는 대신 잘라가면서 가니까 속도가 느리다.

```java
orders.stream().map(OrderDto::new).collect(Collectors.toList());
```

- 애플리케이션은 100이든 1000이든 결국 loop를 돌면서 전체 데이터를 로딩한다.
- 따라서 WAS 입장에서는 메모리 사용량이 같다.
- 1000이 쿼리를 덜 날려도 되니 성능상 가장 좋지만, DB와 애플리케이션 모두가 순간 부하를 견딜 수 있는 값으로 한다.

## @BatchSize

- 개별로 설정하고 싶다면 컬렉션은 컬렉션 `필드`단에, Entity는 Entity `클래스`단에 `@BatchSize`를 적용한다.