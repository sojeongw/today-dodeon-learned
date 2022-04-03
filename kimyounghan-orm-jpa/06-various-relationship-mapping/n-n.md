# 다대다

- @ManyToMany
    - 실무에서는 사용하지 않는다.
    - 관계형 데이터베이스는 정규화된 테이블 2개라 다대다 관계를 표현할 수 없기 때문이다.
- 연결 테이블을 추가해서 일대다, 다대일 관계로 풀어내야 한다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/06/screenshot%202021-03-20%20오후%203.15.11.png)

- DB는 테이블로 다대다를 만들 수 없기 때문에 중간에 member_product 테이블을 만들어 풀어낸다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/06/screenshot%202021-03-20%20오후%203.18.07.png)

- 객체는 컬렉션을 사용해 객체 2개로 다대다 관계를 표현할 수 있다.
- member가 List<Product>를, Product가 List<Member>를 가질 수 있다.

## 단방향 매핑

{% tabs %} {% tab title="Member.java" %}

```java

@Entity
public class Member {

    ...

    @ManyToMany
    // @JoinTable에 연결 테이블의 이름을 적어준다.
    // 그럼 알아서 member_id와 product_id를 pk, fk로 갖는다.
    @JoinTable(name = "MEMBER_PRODUCT")
    private List<Product> products = new ArrayList<>();
}

```

{% endtab %} {% tab title="Product.java" %}

```java

@Entity
public class Product {
    @Id
    @GeneratedValue
    private Long id;

    private String name;
}

```

{% endtab %} {% endtabs %}

![](../../.gitbook/assets/kimyounghan-orm-jpa/06/screenshot%202021-03-20%20오후%203.28.18.png)

`MEMBER_PRODUCT`라는 테이블이 자동으로 생성되었다.

## 양방향 매핑

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
    @Id
    @GeneratedValue
    private Long id;
    private String name;

    // 일반적인 양방향 매핑과 같이 mappedBy를 해준다.
    @ManyToMany(mappedBy = "products")
    private List<Member> members = new ArrayList<>();
}

```

{% endtab %} {% endtabs %}

- members.products가 연관 관계의 주인임을 mappedBy로 표현한다.

## 한계

![](../../.gitbook/assets/kimyounghan-orm-jpa/06/screenshot%202021-03-20%20오후%203.32.29%201.png)

- 편해 보이지만 실무에서 쓰면 안된다.
    - 실무에서는 연결 테이블이 단순이 연결하는 기능만 하지 않는다.
        - 주문 시간, 수량 같은 추가적인 데이터가 필요해도 위의 그림처럼 테이블에 데이터를 추가할 수 없다.
    - 예상치 못한 쿼리가 나갈 수 있다.
        - 중간 테이블이과 조인해서 나가야 하는데, 중간 테이블이 숨겨져 있기 때문에 내가 모르는 쿼리가 나가서 문제가 생길 수 있다.

## 극복 방법

- 연결용 테이블 Entity를 추가한다.
- 즉, 연결 테이블을 Entity로 승격하고 일대다, 다대일로 풀어낸다.

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
// 중간 테이블
@Entity
public class MemberProduct {
    @Id
    @GeneratedValue
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
    @Id
    @GeneratedValue
    private Long id;
    private String name;

    @OneToMany(mappedBy = "product")
    private List<MemberProduct> memberProducts = new ArrayList<>();
}

```

{% endtab %}{% endtabs %}

![](../../.gitbook/assets/kimyounghan-orm-jpa/06/screenshot%202021-03-20%20오후%203.34.36.png)

- 중간 테이블을 연관 관계의 주인으로 두고 푼다.
- 중간 테이블의 이름은 Orders 등 의미있는 이름으로 변경해도 된다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/06/screenshot%202021-03-20%20오후%203.32.29%201.png)

- FK인 member_id와 product_id 둘을 묶어 PK가 되게 잡을 수도 있다.
- 하지만 웬만하면 PK는 자동 생성 되는 의미 없는 값을 써야 유연성이 생긴다.
    - id가 어디에 종속되면 걸림돌이 되는 일이 많다.