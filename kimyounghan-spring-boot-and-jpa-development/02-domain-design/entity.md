# 엔티티 클래스 개발

해당 예제에서는 설명을 쉽게 하기 위해 엔티티 클래스에 getter, setter를 모두 열어두고 최대한 단순하게 작업한다. 실무에서는 가급적 getter는 열고 setter는 꼭
필요한 경우에만 사용하자.

이론적으로는 꼭 필요한 별도 메서드로 제공하는 게 좋으나 실무에서는 데이터를 조회할 일이 많으므로 getter는 모두 열어두는 게 편하다.

하지만 setter는 호출하면 데이터가 변하므로 막 열어두면 엔티티가 도대체 왜 변경되는지 추적하기가 힘들다. 따라서 엔티티를 변경할 때는 setter 대신 변경만을 위한 비즈니스
메서드를 따로 제공해야 한다. 그럼 변경 지점이 명확해진다.

## 회원 엔티티

```java

@Entity
@Getter
@Setter
public class Member {

  @Id
  @GeneratedValue
  @Column(name = "member_id")
  private Long id;
  private String name;

  // 내장 타입을 사용할 때 붙인다.
  // Embedded, Embeddable 둘 중 하나만 있으면 되지만 명시적으로 둘 다 적는다.
  @Embedded
  private Address address;

  // order 테이블에 있는 member 필드에 의해 매핑된다는 것을 알려준다.
  // 읽기 전용이기 때문에 값을 수정해도 변경되지 않는다.
  @OneToMany(mappedBy = "member")
  private List<Order> orders = new ArrayList<>();
}
```

```java
// 내장 타입 정의
@Embeddable
@Getter
public class Address {

  @Column(length = 10)
  private String city;
  @Column(length = 20)
  private String street;
  @Column(length = 5)
  private String zipcode;
}
```

엔티티 식별자는 `id`를 사용하고 PK 컬럼은 `member_id`를 사용했다. 엔티티는 `Member`라는 타입이 있으니까 `id`라는 필드만으로도 쉽게 구분할 수 있지만, 테이블은 타입이 없어 구분이 힘들기 떄문이다.

테이블은 관례상 `테이블명 + id`를 많이 사용한다. 객체에서도 `id` 대신 `memberId`를 써도 된다. 중요한 건 일관성이다.

## 주문 엔티티

객체는 가지고 있는 엔티티 둘 다 변경해줘야 하지만 DB는 FK로 한 번만 변경해주면 된다. 따라서 FK가 있는 곳을 연관 관계의 주인으로 정한다. 해당 FK로 값을 변경한다는 의미다.

Order 테이블 다이어그램을 보면 `MEMBER_ID`라는 FK가 존재한다. 따라서 Order가 연관 관계의 주인이다. 

```java
@Entity
@Getter
@Setter
@Table(name = "orders") // DB에 order by가 예약어로 걸려있어서 테이블명을 따로 지정해준다.
public class Order {

  @Id
  @GeneratedValue
  @Column(name = "order_id")
  private Long id;

  // 누가 주문했는지 알기 위한 용도
  // 회원에게 주문은 여러 개지만, 해당 주문을 갖고있는 회원은 한 명이다.
  @ManyToOne
  @JoinColumn(name = "member_id")
  private Member member;

  @OneToMany(mappedBy = "order")
  private List<OrderItem> orderItems = new ArrayList<>();

  @OneToOne(fetch = FetchType.LAZY)
  @JoinColumn(name = "delivery_id")
  private Delivery delivery;

  private LocalDateTime orderDate;

  @Enumerated(EnumType.STRING)
  private OrderStatus status;
}

```

## 주문 상품 엔티티

```java
@Entity
@Getter
@Setter
public class OrderItem {

  @Id
  @GeneratedValue
  @Column(name = "order_item_id")
  private Long id;

  @ManyToOne
  @JoinColumn(name = "item_id")
  private Item item;

  @ManyToOne
  @JoinColumn(name = "order_id")
  private Order order;

  private int orderPrice;

  private int count;
}

```

하나의 주문 아이디에 여러 주문 상품이 들어가므로 FK로 `order_id`와 `item_id`를 갖는다. 

## 상품 엔티티

{% tabs %} {% tab title="Item.java" %}

