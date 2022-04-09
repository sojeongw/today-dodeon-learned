# 값 타입 컬렉션

- 값 타입을 컬렉션에 담아서 사용하는 방식

![](../../.gitbook/assets/kimyounghan-orm-jpa/09/screenshot%202021-04-03%20오후%201.54.45.png)

- favoriteFoods와 addressHistory는 컬렉션 타입이다.
    - DB는 컬렉션을 담을 수 있는 타입이 없다.
- 일대다 개념처럼 별도의 테이블로 만든다.
- 별도의 식별자 없이 소속된 테이블의 외래 키와 값 타입을 조합해 PK로 쓴다.
    - ID를 따로 만들어 PK를 쓰게 되면 값 타입이 아니라 Entity가 되기 때문이다.

## @ElementCollection, @CollectionTable

- 값 타입을 하나 이상 저장할 때 사용한다.

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
    private Address homeAddress;

    // 컬렉션으로 이루어진 값 타입에 달아준다.
    @ElementCollection
    @CollectionTable(
            // 값을 넣을 테이블 이름을 정의한다. 
            name = "FAVORITE_FOOD",
            // 외래키를 명시한다.
            joinColumns = @JoinColumn(name = "MEMBER_ID"))
    // addressHistory는 Address 타입 내부에 city, address 등 다양한 필드가 있지만
    // favoriteFoods는 String 하나이고 내가 정의한 타입이 아니기 때문에 칼럼 이름을 지정해줄 수 있다.
    @Column(name = "FOOD_NAME")
    private Set<String> favoriteFoods = new HashSet<>();

    @ElementCollection
    @CollectionTable(name = "ADDRESS", joinColumns = @JoinColumn(name = "MEMBER_ID"))
    private List<Address> addressHistory = new ArrayList<>();
}
```

{% endtab %} {% endtabs %}

![](../../.gitbook/assets/kimyounghan-orm-jpa/09/screenshot%202021-04-03%20오후%202.26.03.png)

- 임베디드 타입인 Address의 값들이 추가되었다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/09/screenshot%202021-04-03%20오후%202.25.50.png)

![](../../.gitbook/assets/kimyounghan-orm-jpa/09/screenshot%202021-04-03%20오후%202.25.39.png)

- 설정한대로 FAVORITE_FOOD, ADDRESS 테이블을 생성했다.
- MEMBER_ID가 PK, FK가 되었다.
    - 어느 회원에 속하는지 연관 관계를 알아야 하기 때문에 FK는 반드시 필요하다.

## 값 타입 저장

{% tabs %} {% tab title="JpaMain.java" %}

```java
public class JpaMain {

    public static void main(String[] args) {
        Member member = new Member();
        member.setUsername("member1");
        member.setHomeAddress(new Address("home city", "street", "12345"));

        // 값 타입 컬렉션
        member.getFavoriteFoods().add("치킨");
        member.getFavoriteFoods().add("족발");
        member.getFavoriteFoods().add("피자");

        member.getAddressHistory().add(new Address("old1", "street", "12345"));
        member.getAddressHistory().add(new Address("old2", "street", "12345"));

        // member만 영속화 한다.
        em.persist(member);

        tx.commit();
    }
}
```

{% endtab %} {% endtabs %}

![](../../.gitbook/assets/kimyounghan-orm-jpa/09/screenshot%202021-04-03%20오후%202.35.39.png)

`Member`를 생성한 뒤 `AddressHistory` 2개, `FavoriteFood` 3개가 insert 된다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/09/screenshot%202021-04-03%20오후%202.40.05.png)

해당 멤버에 잘 매핑되었다.

- 값 타입 컬렉션을 따로 persist 하지 않고 `member`만 persist했는데도 같이 저장됐다.
- 값 타입은 `member`에 의존하기 때문에 `member`가 변경되면 같이 변경된다.
    - `member`와 라이프사이클을 같이 한다.
- 즉, `cascade`와 `orphanRemoval`을 모두 켠 상태를 필수로 가지고 있다고 볼 수 있다.

## 값 타입 조회

{% tabs %} {% tab title="JpaMain.java" %}

```java
public class JpaMain {

    public static void main(String[] args) {
        Member member = new Member();
        member.setUsername("member1");
        member.setHomeAddress(new Address("home city", "street", "12345"));

        member.getFavoriteFoods().add("치킨");
        member.getFavoriteFoods().add("족발");
        member.getFavoriteFoods().add("피자");

        member.getAddressHistory().add(new Address("old1", "street", "12345"));
        member.getAddressHistory().add(new Address("old2", "street", "12345"));

        em.persist(member);

        // DB에는 데이터가 insert되고 영속성 컨텍스트를 깔끔하게 처리한 상태에서
        em.flush();
        em.clear();

        // member를 다시 조회한다.
        Member findMember = em.find(Member.class, member.getId());

        tx.commit();
    }
}
```

{% endtab %} {% endtabs %}

![](../../.gitbook/assets/kimyounghan-orm-jpa/09/screenshot%202021-04-03%20오후%202.44.59.png)

- `member`를 조회하면 값 타입 컬렉션인 favoriteFoods와 addressHistory는 조회되지 않는다.
    - 값 타입 컬렉션에 자동으로 지연 로딩이 적용됐기 때문이다.
    - 임베디드 타입은 소속된 값이기 때문에 `homeAddress`는 당연히 조회되었다.

{% tabs %} {% tab title="JpaMain.java" %}

```java
public class JpaMain {

