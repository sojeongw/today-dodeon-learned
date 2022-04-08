# 실전 예제

- 상품의 종류는 음반, 도서, 영화가 있고 이후 더 확장될 수 있다.
- 모든 데이터는 등록일과 수정일이 필수다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/07/screenshot%202021-05-16%20오후%208.24.02.png)

![](../../.gitbook/assets/kimyounghan-orm-jpa/07/screenshot%202021-05-16%20오후%208.23.49.png)

- 상품과 도서, 음반, 영화를 상속 관계로 만든다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/07/screenshot%202021-05-16%20오후%208.24.08.png)

- DB는 단일 테이블 전략을 사용한다.
- Book 데이터를 저장하면 Item 테이블에 쿼리가 들어간다.
    - Album 등 다른 데이터는 null로 세팅된다.

{% tabs %} {% tab title="Item.java" %}

```java

@Entity
// 단일 테이블 전략을 사용한다.
@Inheritance(strategy = SINGLE_TABLE)
// DTYPE으로 데이터를 구분한다.
@DiscriminatorColumn
// 아이템만 단독으로 테이블에 저장할 일이 없다고 가정하고 추상 클래스로 선언한다.
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

{% endtab %} {% tab title="Album.java" %}

```java

@Entity
public class Album extends Item {

    private String artist;
    private String etc;
}
```

{% endtab %} {% tab title="Book.java" %}

```java

@Entity
public class Book extends Item {

    private String author;
    private String isbn;
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

{% endtab %} {% endtabs %}

![](../../.gitbook/assets/kimyounghan-orm-jpa/07/screenshot%202021-05-16%20오후%208.33.35.png)

- Item 테이블에 모든 데이터가 모아지고 DTYPE으로 구분한다.

{% tabs %} {% tab title="BaseEntity.java" %}

```java

@MappedSuperclass
public abstract class BaseEntity {

    private String createdBy;
    private LocalDateTime createdDate;
    private String lastModifiedBy;
    private LocalDateTime lastModifiedDate;
}
```

{% endtab %} {% tab title="Item.java" %}

```java

@Entity
@Inheritance(strategy = SINGLE_TABLE)
@DiscriminatorColumn
// BaseEntity를 상속한다.
public abstract class Item extends BaseEntity {
    ...
}
```

{% endtab %} {% endtabs %}

![](../../.gitbook/assets/kimyounghan-orm-jpa/07/screenshot%202021-05-16%20오후%208.40.17.png)

createdBy 등의 칼럼이 추가되었다.

## 실무에서도 상속을 적용할 수 있을까?

관리하기 복잡하고 데이터가 많다면 사용할 수도 있다. 일단 객체지향적으로 설계 한 뒤 트레이드 오프를 감안해 상황에 따라 적용하면 된다.