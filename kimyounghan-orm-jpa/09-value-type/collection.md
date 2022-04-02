# 값 타입 컬렉션

값 타입을 컬렉션에 담아서 사용하는 것을 말한다. DB는 컬렉션을 넣을 수 있는 타입이 없기 때문에 컬렉션을 저장하기 위한 별도의 테이블이 필요하다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/09/screenshot%202021-04-03%20오후%201.54.45.png)

테이블로 보면 이렇게 매핑이 된다. 값 타입은 식별자를 따로 두지 않고 값들을 묶어서 PK를 만든다. 별도의 식별자 ID 개념을 넣어서 PK를 쓰게 되면 값 타입이 아니라 Entity가 되기 때문이다.

이렇게 값 타입을 하나 이상 저장할 때는 `@ElementCollection`, `@CollectionTable`을 사용한다. 

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

  @ElementCollection
  // 값을 넣을 테이블 이름을 정의하고 외래키를 잡는다.
  @CollectionTable(name = "FAVORITE_FOOD", joinColumns = @JoinColumn(name = "MEMBER_ID"))
  // 값이 String 하나이고 내가 정의한 타입이 아니기 때문에 예외적으로 칼럼 이름을 지정해준다.
  @Column(name = "FOOD_NAME")
  private Set<String> favoriteFoods = new HashSet<>();

  @ElementCollection
  @CollectionTable(name = "ADDRESS", joinColumns = @JoinColumn(name = "MEMBER_ID"))
  private List<Address> addressHistory = new ArrayList<>();
}
```

{% endtab %} {% endtabs %}

![](../../.gitbook/assets/kimyounghan-orm-jpa/09/screenshot%202021-04-03%20오후%202.26.03.png)

임베디드 타입인 주소가 추가되었다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/09/screenshot%202021-04-03%20오후%202.25.50.png)

![](../../.gitbook/assets/kimyounghan-orm-jpa/09/screenshot%202021-04-03%20오후%202.25.39.png)

지정한 이름대로 테이블이 생성되었고 `MEMBER_ID`가 PK, FK가 되었다. 어디에 속하는지 알아야 하기 때문에 FK는 반드시 필요하다.

## 값 타입 저장

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

    tx.commit();
  }
}
```

{% endtab %} {% endtabs %}

![](../../.gitbook/assets/kimyounghan-orm-jpa/09/screenshot%202021-04-03%20오후%202.35.39.png)

처음에 `Member`를 생성한 다음 `AddressHistory` 2개, `FavoriteFood` 3개가 insert된 것을 볼 수 있다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/09/screenshot%202021-04-03%20오후%202.40.05.png)

해당 멤버에 잘 매핑되었다.

값 타입 컬렉션을 따로 persist 하지 않고 `member`만 persist했는데도 반영되었다. `member`와 라이프사이클을 같이 한 것이다. 값 타입은 `member`에 의존하기 때문에 `member`가 변경되면 같이 변경된다. 즉, `cascade`와 `orphanRemoval`을 모두 켠 상태를 필수로 가지고 있다고 볼 수 있다.

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

    // DB에는 데이터가 있고 애플리케이션을 아예 처음부터 시작하는 것처럼 실행할 수 있다.
    em.flush();
    em.clear();

    Member findMember = em.find(Member.class, member.getId());

    tx.commit();
  }
}
```

{% endtab %} {% endtabs %}

![](../../.gitbook/assets/kimyounghan-orm-jpa/09/screenshot%202021-04-03%20오후%202.44.59.png)

`member`를 조회하면 값 타입 컬렉션은 조회되지 않는다. 임베디드 타입은 소속된 값이기 때문에 `homeAddress`만 불러와진다. 이 말은 값 타입 컬렉션이 지연 로딩이라는 뜻이다.

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
    for(Address address : addressHistory) {
      System.out.println("address = " + address.getCity());
    }

    tx.commit();
  }
}
```

{% endtab %} {% endtabs %}

![](../../.gitbook/assets/kimyounghan-orm-jpa/09/screenshot%202021-04-03%20오후%203.00.56.png)

실제 조회 시점이 되어서야 가져오는 것을 볼 수 있다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/09/screenshot%202021-04-03%20오후%203.02.34.png)

`@ElementCollection`을 보면 기본 fetch 전략이 지연 로딩으로 되어있다.

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

    // 값 타입은 immutable 해야 하기 때문에 이렇게 변경하면 절대 안된다.