    public static void main(String[] args) {
        Member member = new Member();
        member.setUsername("member1");
        member.setHomeAddress(new Address("home city", "street", "12345"));

        member.getFavoriteFoods().add("치킨");
        member.getFavoriteFoods().add("족발");
        member.getFavoriteFoods().add("피자");

        member.getAddressHistory().add(new Address("old1", "street", "12345"));
        member.getAddressHistory().add(new Address("old2", "street", "12345"));

        em.persist(member);

        em.flush();
        em.clear();

        Member findMember = em.find(Member.class, member.getId());

        // 값 타입 컬렉션을 조회한다.
        List<Address> addressHistory = findMember.getAddressHistory();
        for (Address address : addressHistory) {
            System.out.println("address = " + address.getCity());
        }

        tx.commit();
    }
}
```

{% endtab %} {% endtabs %}

![](../../.gitbook/assets/kimyounghan-orm-jpa/09/screenshot%202021-04-03%20오후%203.00.56.png)

- 값 타입 컬렉션을 실제 조회할 때가 되어서야 쿼리가 나간다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/09/screenshot%202021-04-03%20오후%203.02.34.png)

- `@ElementCollection` 선언부를 보면 기본 fetch 전략이 지연 로딩으로 되어있다.

## 값 타입 수정

{% tabs %} {% tab title="JpaMain.java" %}

```java
public class JpaMain {

    public static void main(String[] args) {
        Member member = new Member();
        member.setUsername("member1");
        member.setHomeAddress(new Address("home city", "street", "12345"));

        member.getFavoriteFoods().add("치킨");
        member.getFavoriteFoods().add("족발");
        member.getFavoriteFoods().add("피자");

        member.getAddressHistory().add(new Address("old1", "street", "12345"));
        member.getAddressHistory().add(new Address("old2", "street", "12345"));

        em.persist(member);

        em.flush();
        em.clear();

        Member findMember = em.find(Member.class, member.getId());

        // 값 타입인 임베디드 타입은 immutable 해야 하기 때문에 이렇게 변경하면 절대 안된다.
        // findMember.getHomeAddress().setCity("new city");

        // address 인스턴스 자체를 갈아끼워야 한다.
        Address a = findMember.getHomeAddress();
        findMember.setHomeAddress(new Address("new city", a.getStreet(), a.getZipcode()));

        // 컬렉션 값 타입도 마찬가지로 불변성을 유지해야 하므로 업데이트가 아니라 통째로 갈아끼운다.
        // 기존 값을 지우고 다시 넣는다.
        findMember.getFavoriteFoods().remove("치킨");
        findMember.getFavoriteFoods().add("한식");

        // 임베디드 타입과 컬렉션 값 타입을 영속화 하는 코드가 없지만 쿼리가 나간다.
        // 영속성 전이와 고아 객체 제거 기능을 필수로 가지기 때문이다.

        tx.commit();
    }
}
```

{% endtab %} {% endtabs %}

- 값 타입은 불변이어야 하므로 꼭 인스턴스를 통으로 갈아 끼워서 수정한다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/09/screenshot%202021-04-03%20오후%203.11.01.png)

- 컬렉션의 값만 변경해도 어떤 값이 변경되었는지 알고 JPA가 알아서 DB에 쿼리를 날려준다.
    - 마치 영속성 전이가 된 것처럼 동작한다.
    - 값 타입 컬렉션은 `member`로 라이프 사이클이 관리되는 단순한 속성이기 때문이다.

{% tabs %} {% tab title="JpaMain.java" %}

```java
public class JpaMain {

