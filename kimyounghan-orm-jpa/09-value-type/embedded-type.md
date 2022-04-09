# 임베디드 타입

- 새로운 값 타입을 직접 정의할 수 있다.
- 주로 기본 값 타입을 모아서 만들기 때문에 복합 값 타입이라고도 한다.
- int, String처럼 임베디드 타입도 Entity가 아닌 그냥 **값 타입**이다.
    - 변경해도 추적이 되지 않는다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/09/screenshot%202021-03-31%20오후%207.52.44.png)

- 근무 시작일, 근무 종료일과 도시, 우편 번호, 주소는 공통으로 묶을 수 있는 데이터다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/09/screenshot%202021-03-31%20오후%207.52.51.png)

- 이 데이터는 workPeriod, homeAddress처럼 묶어서 나타낼 수 있다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/09/screenshot%202021-03-31%20오후%207.52.58.png)

- workPeriod, homeAddress 클래스에 데이터를 정의한다.

## 사용법

- 기본 생성자를 필수로 만들어야 한다.

### @Embeddable

- 값을 정의하는 곳에 사용

### @Embedded

- 값 타입을 사용하는 곳에 사용

## 특징

- 재사용이 가능하다.
    - 기간이나 주소는 시스템 전체에서 재사용할 수 있는 데이터다.
- 응집도가 높다.
    - `Period.isWork()`처럼 해당 값 타입만 사용하는 의미있는 메서드를 만들 수 있다.
- 임베디드 타입을 포함한 모든 값 타입은 값 타입을 소유한 Entity에 생명주기를 의존한다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/09/screenshot%202021-03-31%20오후%208.02.23.png)

- 테이블 안의 칼럼은 똑같다.
- 매핑만 그림과 같이 적절하게 해주면 된다.

{% tabs %} {% tab title="Member.java" %}

```java

@Entity
public class Member {

    @Id
    @GeneratedValue
    private Long id;

    @Column(name = "name")
    private String username;

    // period
    private LocalDateTime startDate;
    private LocalDateTime endDate;

    // address
    private String city;
    private String street;
    private String zipcode;
}
```

{% endtab %} {% endtabs %}

![](../../.gitbook/assets/kimyounghan-orm-jpa/09/screenshot%202021-03-31%20오후%208.07.14.png)

일단 개별 데이터로 정의해서 실행하면 테이블에 잘 생성된다.

{% tabs %} {% tab title="Member.java" %}

```java

@Entity
public class Member {

    @Id
    @GeneratedValue
    private Long id;

    @Column(name = "name")
    private String username;

    // 임베디드 타입으로 수정한다.
    @Embedded
    private Period period;

    // 임베디드 타입으로 수정한다.
    @Embedded
    private Address address;

    // 응집성 있는 로직 구현
    public boolean isWork() {
        return true;
    }
}
```

{% endtab %} {% tab title="Period.java" %}

```java

@Embeddable
public class Period {

    private LocalDateTime startDate;
    private LocalDateTime endDate;
}
```

{% endtab %}{% tab title="Address.java" %}

```java

@Embeddable
public class Address {

    private String city;
    private String street;
    private String zipcode;

    // 기본 생성자 필수
    public Address() {
    }

    public Address(String city, String street, String zipcode) {
        this.city = city;
        this.street = street;
        this.zipcode = zipcode;
    }
}

```

{% endtab %}{% tab title="JpaMain.java" %}

```java
public class JpaMain {

    public static void main(String[] args) {
        Member member = new Member();
        member.setUsername("name");
        member.setAddress(new Address("city", "street", "10001"));
        member.setPeriod(new Period());

        em.persist(member);

        tx.commit();
    }
}


```

{% endtab %}{% endtabs %}

![](../../.gitbook/assets/kimyounghan-orm-jpa/09/screenshot%202021-03-31%20오후%208.18.51.png)

임베디드 값으로 넣어도 이전과 같은 형태로 insert 된다.

## 임베디드 타입과 테이블 매핑

- 임베디드 타입은 Entity의 값일 뿐이다.
- 임베디드 타입을 사용하기 전과 후에 **매핑하는 테이블은 같다.**
- 대신, 객체와 테이블을 아주 세밀하게 매핑하는 게 가능해진다.
    - 잘 설계한 ORM 애플리케이션은 매핑한 테이블의 수보다 클래스의 수가 더 많다.
- 공통으로 사용할 수 있는 도메인 언어가 많아지는 장점이 있다.

## 임베디드 타입과 연관 관계

![](../../.gitbook/assets/kimyounghan-orm-jpa/09/screenshot%202021-03-31%20오후%208.02.29.png)

- `Address`와 `ZipCode`처럼 임베디드 타입도 임베디드 타입을 가질 수 있다.
- `PhoneNumber`와 `PhoneEntity`처럼 임베디드 타입이 Entity를 가질 수도 있다.
    - FK만 가지고 있으면 가능하다.

## @AttributeOverride

- 한 엔티티에서 같은 값 타입을 사용할 때 적용한다.

{% tabs %} {% tab title="Member.java" %}

```java

@Entity
public class Member {

    @Id
    @GeneratedValue
    private Long id;

    @Column(name = "name")
    private String username;

    @Embedded
    private Period period;

    // 같은 임베디드 타입을 중복해서 사용할 경우 에러가 난다.
    @Embedded
    private Address homeAddress;

    @Embedded
    private Address workAddress;
}
```

{% endtab %} {% endtabs %}

- 같은 값 타입을 사용하면 컬럼명이 중복되면서 에러가 발생한다.

{% tabs %} {% tab title="Member.java" %}

```java

@Entity
public class Member {

    @Id
    @GeneratedValue
    private Long id;

    @Column(name = "name")
    private String username;

    @Embedded
    private Period period;

    @Embedded
    private Address homeAddress;

    // 칼럼 이름을 재정의 해준다.
    @Embedded
    @AttributeOverrides({
            @AttributeOverride(name = "city", column = @Column(name = "work_city")),
            @AttributeOverride(name = "street", column = @Column(name = "work_street")),
            @AttributeOverride(name = "zipcode", column = @Column(name = "work_zipcode"))
    })
    private Address workAddress;
}
```

{% endtab %} {% endtabs %}

![](../../.gitbook/assets/kimyounghan-orm-jpa/09/screenshot%202021-03-31%20오후%208.30.13.png)

- 칼럼 명을 새로 매핑해준다.

## 임베디드 타입과 null

- 임베디드 타입의 값이 null이면, 그 타입 안에 정의해서 매핑한 칼럼 값은 모두 null이다.
    - `Period`가 null이면 그 안에 있는 `startDate` 등도 null이다.