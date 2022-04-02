# 다대다

- `@ManyToMany`를 사용한다.
- `@JoinTable`로 연결 테이블을 지정한다.
- 단방향, 양방향이 가능하다.

실무에서는 사용하지 않는다. 관계형 데이터베이스는 정규화된 테이블 2개라 다대다 관계를 표현할 수 없기 때문이다. 연결 테이블을 추가해서 일대다, 다대일 관계로 풀어내야 한다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/06/screenshot%202021-03-20%20오후%203.15.11.png)
 
회원이 여러 개의 상품을 주문할 수 있다면 DB는 테이블 2개만으로 다대다 관게를 못 만들어낸다. 중간에 `member_product` 테이블을 만들어 풀어낸다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/06/screenshot%202021-03-20%20오후%203.18.07.png)

하지만 객체는 컬렉션을 사용해 객체 2개로 다대다 관계를 표현할 수 있다. 멤버가 상품 리스트를, 상품은 멤버 리스트를 가질 수 있다.

## 단방향

{% tabs %} {% tab title="Member.java" %}

```java
@Entity
public class Member {

  @Id
  @GeneratedValue
  private Long id;

  @ManyToMany
  // 연결 테이블이 될 테이블의 이름을 적어준다.
  // 그럼 알아서 member id와 product id를 pk, fk로 갖는다.
  @JoinTable(name = "MEMBER_PRODUCT")
  private List<Product> products = new ArrayList<>();
}

```

{% endtab %} {% tab title="Product.java" %}

```java
@Entity
public class Product {
  @Id @GeneratedValue
  private Long id;

  private String name;
}

```

{% endtab %} {% endtabs %}

![](../../.gitbook/assets/kimyounghan-orm-jpa/06/screenshot%202021-03-20%20오후%203.28.18.png)

`MEMBER_PRODUCT`라는 테이블이 자동으로 생성된 것을 볼 수 있다.

## 양방향

{% tabs %} {% tab title="Member.java" %}

```java
@Entity
public class Member {

  @Id
  @GeneratedValue
  private Long id;

  @ManyToMany
  @JoinTable(name = "MEMBER_PRODUCT")
  private List<Product> products = new ArrayList<>();
}

```

{% endtab %} {% tab title="Product.java" %}

```java
@Entity
public class Product {
  @Id @GeneratedValue
  private Long id;
  private String name;

  // 양방향을 할 때는 똑같이 mappedBy를 해준다.
  @ManyToMany(mappedBy = "products")
  private List<Member> members = new ArrayList<>();
}

```

{% endtab %} {% endtabs %}

## 한계

![](../../.gitbook/assets/kimyounghan-orm-jpa/06/screenshot%202021-03-20%20오후%203.32.29%201.png)

간편해 보이지만 실무에서 쓰면 안된다. 연결 테이블은 단순히 연결만 하고 끝나지 않는다. 주문 시간, 수량 같은 데이터가 필요해도 위의 그림처럼 데이터를 추가할 수가 없다.

쿼리가 이상하게 나가는 문제도 있다. 중간 테이블이랑 조인해서 나가야 하는데 중간 테이블이 숨겨져 있기 때문에 내가 모르는 쿼리가 나가서 문제가 생길 수 있다.

## 극복 방법

연결용 테이블 Entity를 추가한다. 즉, 연결 테이블을 Entity로 승격한다. 다대다를 일대다, 다대일로 풀어낸다.

{% tabs %} {% tab title="Member.java" %}

```java
@Entity
public class Member {

  @Id
  @GeneratedValue
  private Long id;

  @OneToMany(mappedBy = "member")
  private List<MemberProduct> memberProducts = new ArrayList<>();
}

```

{% endtab %} {% tab title="MemberProduct.java" %}

```java
@Entity
public class MemberProduct {
  @Id @GeneratedValue
  private Long id;

  @ManyToOne
  @JoinColumn(name = "MEMBER_ID")
  private Member member;

  @ManyToOne
  @JoinColumn(name = "PRODUCT_ID")
  private Product product;

  private int orderAmount;
  private LocalDateTime orderDateTime;
}
```

{% endtab %} {% tab title="Product.java" %}

```java
@Entity
public class Product {
  @Id @GeneratedValue
  private Long id;
  private String name;

  @OneToMany(mappedBy = "product")
  private List<MemberProduct> memberProducts = new ArrayList<>();
}

```

{% endtab %}{% endtabs %}

![](../../.gitbook/assets/kimyounghan-orm-jpa/06/screenshot%202021-03-20%20오후%203.34.36.png)

풀면 결과적으로 위와 같은 그림이 된다. 중간 테이블은 `Orders` 같은 의미있는 이름으로 변경해도 된다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/06/screenshot%202021-03-20%20오후%203.32.29%201.png)

위처럼 PK와 FK를 묶어서 잡는 방법도 있다. 하지만 이것보다는 웬만하면 PK에 의미 없는 값을 쓰는 게 나중에 유연성이 생긴다. Id가 어디에 종속되면 걸림돌이 되는 일이 많다.