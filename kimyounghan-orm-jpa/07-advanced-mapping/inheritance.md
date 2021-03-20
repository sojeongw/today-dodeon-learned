# 상속 관계 매핑

## 조인 전략

![](../../.gitbook/assets/kimyounghan-orm-jpa/07/screenshot%202021-03-20%20오후%204.41.45.png)

`item`이라는 테이블을 만들고 `album`, `movie`, `book` 테이블을 만든 다음 조인으로 구성하는 방법이다.

공통의 데이터는 `item`에 들어가고 각각의 정보는 `album`, `movie`, `book`에 들어가서 두 개의 insert가 일어난다. 그후 PK로 조인을 해서 데이터를
가져오게 된다.

`item`만 보면 그게 무엇인지 모르기 때문에 구분하는 칼럼을 둔다. 여기서는 `DTYPE`에 표시를 해둔다(칼럼 이름은 자유).

데이터가 굉장히 정규화된 모델이다.

{% tabs %} {% tab title="Item.java" %}

```java

@Entity
public abstract class Item {

  @Id
  @GeneratedValue
  private Long id;

  private String name;
  private int price;
}
```

{% endtab %} {% tab title="Album.java" %}

```java

@Entity
public class Album extends Item {

  private String artist;
}
```

{% endtab %} {% tab title="Movie.java" %}

```java

@Entity
public class Movie extends Item {

  private String director;
  private String actor;
}

```

{% endtab %}{% tab title="Book.java" %}

```java

@Entity
public class Book extends Item {

  private String author;
  private String isbn;
}
```

{% endtab %}{% endtabs %}

![](../../.gitbook/assets/kimyounghan-orm-jpa/07/screenshot%202021-03-20%20오후%205.21.23.png)

`item` 테이블에 모든 데이터가 모여서 생성되었다. JPA 기본 전략 자체가 싱글 테이블에 다 때려박는 방식이기 때문이다.

### 장점

- 테이블이 정규화 되어있다.
- 외래 키 참조 무결성 제약 조건을 활용할 수 있다.
    - 주문 테이블에서 아이템을 보고 싶으면 `item_id`로 `item`만 보면 된다. 설계가 깔끔해진다.
- 정규화 덕분에 저장 공간을 효율적으로 사용한다.

### 단점

- 조회 시 join을 많이 사용해서 성능이 저하된다.
- 조회 쿼리가 복잡하다.
- 데이터를 저장할 때 insert 쿼리를 2번 호출한다.

join도 잘 하면 성능이 좋아지기 때문에 크게 성능을 저하시키지는 않는다. 기본적으로 조인 전략이 정석이라고 생각하고 들어가면 된다. 객체 지향과도 맞고 설계가 깔끔하기 때문이다.

## 단일 테이블 전략

![](../../.gitbook/assets/kimyounghan-orm-jpa/07/screenshot%202021-03-20%20오후%204.41.52.png)

논리 테이블을 하나로 합쳐버리는 방법이다. 모든 칼럼을 모아서 `DTYPE`에 어떤 물품인지 표시 해둔다.

{% tabs %} {% tab title="Item.java" %}

```java

@Entity
// 애너테이션에서 전략만 바꿔주면 된다.
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
public abstract class Item {

  @Id
  @GeneratedValue
  private Long id;

  private String name;
  private int price;
}
```

{% endtab %} {% endtabs %}

![](../../.gitbook/assets/kimyounghan-orm-jpa/07/screenshot%202021-03-20%20오후%205.55.42.png)

모든 필드가 `item` 테이블에 합쳐졌다. 자식 테이블은 생성되지 않는다.

`@DiscriminatorColumn`을 넣지 않아도 `DTYPE`이 필수로 생성된다. 원래 JPA에서는 조인 전략에서도 `DTYPE`을 만들도록 되어있는데, 하이버테이트에서
생략한다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/07/screenshot%202021-03-20%20오후%205.58.12.png)

해당 자식 엔티티와 관련이 없는 데이터는 `null`로 들어간다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/07/screenshot%202021-03-20%20오후%206.06.58.png)

이 방식은 insert가 한 방에 들어가고 join 할 필요도 없기 때문에 성능상 이점이 있다.

### 장점

- 조인이 필요 없어서 조회 성능이 빠르다.
- 조회 쿼리가 단순하다.

### 단점

- 자식 엔티티가 매핑한 컬럼은 모두 null을 허용해야 한다.
    - 데이터 무결성 측면에서 애매하다.
- 단일 테이블에 모든 것을 저장하므로 테이블이 커질 수 있다.
    - 상황에 따라 조회 성능이 오히려 느려질 수 있다.

## 구현 클래스마다 단일 테이블 전략

![](../../.gitbook/assets/kimyounghan-orm-jpa/07/screenshot%202021-03-20%20오후%204.41.58.png)

그냥 `item` 테이블을 없애고 각각의 테이블을 만들어 중복 속성을 허용하는 방법이다.

{% tabs %} {% tab title="Item.java" %}

```java

@Entity
@Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)
public abstract class Item {

  @Id
  @GeneratedValue
  private Long id;

  private String name;
  private int price;
}
```

{% endtab %} {% endtabs %}

![](../../.gitbook/assets/kimyounghan-orm-jpa/07/screenshot%202021-03-20%20오후%206.13.02.png)

`item`은 테이블이 생성되지 않았다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/07/screenshot%202021-03-20%20오후%206.13.11.png)

JPA가 알아서 단일 테이블로 조회한다. 이때는 `DTYPE`이 필요 없으므로 `@DiscriminatorColumn`를 붙여도 적용되지 않는다.

{% tabs %} {% tab title="JpaMain.java" %}

```java
public class JpaMain {

  public static void main(String[] args) {
    // 상속했기 때문에 부모 클래스로도 조회할 수 있어야 한다.
    Item item = em.find(Item.class, movie.getId());
    System.out.println("item: " + item);
  }
}
```

{% endtab %} {% endtabs %}

![](../../.gitbook/assets/kimyounghan-orm-jpa/07/screenshot%202021-03-20%20오후%206.17.44.png)

이 전략의 문제는 부모 클래스로 조회할 경우 union을 이용해 모든 테이블에 데이터가 있는지 다 뒤져본다는 것이다. `item_id`로 책인지, 앨범인지, 영화인지 다 찾아봐야 하기 때문이다.

데이터베이스 설계자와 ORM 전문가 모두 추천하지 않는 전략이다.

### 장점

- 서브 타입을 명확하게 구분해서 처리할 때 효과적이다.
- `not null` 제약 조건을 사용할 수 있다.

### 단점

- 여러 자식 테이블을 함께 조회할 때 union 쿼리가 필요해 성능이 느리다.
- 자식 테이블을 통합해서 쿼리하기 어렵다.
    - 수정 사항이 생길 때마다 각 테이블을 다 수정해야 한다.

---

객체 입장에서는 상속이 되기 때문에 뭘 해도 똑같다. JPA에서는 다 매핑이 가능하다.

지금까지 DB 설계는 바뀌는데 코드는 손대지 않고 애너테이션의 전략만 바꿨다. JPA를 쓰지 않았다면 쿼리 등 많은 곳을 고쳐야 했을 것이다.