    public static void main(String[] args) {
        Member member = new Member();
        member.setUsername("member1");
        member.setHomeAddress(new Address("home city", "street", "12345"));

        member.getFavoriteFoods().add("치킨");
        member.getFavoriteFoods().add("족발");
        member.getFavoriteFoods().add("피자");

        member.getAddressHistory().add(new Address("old1", "street", "12345"));
        member.getAddressHistory().add(new Address("old2", "street", "12345"));

        em.persist(member);

        em.flush();
        em.clear();

        Member findMember = em.find(Member.class, member.getId());

        // address를 하나만 바꾸고 싶다면 지우고 싶은 값을 넣고 remove 한다.
        // 컬렉션은 대부분 equals()를 사용해 찾고 싶은 값을 그대로 찾아준다.
        // 따라서 equals()를 재정의 하지 않았다면 그냥 망하는 것이다. equals()를 꼭 재정의 해주자.
        findMember.getAddressHistory().remove(new Address("old1", "street", "12345"));
        // 지운 값 대신 새로운 값을 넣어준다.
        findMember.getAddressHistory().add(new Address("new city", "street", "12345"));

        tx.commit();
    }
}
```

{% endtab %} {% endtabs %}

- `addressHistory`에서 `old1`만 수정하고 싶다면 해당 값을 찾아서 지우고 다시 넣는다.
- 컬렉션은 값을 찾을 때 기본적으로 `equals()`를 사용하므로 반드시 오버라이딩이 필요하다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/09/screenshot%202021-04-03%20오후%203.21.19.png)

![](../../.gitbook/assets/kimyounghan-orm-jpa/09/screenshot%202021-04-03%20오후%203.24.49.png)

- 쿼리를 확인해보니 insert문이 2개가 나갔다.
- 테이블에 있는 데이터를 완전히 갈아끼우는 것이기 때문이다.
    1. delete로 `member_id`에 해당하는 `address` 테이블 값을 통째로 삭제한다.
    2. 기존 값이었던 `old2`를 insert 한다.
    3. 새로 추가한 `new city`를 insert 한다.

## 제약 사항

- 값 타입 컬렉션을 매핑하는 테이블은 모든 칼럼을 묶어서 기본키를 구성해야 한다.
    - null이나 중복 저장은 되지 않는다.
- 값 타입은 Entity와 다르게 식별자 개념이 없다.
    - 따라서 값을 변경하면 추적이 어렵다.
- 값 타입 컬렉션에 변경 사항이 발생하면
    - 주인 Entity와 연관된 모든 데이터를 삭제하고
    - 값 타입 컬렉션에 있는 현재 값을 모두 다시 저장한다.
- 따라서 값 타입 컬렉션은 사용하지 않는 게 좋다.

## 대안

{% tabs %} {% tab title="AddressEntity.java" %}

```java

@Entity
@Table(name = "ADDRESS")
public class AddressEntity {
    @Id
    @GeneratedValue
    private Long id;

    private Address address;
}
```

{% endtab %} {% tab title="Member.java" %}

```java

@Entity
public class Member {
    ...

    @OneToMany(cascade = ALL, orphanRemoval = true)
    @JoinColumn(name = "MEMBER_ID")
    private List<AddressEntity> addressHistory = new ArrayList<>();

}
```

{% endtab %} {% endtabs %}

- 실무에서는 상황에 따라 일대다 관계로 우회한다.
    - 일대다 관계를 위한 Entity를 만들어 값 타입을 wrapping 한다.
    - ex. Address라는 Entity를 만들 그 안에 Address라는 값 타입을 넣는다.
        - Member에는 Addresss 엔티티를 넣고 일대다 연관 관계를 맺는다.
        - 영속성 전이와 고아 객체 제거를 적용한다.
        - 이렇게 하면 실무에서 훨씬 유용하게 사용할 수 있다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/09/screenshot%202021-04-03%20오후%203.37.28.png)

- AddressEntity에 값 2개를 저장한다.
    - update 쿼리가 두 번 나가는 건 어쩔 수 없다.
    - 1:N 단방향은 다른 테이블에 외래키가 있기 때문이다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/09/screenshot%202021-04-03%20오후%203.39.02.png)

- 식별자가 있는 ADDRESS 테이블로 저장됐다.

## 활용

- 업데이트나 추적할 필요 없는 단순한 상황일 때 사용하면 된다.
    - ex. 셀렉트 박스에서 치킨, 피자, 족발 중 선택
    - 그게 아닌 이상 웬만하면 Entity로 사용한다.
- 꼭 값을 변경할 일이 없더라도 쿼리 자체를 그 테이블에서 할 때가 많다면 Entity로 하는 게 좋다.
    - ex. 주소 이력은 입력만 하지만 조회할 일이 많으므로 Entity로 만든다.

## 정리

### Entity 타입

- 식별자가 있다.
- 생명 주기를 따로 관리한다.
- 공유할 수 있다.

### 값 타입

- 식별자가 없다.
- 생명 주기를 Entity에 의존한다.
- 공유하지 않고 복사해서 사용한다.
- 불변 객체로 만들어야 안전하다.

값 타입은 정말 값 타입이라고 판단될 때만 사용한다. Entity와 값 타입을 혼동해서, Entity를 값 타입으로 만들면 안된다. 

**식별자가 필요**하고 지속해서 **값을 추적**해야 하며 **변경이 필요**하다면 값 타입이 아니라 Entity가 되어야 한다.