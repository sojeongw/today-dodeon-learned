# 상속 관계 매핑

객체에는 상속이 있지만 관계형 데이터베이스는 없다. 그나마 슈퍼타입, 서브타입 관계라는 모델링 기법이 객체 상속과 유사하다.

상속 관계 매핑이란 객체의 상속 구조와 DB의 슈퍼 타입 서브 타입 관계를 매핑하는 것이다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/07/screenshot%202021-03-20%20오후%204.41.26.png)

DB에는 논리 모델과 물리 모델이 있다. 왼쪽처럼 논리 모델로 음반, 영화, 책을 구상하면, 가격이나 이름같은 공통적인 속성은 물품에 두고 각각에 맞는 데이터는 아래에 둔다.

객체는 명확하게 상속 관계가 있다. 아이템이라는 추상 모델을 만들어두고 그 아래에 상속 관계로 가져가는 것이다.

슈퍼 타입, 서브 타입 논리 모델을 실제 물리 모델로 구현하려면 아래의 방법을 사용한다.

- 각각의 테이블로 변환한다.
    - 조인 전략
- 통합 테이블로 변환한다.
    - 단일 테이블 전략
- 서브 타입 테이블로 변환한다.
    - 구현 클래스마다 테이블 전략

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
public class Item {

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

## 단일 테이블 전략

![](../../.gitbook/assets/kimyounghan-orm-jpa/07/screenshot%202021-03-20%20오후%204.41.52.png)

논리 테이블을 하나로 합쳐버리는 방법이다. 모든 칼럼을 모아서 `DTYPE`에 어떤 물품인지 표시 해둔다.

{% tabs %} {% tab title="Item.java" %}

```java

@Entity
// 애너테이션에서 전략만 바꿔주면 된다.
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
public class Item {

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

![](../../.gitbook/assets/kimyounghan-orm-jpa/07/screenshot%202021-03-20%20오후%205.58.12.png)

해당 자식 엔티티와 관련이 없는 데이터는 `null`로 들어간다.

이 방식은 insert가 한 방에 들어가고 join 할 필요도 없기 때문에 성능상 이점이 있다.

## 구현 클래스마다 테이블 전략

![](../../.gitbook/assets/kimyounghan-orm-jpa/07/screenshot%202021-03-20%20오후%204.41.58.png)

그냥 각각의 테이블을 만드는 방법이다. 공통의 필드를 각각 가지고 있다.

---

객체 입장에서는 상속이 되기 때문에 뭘 해도 똑같다. JPA에서는 다 매핑이 가능하다.

## 주요 애너테이션

### @Inheritance(strategy = InheritanceType.XXX)

- JOINED
    - 조인 전략
- SINGLE_TABLE
    - 단일 테이블 전략
- TABLE_PER_CLASS
    - 구현 클래스마다 테이블 전략

{% tabs %} {% tab title="Item.java" %}

```java

@Entity
// 싱글 테이블에 다 모아두는 기본 전략에서 JOINED로 바꿔준다.
@Inheritance(strategy = InheritanceType.JOINED)
public class Item {

  @Id
  @GeneratedValue
  private Long id;

  private String name;
  private int price;
}

```

{% endtab %} {% tab title="JpaMain.java" %}

```java
public class JpaMain {

  public static void main(String[] args) {

    Movie movie = new Movie();
    movie.setDirector("A");
    movie.setActor("B");
    movie.setName("바람과 함께 사라지다");
    movie.setPrice(10000);

    entityManager.persist(movie);

    entityManager.flush();
    entityManager.clear();

    Movie findMovie = entityManager.find(Movie.class, movie.getId());
    System.out.println("result: " + findMovie);

    tx.commit();
  }
}

```

{% endtab %} {% endtabs %}

![](../../.gitbook/assets/kimyounghan-orm-jpa/07/screenshot%202021-03-20%20오후%205.26.23.png)

![](../../.gitbook/assets/kimyounghan-orm-jpa/07/screenshot%202021-03-20%20오후%205.26.34.png)

![](../../.gitbook/assets/kimyounghan-orm-jpa/07/screenshot%202021-03-20%20오후%205.27.57.png)

다이어그램과 똑같이 테이블이 생성되었다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/07/screenshot%202021-03-20%20오후%205.31.22.png)

![](../../.gitbook/assets/kimyounghan-orm-jpa/07/screenshot%202021-03-20%20오후%205.34.06.png)

데이터를 저장할 때는 `item`과 `movie`에 동시에 insert를 날린다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/07/screenshot%202021-03-20%20오후%205.39.00.png)

조회할 때는 `item`과 inner join을 이용해서 가져온다.

### @DiscriminatorColumn(name = "DTYPE")

{% tabs %} {% tab title="Item.java" %}

```java

@Entity
@Inheritance(strategy = InheritanceType.JOINED)
@DiscriminatorColumn
public class Item {

  @Id
  @GeneratedValue
  private Long id;

  private String name;
  private int price;
}

```

{% endtab %} {% endtabs %}

![](../../.gitbook/assets/kimyounghan-orm-jpa/07/screenshot%202021-03-20%20오후%205.43.27.png)

`@DiscriminatorColumn`을 추가하면 `DTYPE`이 추가된다. `DTYPE`이 없으면 어떤 게 생성된지 모르기 때문에 웬만하면 넣어주는 게 좋다. `name`
옵션으로 이름을 바꿔줄 수도 있다.

### @DiscriminatorValue("XXX")

`DTYPE`에 들어갈 자식의 값을 정해줄 수 있는 기능이다.

{% tabs %} {% tab title="Album.java" %}

```java

@Entity
@DiscriminatorValue("A")
public class Album extends Item {

  private String artist;
}
```

{% endtab %} {% tab title="Movie.java" %}

```java

@Entity
@DiscriminatorValue("M")
public class Movie extends Item {

  private String director;
  private String actor;
}
```

{% endtab %} {% tab title="Book.java" %}

```java

@Entity
@DiscriminatorValue("B")
public class Book extends Item {

  private String author;
  private String isbn;
}

```

{% endtab %} {% endtabs %}

![](../../.gitbook/assets/kimyounghan-orm-jpa/07/screenshot%202021-03-20%20오후%205.49.54.png)

자식인 `movie`가 `M`으로 들어간 걸 확인할 수 있다.

조인 전략에서는 이게 없어도 자식 테이블 쿼리하면 알 수 있지만 싱글 테이블 전략에서는 꼭 필요하다.