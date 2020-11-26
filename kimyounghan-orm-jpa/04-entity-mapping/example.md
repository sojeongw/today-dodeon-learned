# 실전 예제 - 요구 사항 분석과 기본 매핑
## 요구 사항

![](../../.gitbook/assets/kimyounghan-orm-jpa/04/screenshot%202021-03-17%20오전%207.23.34.png)

- 회원은 상품을 주문할 수 있다.
- 주문 시 여러 종류의 상품을 선택할 수 있다.

## 도메인 설계

![](../../.gitbook/assets/kimyounghan-orm-jpa/04/screenshot%202021-03-17%20오전%207.24.28.png)

- 회원과 주문
    - 회원은 여러 번 주문할 수 있다.
    - 일대다
- 주문과 상품
    - 주문할 때 여러 상품을 선택할 수 있다.
    - 같은 상품도 여러번 주문될 수 있다.
    - 다대다 관계를 일대다, 다대일 관계로 풀어낸다.
    
## 테이블 설계

![](../../.gitbook/assets/kimyounghan-orm-jpa/04/screenshot%202021-03-17%20오전%207.26.14.png)

## 엔티티 설계와 매핑

![](../../.gitbook/assets/kimyounghan-orm-jpa/04/screenshot%202021-03-17%20오전%207.28.51.png)

보통 처음에는 테이블과 엔티티를 똑같이 설계한다. 이렇게 됐을 때의 문제점이 무엇일지 살펴보자.

{% tabs %}{% tab title="Member.java" %}

```java
@Entity
// 인덱스나 length 같은 컬럼 제약 조건은 매핑 애너테이션을 이용해
// 코드에 표시해주는게 좋다. DB를 까보지 않고도 알 수 있기 때문이다.
public class Member {
  @Id
  @GeneratedValue(strategy = GenerationType.AUTO)
  @Column(name = "member_id")
  private Long id;
  private String name;
  private String city;
  private String street;
  private String zipcode;
}
```

{% endtab %} {% tab title="Order.java" %}

```java
@Entity
// DB에 order가 예약어로 걸려있어서 테이블명을 따로 지정해준다.
@Table(name = "ORDERS") 
public class Order {
  @Id
  @GeneratedValue
  @Column(name = "order_id")
  private Long id;

  // 누가 주문했는지 알기 위한 용도
  @Column(name = "member_id")
  private Long memberId;

  private LocalDateTime orderDate;

  @Enumerated(EnumType.STRING)
  private OrderStatus status;
}
```

{% endtab %} {% tab title="OrderItem.java" %}

```java
@Entity
public class OrderItem {
  @Id
  @GeneratedValue
  @Column(name = "order_item_id")
  private Long id;

  @Column(name = "order_id")
  private Long orderId;

  @Column(name = "item_id")
  private Long itemId;

  @Enumerated(EnumType.STRING)
  private OrderStatus status;
}
```

{% endtab %} {% tab title="Item.java" %}

```java
@Entity
public class Item {
  @Id
  @GeneratedValue
  @Column(name = "item_id")
  private Long id;

  private String name;

  private int price;

  private int stockQuantity;
}
```

{% endtab %}{% tab title="JpaMain.java" %}

```java
public class JpaMain {
  public static void main(String[] args) {
    EntityManagerFactory entityManagerFactory = Persistence.createEntityManagerFactory("hello");
    EntityManager em = entityManagerFactory.createEntityManager();
    EntityTransaction tx = em.getTransaction();
    tx.begin();

    try {
      Order order = em.find(Order.class, 1L);
      Long memberId = order.getMemberId();

      Member member = em.find(Member.class, memberId);

      tx.commit();
    } catch (Exception e) {
      tx.rollback();
    } finally {
      em.close();
    }

    entityManagerFactory.close();
  }
}
```

{% endtab %}{% endtabs %}

이렇게 세팅하면 `JpaMain`에서는 주문한 회원을 찾기 위해 `order` 조회 후 거기서 `memberId`를 찾아 다시 `member`를 조회해야 하는 번거로움이 있다.

객체는 참조로 쭉쭉 찾아가야 하는데 식별자를 통해서 해야 하면 끊겨버린다. 뭔가 객체지향스럽지 않다. 이런 설계는 관계형 DB에 객체를 맞춘 것이다.

## 데이터 중심 설계의 문제점

- 테이블의 외래 키를 객체에 그대로 가져온다.
- 객체 그래프 탐색이 불가능하다.
- 참조가 없으므로 UML도 잘못된다.
  - 앞서 살펴본 엔티티 매핑 다이어그램을 보면 id만 있을 뿐 실제로 참조는 다 끊긴다.
  
이런 문제를 해결하기 위해 연관 관계 매핑을 배운다.