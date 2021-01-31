# 실전 예제
## 글로벌 fetch 전략 설정

- 모든 연관 관계를 지연 로딩으로 설정해야 한다.
- `@ManyToOne`, `@OneToOne`은 기본이 즉시 로딩이므로 지연 로딩으로 바꿔준다.

## 영속성 전이 설정

{% tabs %} {% tab title="Order.java" %}

```java
@Entity
@Table(name = "ORDERS")
public class Order {
  @Id
  @GeneratedValue
  @Column(name = "order_id")
  private Long id;

  // 주문을 할 때 배송도 생성하겠다는 의미
  // 둘의 라이프 사이클을 맞추게 된다.
  @OneToOne(fetch = FetchType.LAZY, cascade = CascadeType.ALL)
  @JoinColumn(name = "DELIVERY_ID")
  private Delivery delivery;

  // item로 마찬가지로 라이프 사이클을 맞춰준다.
  @OneToMany(mappedBy = "order", cascade = CascadeType.ALL)
  private List<OrderItem> orderItems = new ArrayList<>();
}

```

{% endtab %} {% endtabs %}

만약 라이프 사이클이 서로 일치하지 않고 로직이 복잡하다면 굳이 영속성 전이를 해줄 필요는 없다.