# 실전 예제

![](../../.gitbook/assets/kimyounghan-orm-jpa/09/screenshot%202021-04-03%20오후%203.49.30.png)

주소를 값 타입으로 정의해보자.

굳이 여러 곳에서 쓰지 한 곳에서만 쓰는 값이어도 값 타입으로 해주는 것은 의미가 있다.

{% tabs %} {% tab title="Address.java" %}

```java
@Embeddable
public class Address {
  private String city;
  private String street;
  private String zipcode;

  private void setCity(String city) {
    this.city = city;
  }

  private void setStreet(String street) {
    this.street = street;
  }

  private void setZipcode(String zipcode) {
    this.zipcode = zipcode;
  }

  public String getCity() {
    return city;
  }

  public String getStreet() {
    return street;
  }

  public String getZipcode() {
    return zipcode;
  }

  @Override
  public boolean equals(Object o) {
    if (this == o) {
      return true;
    }
    if (o == null || getClass() != o.getClass()) {
      return false;
    }
    Address address = (Address) o;
    return Objects.equals(getCity(), address.getCity()) && Objects
        .equals(getStreet(), address.getStreet()) && Objects
        .equals(getZipcode(), address.getZipcode());
  }

  @Override
  public int hashCode() {
    return Objects.hash(getCity(), getStreet(), getZipcode());
  }
}
```

{% endtab %} {% endtabs %}

![](../../.gitbook/assets/kimyounghan-orm-jpa/09/screenshot%202021-04-03%20오후%203.53.29.png)

`equals()`를 구현할 때는 `Use getters` 옵션을 체크해야 필드에 직접 접근하지 않고 getter로 접근하게 해준다. 프록시일 때는 getter로만 진짜 객체에 접근할 수 있기 때문이다.

{% tabs %} {% tab title="Delivery.java" %}

```java
@Entity
public class Delivery {
  @Id @GeneratedValue
  private Long id;

  @Embedded
  private Address address;

  private DeliveryStatus status;

  @OneToOne(mappedBy = "delivery", fetch = FetchType.LAZY)
  private Order order;
}
```

{% endtab %} {% tab title="Member.java" %}

```java
@Entity
public class Member {
  @Id
  @GeneratedValue(strategy = GenerationType.AUTO)
  @Column(name = "member_id")
  private Long id;
  private String name;

  @Embedded
  private Address address;

  @OneToMany(mappedBy = "member")
  private List<Order> orders = new ArrayList<>();
}
```

{% endtab %} {% endtabs %}

![](../../.gitbook/assets/kimyounghan-orm-jpa/09/screenshot%202021-04-03%20오후%203.57.49.png)

![](../../.gitbook/assets/kimyounghan-orm-jpa/09/screenshot%202021-04-03%20오후%203.57.56.png)

`member`와 `delivery`에 주소가 생성되었다.

{% tabs %} {% tab title="Address.java" %}

```java
@Embeddable
public class Address {
  @Column(length = 10)
  private String city;
  
  @Column(length = 20)
  private String street;
  
  @Column(length = 5)
  private String zipcode;
  
  public String fullAddress() {
    return getCity() + getStreet() + getZipcode();
  }
}
```

{% endtab %} {% endtabs %}

![](../../.gitbook/assets/kimyounghan-orm-jpa/09/screenshot%202021-04-03%20오후%204.02.40.png)

값 타입을 만들면 공통으로 값을 관리할 수 있으므로 조건을 넣거나 의미있는 메서드를 만들 수 있어 편리하다.