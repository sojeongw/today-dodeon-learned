# 주문 도메인 개발
## 주문, 주문 상품 Entity 개발
### 생성 메서드

```java
@Entity
@Getter
@Setter
public class OrderItem {

  ...

  // 생성 메서드
  public static OrderItem createOrderItem(Item item, int orderPrice, int count) {
    // 쿠폰 등의 가격 변경 가능성 때문에 객체를 따로 만든다.
    OrderItem orderItem = new OrderItem();

    orderItem.setItem(item);
    orderItem.setOrderPrice(orderPrice);
    orderItem.setCount(count);

    // 넘어온 것만큼 재고를 뺀다.
    item.removeStock(count);

    return orderItem;
  }
}

```

Entity에 생성 메서드를 만들어주면, 생성하는 로직을 바꿀 때 여기만 손대면 된다.

```java
@Entity
@Getter
@Setter
@Table(name = "orders")
public class Order {

  ...

  // 생성 메서드
  public static Order createOrder(Member member, Delivery delivery, OrderItem... orderItems) {
    Order order = new Order();
    order.setMember(member);
    order.setDelivery(delivery);

    for (OrderItem orderItem : orderItems) {
      order.addOrderItem(orderItem);
    }

    order.setStatus(OrderStatus.ORDER);
    order.setOrderDate(LocalDateTime.now());

    return order;
  }
}
```

createOrder를 할 때는 createOrderItem에서 이미 removeStock()으로 재고를 깐 상태로 orderItem이 넘어오게 된다.

### 주문 취소

{% tabs %} {% tab title="Order.java" %}

```java
@Entity
@Getter
@Setter
@Table(name = "orders")
public class Order {
  
  ...

  // 비즈니스 로직
  /**
   * 주문 취소
   */
  public void cancel() {
    if(delivery.getStatus() == DeliveryStatus.COMP) {
      throw new IllegalStateException("이미 배송 완료된 상품은 취소가 불가능합니다.");
    }

    this.setStatus(OrderStatus.CANCEL);

    // IDE 에디터 색때문에 정말 this를 꼭 명시적으로 보여줘야할 때가 아니면
    // this.orderItems보다 orderItems를 선호한다.
    for (OrderItem orderItem : orderItems) {
      // 여러 상품을 주문했을 때 다 주문 취소를 해줘야 한다.
      orderItem.cancel();
    }
  }
}
```

{% endtab %} {% tab title="OrderItem.java" %}

```java
@Entity
@Getter
@Setter
public class OrderItem {
  
  ...

  // 비즈니스 로직
  public void cancel() {
    // 해당 아이템에 대한 주문 수량을 취소한 만큼 원복시켜야 한다.
    getItem().addStock(count);
  }
}
```

{% endtab %} {% endtabs %}

담당 도메인에서 로직을 처리하도록 개발한다.

### 전체 주문 가격 조회

{% tabs %} {% tab title="Order.java" %}

```java
@Entity
@Getter
@Setter
@Table(name = "orders")
public class Order {
  
  ...
  
  // 조회 로직
  /**
   * 전체 주문 가격 조회
   */
  public int getTotalPrice() {
    return orderItems.stream().mapToInt(OrderItem::getTotalPrice).sum();
  }
}

```

{% endtab %} {% tab title="OrderItem.java" %}

```java
@Entity
@Getter
@Setter
public class OrderItem {
  
  ...

  // 주문 가격과 수량이 orderItem에 있으므로 여기서 가격을 계산한다.
  public int getTotalPrice() {
    return getOrderPrice() * getCount();
  }
}
```

{% endtab %} {% endtabs %}

전체 주문 가격을 알려면 주문 상품 각각의 가격을 알아야 한다. 따라서 orderItem에서 가격을 조회한 다음 그 가격을 합한다.

실무에서는 주로 주문에 전체 주문 가격 필드를 추가하는 역정규화 방식을 쓴다.

## 주문 리포지토리 개발

```java
@Repository
@RequiredArgsConstructor
public class OrderRepository {

  private final EntityManager em;

  public void save(Order order) {
    em.persist(order);
  }

  public Order findOne(Long id) {
    return em.find(Order.class, id);
  }
}
```

