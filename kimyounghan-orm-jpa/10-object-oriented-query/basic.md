# 기본 문법과 쿼리 API

## JPQL(Java Persistence Query Language)

- 객체 지향 쿼리 언어
    - 테이블이 아니라 Entity 객체를 대상으로 쿼리한다.
- SQL을 추상화해서 특정 DB SQL에 의존하지 않는다.
- 매핑 정보를 조합해 최종적으로 SQL로 변환되어 실행된다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/10/screenshot%202021-04-03%20오후%204.50.16.png)

- member는 team, order와 연관 관계를 맺는다.
- order는 address라는 값 타입을 가지고 있다.

{% tabs %} {% tab title="Member.java" %}

```java

@Entity
public class Member {

    @Id
    @GeneratedValue
    private Long id;

    private String username;
    private int age;

    @ManyToOne
    @JoinColumn(name = "TEAM_ID")
    private Team team;
}
```

{% endtab %} {% tab title="Team.java" %}

```java

@Entity
public class Team {

    @Id
    @GeneratedValue
    private Long id;
    private String name;

    @OneToMany(mappedBy = "team")
    private List<Member> members = new ArrayList<>();
}
```

{% endtab %}{% tab title="Order.java" %}

```java

@Entity
@Table(name = "ORDERS")
public class Order {

    @Id
    @GeneratedValue
    private Long id;

    private int orderAmount;

    @Embedded
    private Address address;

    @ManyToOne
    @JoinColumn(name = "PRODUCT_ID")
    private Product product;
}
```

{% endtab %}{% tab title="Product.java" %}

```java

@Entity
public class Product {

    @Id
    @GeneratedValue
    private Long id;

    private String name;
    private int price;
    private int stockAmount;
}

```

{% endtab %}{% tab title="Address.java" %}

```java

@Embeddable
public class Address {

    private String city;
    private String street;
    private String zipcode;
}
```

{% endtab %}{% endtabs %}

## 문법

![](../../.gitbook/assets/kimyounghan-orm-jpa/10/screenshot%202021-04-03%20오후%206.30.53.png)

```jpaql
select m from Member as m where m.age > 18
```

- Entity와 속성은 대소문자를 구분한다.
    - Entity
        - Member
    - 속성
        - age
- JPQL 키워드는 대소문자를 구분하지 않는다.
    - ex. select, from, where
- 테이블 이름이 아니라 Entity 이름을 사용해야 한다.
    - ex. Member
    - `@Entity(name = "")` 값으로 설정된다.
    - 기본 값은 클래스와 같은 이름이다.
- 별칭은 필수로 사용해야 한다.
    - ex. as m
    - as는 생략 가능하다.

## 집합과 정렬

![](../../.gitbook/assets/kimyounghan-orm-jpa/10/screenshot%202021-04-03%20오후%206.38.21.png)

`group by`, `having`등도 똑같이 사용하면 된다.

## TypeQuery, Query 타입

### TypeQuery

```java
public class JpaMain {

    public static void main(String[] args) {
        TypedQuery<Member> a = em.createQuery("select m from Member m", Member.class);
        TypedQuery<String> b = em.createQuery("select m.username from Member m", String.class);
    }
}
```

- 반환 타입이 명확할 때 사용한다.

### Query

```java
public class JpaMain {

    public static void main(String[] args) {
        // username과 age는 타입이 달라서 명시할 수 없으므로 Query 타입을 사용한다.
        Query a = em.createQuery("select m.username, m.age from Member m");
    }
}
```

- 반환 타입이 명확하지 않을 때 사용한다.

## 결과 조회 API

### query.getResultList()

- 결과가 하나 이상일 때 리스트를 반환한다.
- 결과가 없으면 빈 리스트를 반환한다.

### query.getSingleResult()

- 결과가 **정확히** 하나일 때 단일 객체를 반환한다.
- `javax.persistence.NoResultException`
    - 결과가 없을 때
    - 결과가 당연히 없을 수도 있는 건데 try-catch로 다시 처리해줘야 해서 불편하다.
        - Spring Data JPA를 사용하면 Optional로 반환해 예외를 던지지 않는다.
- `javax.persistence.NonUniqueResultException`
    - 결과가 2개 이상일 때

## 파라미터 바인딩

### 이름 기준

```java
public class JpaMain {

    public static void main(String[] args) {
        Member singleResult = em
                .createQuery("SELECT m FROM Member m where m.username=:username", Member.class)
                .setParameter("username", "member1");

        System.out.println(singleResult.getUserName());
    }
}
```

- 이름으로 지정해서 가져오기 때문에 순서가 뒤바뀌는 실수가 발생하지 않는다.

### 위치 기준

```java
public class JpaMain {

    public static void main(String[] args) {
        TypedQuery<Member> query = em
                .createQuery("SELECT m FROM Member m where m.username=?1", Member.class);

        query.setParameter(1, usernameParam);
    }
}
```

- 중간에 끼워 넣다보면 실수할 수 있다.
    - 웬만하면 사용하지 말자.
