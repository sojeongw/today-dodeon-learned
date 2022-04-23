# Entity 직접 노출

- 주문, 배송 정보, 회원의 조회 API 개발한다.
- 지연 로딩으로 발생하는 성능 문제를 단계적으로 해결한다.

## 무한 루프

```java
/**
 * Order 조회
 * Order - Member는 ManyToOne
 * Order - Delivery는 OneToOne
 */
@RestController
@RequiredArgsConstructor
public class OrderSimpleApiController {
    private final OrderRepository orderRepository;

    @GetMapping("/api/v1/simple-orders")
    public List<Order> ordersV1() {
        List<Order> all = orderRepository.findAllByString(new OrderSearch());

        return all;
    }
}

```

![](../../.gitbook/assets/kimyounghan-spring-boot-and-jpa-optimization/02/screenshot%202021-05-30%20오후%205.26.36.png)

- API를 요청하면 무한 루프를 돈다.

{% tabs %} {% tab title="Order.java" %}

```java
public class Order {

  ...

    @ManyToOne(fetch = LAZY)
    @JoinColumn(name = "member_id")
    private Member member;
}
```

{% endtab %} {% tab title="Member.java" %}

```java
public class Member {
    
    ...

    @OneToMany(mappedBy = "member")
    private List<Order> orders = new ArrayList<>();
}

```

{% endtab %} {% endtabs %}

- Order와 Member가 서로를 참조하고 있기 때문이다.
    - Order에 Member가 있어서 들어가보면 Member에 Order가 있고...
- jackson이 무한 루프를 돌면서 계속 객체를 만든다.

### @JsonIgnore

```java
public class Member {
    
    ...

    @JsonIgnore
    @OneToMany(mappedBy = "member")
    private List<Order> orders = new ArrayList<>();
}

```

- 양방향이 걸리는 곳은 모두 한 쪽에 `@JsonIgnore`를 붙여야 피할 수 있다.
    - Order - Member
    - Delivery - Order
    - OrderItem - Order

## 지연 로딩 객체

![](../../.gitbook/assets/kimyounghan-spring-boot-and-jpa-optimization/02/screenshot%202021-05-30%20오후%205.35.34.png)

- 이번에는 bytebuddy 예외가 발생한다.

```java
public class Order {

    ...

    @ManyToOne(fetch = LAZY)
    @JoinColumn(name = "member_id")
    private Member member;
}
```

- member는 지연 로딩으로 가져오게 되어있다.
- DB에서 가져올 때는 order 데이터만 가져온다.
- member에는 프록시 객체를 생성해서 넣어둔다.
    - 이때 쓰는 라이브러리가 예외에 나온 `bytebuddy`다.
- jackson 라이브러리가 member를 변환하려는 순간, 프록시 객체를 json으로 어떻게 생성할지 몰라 에러를 낸 것이다.

### Hibernate5Module

{% tabs %} {% tab title="build.gradle" %}

```groovy
dependencies {
    implementation 'com.fasterxml.jackson.datatype:jackson-datatype-hibernate5'
}
```

{% endtab %} {% tab title="JpashopApplication.java" %}

```java

@SpringBootApplication
public class JpashopApplication {

    public static void main(String[] args) {
        SpringApplication.run(JpashopApplication.class, args);
    }

    @Bean
    Hibernate5Module hibernate5Module() {
        return new Hibernate5Module();
    }

}
```

{% endtab %} {% endtabs %}

- `Hibernate5Module`을 스프링 빈으로 등록하면 해결된다.

```json
[
  {
    "id": 4,
    "member": null,
    "orderItems": null,
    "delivery": null,
    "orderDate": "2021-05-30T17:42:39.704062",
    "status": "ORDER",
    "totalPrice": 50000
  },
  {
    "id": 11,
    "member": null,
    "orderItems": null,
    "delivery": null,
    "orderDate": "2021-05-30T17:42:39.738311",
    "status": "ORDER",
    "totalPrice": 220000
  }
]
```

- 지연 로딩 때문에 member, orderItems, delivery는 null로 출력된다.

### FORCE_LAZY_LOADING

```java

@SpringBootApplication
public class JpashopApplication {

    public static void main(String[] args) {
        SpringApplication.run(JpashopApplication.class, args);
    }

    @Bean
    Hibernate5Module hibernate5Module() {
        Hibernate5Module hibernate5Module = new Hibernate5Module();
        hibernate5Module.configure(Hibernate5Module.Feature.FORCE_LAZY_LOADING, true);
        return hibernate5Module;
    }

}
```

- FORCE_LAZY_LOADING
    - 강제로 지연 로딩하면서 제대로 된 데이터가 반환된다.
    - 양방향 연관 관계를 계속 로딩하기 때문에 `@JsonIgnore`를 꼭 한쪽에 추가해줘야 한다.
- 하지만 이렇게 강제로 모든 데이터를 로딩하는 옵션은 좋지 않다.
    - API 스펙 상 필요 없는 데이터까지 노출된다.
    - 추가로 데이터를 끌고 오면서 성능에도 문제가 발생한다.

### 직접 조회

{% tabs %} {% tab title="OrderSimpleApiController.java" %}

```java

@RestController
@RequiredArgsConstructor
public class OrderSimpleApiController {
    private final OrderRepository orderRepository;

    @GetMapping("/api/v1/simple-orders")
    public List<Order> ordersV1() {
        List<Order> all = orderRepository.findAll(new OrderSearch());

        for (Order order : all) {
            order.getMember().getName();
            order.getDelivery().getOrder();
        }

        return all;
    }
}

```

{% endtab %} {% tab title="JpashopApplication.java" %}

```java

@SpringBootApplication
public class JpashopApplication {

    public static void main(String[] args) {
        SpringApplication.run(JpashopApplication.class, args);
    }

    @Bean
    Hibernate5Module hibernate5Module() {
        Hibernate5Module hibernate5Module = new Hibernate5Module();
        return hibernate5Module;
    }

}
```

{% endtab %} {% endtabs %}

- get()을 통해 지연 로딩된 객체를 직접 조회하게 하면 `FORCE_LAZY_LOADING` 없이 불러올 수 있다.

## 정리

- Entity를 API 응답 형태로 외부에 노출하면 안된다.
    - DTO로 변환해서 반환하자.
- 지연 로딩을 피하려고 즉시 로딩을 설정하면 안된다.
    - 연관 관계가 필요없는 경우에도 항상 모든 데이터를 조회하면서 성능 문제가 발생한다.
        - 연관된 데이터를 다 조회하면서 N+1 문제가 터진다.
    - 즉시 로딩으로 설정하면 성능 튜닝이 매우 어려워진다.
    - 항상 지연 로딩을 기본으로 하자.
    - 성능 최적화가 필요하면 fetch join을 사용하자.