//      findMember.getHomeAddress().setCity("new city");

    // address 인스턴스 자체를 갈아끼워야 한다.
    Address a = findMember.getHomeAddress();
    findMember.setHomeAddress(new Address("new city", a.getStreet(), a.getZipcode()));

    // food는 단순 String이기 때문에 지우고 넣어야 한다.
    // String 자체가 값 타입이기 때문에 통으로 갈아 끼우는 것 말고는 방법이 없다. 따로 업데이트가 불가하다.
    findMember.getFavoriteFoods().remove("치킨");
    findMember.getFavoriteFoods().add("한식");

    tx.commit();
  }
}
```

{% endtab %} {% endtabs %}

값 타입은 불변이어야 하므로 꼭 인스턴스를 통으로 갈아 끼워서 수정해야 한다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/09/screenshot%202021-04-03%20오후%203.11.01.png)

조회 후 업데이트와 삭제, 추가 쿼리가 잘 동작한 것을 볼 수 있다.

컬렉션의 값만 변경해도 어떤 값이 변경되었는지 알고 JPA가 알아서 DB에 쿼리를 날려준다. 마치 영속성 전이가 된 것처럼 동작하는 것이다. 값 타입 컬렉션은 `member`로 라이프 사이클이 관리되는 단순한 속성이기 때문이다.

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

    // address를 하나만 바꾸고 싶다면?
    // 기본적으로 컬렉션은 대부분 대상을 찾을 때 equals()로 동작한다.
    // 그래서 찾고 싶은 값을 그대로 넣어주면 그 값을 찾아준다.
    // 따라서 equals()를 재정의 하지 않았다면 그냥 망하는 것이다. equals()를 꼭 재정의 해주자.
    findMember.getAddressHistory().remove(new Address("old1", "street", "12345"));
    // 지운 값 대신 새로운 값을 넣어준다.
    findMember.getAddressHistory().add(new Address("new city", "street", "12345"));

    tx.commit();
  }
}
```

{% endtab %} {% endtabs %}

`addressHistory`에서 `old1`만 수정하고 싶다면 해당 값을 찾아서 지우고 다시 넣는다. 컬렉션은 찾을 때 기본적으로 `equals()`를 사용하므로 반드시 오버라이딩이 필요하다.  

![](../../.gitbook/assets/kimyounghan-orm-jpa/09/screenshot%202021-04-03%20오후%203.21.19.png)

![](../../.gitbook/assets/kimyounghan-orm-jpa/09/screenshot%202021-04-03%20오후%203.24.49.png)

그런데 쿼리를 확인해보니 insert문이 2개가 나갔다. 테이블에 있는 데이터를 완전히 갈아끼우는 것이기 때문에 해당 `member_id`에 해당하는 `address` 테이블 값을 다 지우고, 남아있던 `old2`와 새 주소를 insert 했기 때문이다.

## 제약 사항

- 값 타입은 Entity와 다르게 식별자 개념이 없다. 따라서 값을 변경하면 추적이 어렵다.
- 값 타입 컬렉션에 변경 사항이 발생하면, 주인 Entity와 연관된 모든 데이터를 삭제하고 값 타입 컬렉션이 있는 현재 값을 모두 다시 저장한다.
- 값 타입 컬렉션을 매핑하는 테이블은 모든 칼럼을 묶어서 기본키를 구성해야 한다. null이나 중복 저장은 되지 않는다.

따라서 값 타입 컬렉션을 자제해야 한다.

## 대안

- 실무에서는 상황에 따라 일대다 관계로 우회한다.
- 일대다 관계를 위한 Entity를 만들고 여기에서 값 타입을 사용한다.
- 영속성 전이 + 고아 객체 제거를 사용해 값 타입 컬렉션처럼 사용한다.

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

  // 이렇게 사용해야 쿼리 최적화도 잘 되고 활용도가 높아진다.
  @OneToMany(cascade = CascadeType.ALL, orphanRemoval = true)
  @JoinColumn(name = "MEMBER_ID")
  private List<AddressEntity> addressHistory = new ArrayList<>();
}
```

{% endtab %} {% endtabs %}

![](../../.gitbook/assets/kimyounghan-orm-jpa/09/screenshot%202021-04-03%20오후%203.37.28.png)

참고로 update 쿼리가 두 번 나가는 건 어쩔 수 없다. 1:N 단방향은 다른 테이블에 외래키가 있기 때문이다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/09/screenshot%202021-04-03%20오후%203.39.02.png)

이젠 `address`가 식별자가 있는 Entity로 저장된다.

---

그럼 값 타입 컬렉션은 어떨 때 사용할까? 셀렉트 박스에서 멀티로 선택해야 하는, 업데이트나 추적할 필요 없는 단순한 상황일 때 사용하면 된다. 그게 아닌 이상 웬만하면 Entity로 사용한다. 꼭 값을 변경할 일이 없더라도 쿼리 자체를 그 테이블부터 시작해야 할 때가 있다면 Entity로 하는 게 좋다. 예를 들어 주소 이력은 입력만 하지만 조회할 일이 많으므로 Entity로 만든다. 

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

값 타입은 정말 값 타입이라고 판단될 때만 사용한다. Entity와 값 타입을 혼동해서, Entity를 값 타입으로 만들면 안된다. **식별자가 필요**하고 지속해서 **값을 추적**해야 하며 **변경이 필요**하다면 값 타입이 아니라 Entity가 되어야 한다.