## 주문 서비스 개발
### 주문
```java
@Service
@RequiredArgsConstructor
@Transactional(readOnly = true)
public class OrderService {

  private final OrderRepository orderRepository;
  private final MemberRepository memberRepository;
  private final ItemRepository itemRepository;

  /**
   * 주문
   */
  @Transactional
  public Long order(Long memberId, Long itemId, int count) {
    // Entity 조회
    Member member = memberRepository.findOne(memberId);
    Item item = itemRepository.findOne(itemId);

    // 배송 정보 생성
    Delivery delivery = new Delivery();
    delivery.setAddress(member.getAddress());

    // 주문 상품 생성
    OrderItem orderItem = OrderItem.createOrderItem(item, item.getPrice(), count);

    // 주문 생성
    Order order = Order.createOrder(member, delivery, orderItem);

    // 주문 저장
    orderRepository.save(order);

    return order.getId();
  }
}
```

원래 delivery, orderItem도 repository에 저장해줘야 하지만 여기서는 order만 하고 있다.

```java
@Entity
@Getter
@Setter
@Table(name = "orders")
public class Order {
  
  ...

  @OneToMany(mappedBy = "order", cascade = ALL)
  private List<OrderItem> orderItems = new ArrayList<>();

  @OneToOne(fetch = LAZY, cascade = ALL)
  @JoinColumn(name = "delivery_id")
  private Delivery delivery;

}
```

order에서 orderItem과 delivery가 cascade=ALL로 지정되어 있기 때문에 order만 persist 하면 다 같이 쿼리를 날려준다.

### cascade 범위

지금은 order가 orderItem과 delivery를 관리하는 느낌이다. orderItem과 delivery를 order 외에는 참조하지 않는다. 이렇게 다른 곳에서 참조하지 않으면서 라이프 사이클을 동일하게 관리해야 한다면 cascade를 적용한다.

하지만 만약 delivery나 orderItem이 중요한 도메인이라서 다른 곳에서도 쓰게 된다면 절대 사용하면 안된다.

### 생성 로직

```java
OrderItem orderItem = OrderItem.createOrderItem(item, item.getPrice(), count);
```

```java
OrderItem orderItem = new OrderItem();
orderItem.setCount();
```

내가 생성 로직을 따로 구현해놨더라도 사람에 따라 아래처럼 구현할 수도 있다. 이렇게 되면 수정하기가 어렵다.

```java
@Entity
@Getter
@Setter
public class OrderItem {
  protected OrderItem() {
  }
}
```

생성자를 protected로 해두면 막을 수 있다.

```java
@Entity
@Getter
@Setter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class OrderItem {
}
```

롬복을 쓰면 이렇게 줄일 수 있다.

### 주문 취소

{% tabs %} {% tab title="OrderService.java" %}

```java
@Service
@RequiredArgsConstructor
@Transactional(readOnly = true)
public class OrderService {
  /**
   * 주문 취소
   */
  @Transactional
  public void cancelOrder(Long orderId) {
    // 주문 Entity 조회
    Order order = orderRepository.findOne(orderId);

    // 주문 취소
    order.cancel();
  }
}
```

{% endtab %} {% tab title="Order.java" %}

```java
@Entity
@Getter
@Setter
// DB에 order by가 예약어로 걸려있어서 테이블명을 따로 지정해준다.
@Table(name = "orders")
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class Order {
  /**
   * 주문 취소
   */
  public void cancel() {
    if(delivery.getStatus() == DeliveryStatus.COMP) {
      throw new IllegalStateException("이미 배송 완료된 상품은 취소가 불가능합니다.");
    }

    this.setStatus(OrderStatus.CANCEL);

    for (OrderItem orderItem : orderItems) {
      orderItem.cancel();
    }
  }
}
```

{% endtab %} {% tab title="OrderItem.java" %}

```java
@Entity
@Getter
@Setter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class OrderItem {
  public void cancel() {
    getItem().addStock(count);
  }
}
```

{% endtab %} {% endtabs %}

원래는 order.cancel()의 OrderStatus와 orderItem.cancel()의 count 변경 사항도 각각 저장해줘야 한다. 하지만 JPA는 더티 체킹이 일어나면서 변경된 내용을 감지해 자동으로 쿼리를 날려준다.

### 도메인 모델 패턴

- Entity가 비즈니스 로직을 가지고 객체 지향을 적극 활용하는 패턴이다.
- 예제를 보면 비즈니스 로직 대부분이 Entity에 있다.
- 서비스 계층은 단순히 Entity에 필요한 요청을 위임하는 역할을 한다.

[도메인 모델 패턴](http://martinfowler.com/eaaCatalog/domainModel.html)

### 트랜잭션 스크립트 패턴

- Entity에 비즈니스 로직이 거의 없고 서비스 계층에서 대부분의 로직을 처리한다.

[트랜잭션 스크립트 패턴](http://martinfowler.com/eaaCatalog/transactionScript.html)