```java
@Entity
@Getter
@Setter
// 전략을 부모 클래스에서 잡아준다.
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
// 구분자를 정의한다.
@DiscriminatorColumn(name = "dtype")
public abstract class Item {

  @Id
  @GeneratedValue
  @Column(name = "item_id")
  private Long id;

  private String name;
  private int price;
  private int stockQuantity;

  @ManyToMany(mappedBy = "items")
  private List<Category> categories = new ArrayList<>();
}
```

{% endtab %} {% tab title="Book.java" %}


```java
@Entity
@Getter
@Setter
@DiscriminatorValue("B")
public class Book extends Item{
  private String author;
  private String isbn;
}
```

{% endtab %} {% tab title="Album.java" %}

```java
@Entity
@Getter
@Setter
@DiscriminatorValue("A")
public class Album extends Item {

  private String artist;
  private String etc;
}

```

{% endtab %} {% tab title="Movie.java" %}

```java
@Entity
@Getter
@Setter
@DiscriminatorValue("M")
public class Movie extends Item {

  private String director;
  private String actor;
}

```

{% endtab %} {% endtabs %}

상속 관계로 정의한다.

## 배송 엔티티

```java
@Entity
@Getter
@Setter
public class Delivery {

  @Id
  @GeneratedValue
  @Column(name = "delivery_id")
  private Long id;

  @OneToOne(mappedBy = "delivery")
  private Order order;

  @Embedded
  private Address address;

  @Enumerated(EnumType.STRING)
  private DeliveryStatus status;

}

```

배송과 주문은 일대일 매핑이다. 일대일은 FK를 어디다 둘지 선택할 수 있다. 주로 접근하는 데이터에 두면 좋다. 주문을 보면서 배송을 보는 일이 많으므로 주문에 `delivery_id`를 FK로 둔다. 연관 관계 주인은 `Order.delivery`가 된다.

## 카테고리 엔티티

```java
@Entity
@Getter @Setter
public class Category {

  @Id @GeneratedValue
  @Column(name = "category_id")
  private Long id;

  private String name;

  // 실무에서는 중간 테이블에 추가적인 필드를 넣을 수가 없어서 많이 쓰지 않는다.
  @ManyToMany
  // 중간 테이블에 매핑해주어야 한다.
  @JoinTable(name = "category_item",  // 중간 테이블 이름
      joinColumns = @JoinColumn(name = "category_id"),  // 중간 테이블에 FK로 있는 id
      inverseJoinColumns = @JoinColumn(name = "item_id")) // 반대편(item)에서 가져올 FK
  private List<Item> items = new ArrayList<>();

  // 카테고리는 계층 구조이므로 부모와 자식을 만들어준다.
  @ManyToOne
  @JoinColumn(name = "parent_id")
  private Category parent;

  // parent를 기준으로 매핑된다.
  // 셀프로 양방향 연관 관계를 맺는 방식이다.
  @OneToMany(mappedBy = "parent")
  private List<Category> child = new ArrayList<>();
}

```

## 주소 엔티티

```java
// 내장 타입 정의
@Embeddable
@Getter
public class Address {

  @Column(length = 10)
  private String city;
  @Column(length = 20)
  private String street;
  @Column(length = 5)
  private String zipcode;

  public Address(String city, String street, String zipcode) {
    this.city = city;
    this.street = street;
    this.zipcode = zipcode;
  }
  
  public Address() {
    
  }
}
```

값 타입은 기본적으로 immutable하게 즉, 변경이 되지 않아야 한다. 그래서 setter를 사용하지 않고 무조건 새로 생성해서 쓰도록 생성자를 만들어둔다.

기본 생성자도 함께 만들어주어야 한다. JPA가 생성 시에 리플렉션이나 프록시 같은 기술을 써야하는데 기본 생성자가 없으면 사용할 수가 없다. JPA 스펙 상 엔티티나 임베디드 타입은 기본 생성자를 public이나 protected로 설정해야 한다. public보다는 안전하게 protected로 해주자.

![](../../.gitbook/assets/kimyounghan-spring-boot-and-jpa/02/screenshot%202021-05-16%20오후%206.38.14.png)

실행하면 잘 동작하는 것을 확인할 수 있다. 로그를 보면 자동으로 FK를 걸어주고 있다. DB엔 웬만하면 인덱스만 걸어주면 되는데 정말 정확해야 하는 돈 관련 문제라면 FK까지 걸어주는 것이 좋다.