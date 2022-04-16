# Entity 설계 시 주의점

## Entity에는 가급적 setter를 사용하지 말자

- setter가 모두 열려있으면 변경 포인트가 너무 많아서 어디서 수정되었는지 알 수 없다.

## 모든 연관 관계는 지연 로딩으로 설정한다

- 일대일, 다대일은 기본이 즉시 로딩이므로 직접 설정해준다.
- 즉시 로딩은 한 번 불러올 때 모든 걸 다 불러온다.
    - 예측이 어렵고 어떤 SQL이 실행될지 추적하기 어렵다.
- 적용하지 않으면 N + 1 문제가 발생할 수 있다.
- 연관된 Entity를 함께 조회할 일이 있다면 fetch join이나 Entity 그래프 기능을 사용하자.

{% tabs %} {% tab title="Category.java" %}

```java

@Entity
@Getter
@Setter
public class Category {

  ...

    @ManyToOne(fetch = LAZY)
    @JoinColumn(name = "parent_id")
    private Category parent;
}

```

{% endtab %} {% tab title="Order.java" %}

```java

@Entity
@Getter
@Setter
@Table(name = "orders")
public class Order {
  
  ...

    @ManyToOne(fetch = LAZY)
    @JoinColumn(name = "member_id")

    @OneToOne(fetch = LAZY)
    @JoinColumn(name = "delivery_id")
    private Delivery delivery;
}

```

{% endtab %} {% tab title="OrderItem.java" %}

```java

@Entity
@Getter
@Setter
public class OrderItem {
  
  ...

    @ManyToOne(fetch = LAZY)
    @JoinColumn(name = "item_id")
    private Item item;

    @ManyToOne(fetch = LAZY)
    @JoinColumn(name = "order_id")
    private Order order;
}

```

`@OneToOne`, `@ManyToOne` 관계는 기본이 즉시 로딩이므로 직접 지연 로딩으로 설정해야 한다.

{% endtab %} {% endtabs %}

## 컬렉션은 필드에서 초기화 하자

```java

@Entity
@Getter
@Setter
public class Member {

  ...

    @OneToMany(mappedBy = "member")
    private List<Order> orders = new ArrayList<>();
}

```

컬렉션은 필드에서 바로 초기화 해야 null 문제를 피하고 안전하다.

```java
class Test {

    void test() {
        Member member = new Member();
        System.out.println(member.getOrders().getClass());

        em.persist(team);
        System.out.println(member.getOrders().getClass());

        // 출력 결과
        // class java.util.ArrayList
        // class org.hibernate.collection.internal.PersistentBag
    }
}
```

- 하이버네이트는 Entity를 영속화할 때 컬렉션을 감싼 뒤 하이버네이트가 제공하는 내장 컬렉션으로 변경한다.
    - 데이터 변경을 추적하기 위해 자기 식으로 바꿔서 들고 있는다.
- 만약 `getOrders()`같은 임의의 메서드에서 컬렉션을 잘못 생성하면 하이버네이트 내부에 문제가 발생한다.
- 따라서 필드 레벨에서 생성하는 것이 가장 안전하고 코드도 간결하다.
    - 처음에 딱 컬렉션을 생성해두고 바꾸지 않는다.

## 테이블 칼럼명 생성 전략

[hibernate naming strategy](https://docs.spring.io/spring-boot/docs/2.1.3.RELEASE/reference/htmlsingle/#howto-configure-hibernate-naming-strategy)

[hibernate naming guide](http://docs.jboss.org/hibernate/orm/5.4/userguide/html_single/Hibernate_User_Guide.html#naming)

옛날에는 하이버네이트에서 `SpringPhysicalNamingStrategy`를 기본으로 채택해 Entity 필드명이 그대로 칼럼명이 되었다. 예를 들어 필드가 `orderDate`
면 `orderDate` 그대로 생성된다.

반면 스프링 부트는 아래의 설정을 사용한다.

- 카멜 케이스를 언더스코어로 변경
    - memberPoint -> member_point
- 점을 언더스코어로 변경
- 대문자를 소문자로 변경

### ImplicitNamingStrategy

```yaml
spring.jpa.hibernate.naming.implicit-strategy:
  org.springframework.boot.orm.jpa.hibernate.SpringImplicitNamingStrategy
```

- 논리명 적용
- 명시적으로 칼럼, 테이블명을 직접 적지 않으면 이 전략이 적용된다.

### PhysicalNamingStrategy

```yaml
spring.jpa.hibernate.naming.physical-strategy:
  org.springframework.boot.orm.jpa.hibernate.SpringPhysicalNamingStrategy
```

- 물리명 적용
- 테이블명이 적혀있든 아니든 모든 논리명에 적용된다.

## cascade

```java

@Entity
@Getter
@Setter
@Table(name = "orders")
public class Order {

  ...

    @OneToMany(mappedBy = "order", cascade = ALL)
    private List<OrderItem> orderItems = new ArrayList<>();
}
```

- order를 저장할 때 order.orderItems에 세팅한 값이 같이 저장된다.
    - 원래는 order와 orderItems 각각 persist를 해줘야 저장된다.

## 연관 관계 편의 메서드

```java

@Entity
@Getter
@Setter
@Table(name = "orders")
public class Order {
  
  ...

    // 연관 관계 편의 메서드
    public void setMember(Member member) {
        this.member = member;
        member.getOrders().add(this);
    }

    public void addOrderItem(OrderItem orderItem) {
        orderItems.add(orderItem);
        orderItem.setOrder(this);
    }

    public void setDelivery(Delivery delivery) {
        this.delivery = delivery;
        delivery.setOrder(this);
    }
}

```

- 간편하게 메서드 하나를 이용해 양방향으로 세팅할 수 있게 한다. 
- 핵심적으로 컨트롤하는 쪽에 두면 된다.