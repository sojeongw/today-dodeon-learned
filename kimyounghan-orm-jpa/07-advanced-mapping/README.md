# 고급 매핑

## 상속 관계 매핑

- 상속 관계 매핑
    - 객체의 상속 구조와 DB의 슈퍼 타입, 서브 타입 관계를 매핑하는 것
- 객체에는 상속이 있지만 관계형 데이터베이스는 없다.
    - 그나마 슈퍼타입, 서브타입 관계라는 모델링 기법이 객체 상속과 유사하다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/07/screenshot%202021-03-20%20오후%204.41.26.png)

- DB
    - 논리 모델과 물리 모델이 있다.
    - 논리 모델로 음반, 영화, 책을 구상한다.
    - 가격이나 이름 등 공통적인 속성은 물품에 두고 각각에 맞는 데이터는 아래에 둔다.
- 객체
    - 명확하게 상속 관계가 있다.
    - 아이템이라는 추상 타입을 만들고 그 아래에 상속 관계를 둔다.

슈퍼 타입, 서브 타입 논리 모델을 실제 물리 모델로 구현하려면 아래의 3가지 방법을 사용한다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/07/screenshot%202022-04-09%20오후%201.41.39.png)

1. 조인 전략
    - 각각의 테이블로 변환한다.
        - ITEM과 ALBUM 둘 다 데이터를 insert 한다.
        - 조회할 땐 두 테이블을 join 한다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/07/screenshot%202022-04-09%20오후%201.41.45.png)

2. 단일 테이블 전략
    - 통합 테이블로 변환한다.
        - 컬럼을 한 테이블에 다 몰아넣고 DTYPE에서 album인지 movie인지 구분한다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/07/screenshot%202022-04-09%20오후%201.41.51.png)

3. 구현 클래스마다 테이블을 구성하는 전략
    - album, move, book이 각각 따로 정보를 갖고 있는다.

## 주요 애너테이션

### @Inheritance

- JOINED
    - 조인 전략
- SINGLE_TABLE
    - 단일 테이블 전략
    - 별도의 설정이 없다면 JPA 기본 전략으로 들어간다.
- TABLE_PER_CLASS
    - 구현 클래스마다 테이블 전략

추후 요구 사항이 바뀌어도 @Inheritance의 옵션만 바꿔주면 되기 때문에 편리하다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/07/screenshot%202021-03-20%20오후%205.21.23.png)

JPA 기본 전략은 싱글 테이블이기 때문에 별도로 지정하지 않으면 `item` 테이블에 모든 데이터가 모인다.

{% tabs %} {% tab title="Item.java" %}

```java

@Entity
// 이 애너테이션이 없으면 기본 전략인 단일 테이블로 들어간다. 
// 애너테이션에 JOINED로 명시하면 조인 전략으로 바꿔준다.
@Inheritance(strategy = InheritanceType.JOINED)
public class Item {

    @Id
    @GeneratedValue
    private Long id;

    private String name;
    private int price;
}

```

{% endtab %} {% tab title="Movie.java" %}

```java

@Entity
public class Movie extends Item {

    private String director;
    private String actor;
    private String name;
}

```

{% endtab %}{% tab title="JpaMain.java" %}

```java
public class JpaMain {

    public static void main(String[] args) {

        Movie movie = new Movie();
        movie.setDirector("A");
        movie.setActor("B");
        movie.setName("바람과 함께 사라지다");
        movie.setPrice(10000);

        // 데이터는 각자 테이블에 저장하고
        entityManager.persist(movie);

        entityManager.flush();
        entityManager.clear();

        // 조회할 땐 Item 테이블과 조인해서 나간다.
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

각자의 테이블이 생성되었다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/07/screenshot%202021-03-20%20오후%205.31.22.png)

![](../../.gitbook/assets/kimyounghan-orm-jpa/07/screenshot%202021-03-20%20오후%205.34.06.png)

데이터를 저장할 때는 `item`과 `movie`에 동시에 insert를 날린다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/07/screenshot%202021-03-20%20오후%205.39.00.png)

조회할 때는 `item`과 inner join을 이용해서 가져온다.

### @DiscriminatorColumn

{% tabs %} {% tab title="Item.java" %}

```java

@Entity
@Inheritance(strategy = InheritanceType.JOINED)
// DTYPE이 생기면서 album, movie 등의 테이블 이름이 들어간다.
// item 데이터가 생겼을 때 이 데이터가 어디서 온 건지 알 수 있게 된다.
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

- `@DiscriminatorColumn`
    - `DTYPE`이 추가된다.
    - `DTYPE`이 없으면 어떤 게 생성된지 모르기 때문에 웬만하면 넣어주는 게 좋다.
    - `name` 옵션으로 컬럼 이름을 바꿔줄 수도 있다.
        - @DiscriminatorColumn(name = "DIS_TYPE")
- 단일 테이블 전략에서는 DTYPE이 꼭 필요하기 때문에 @DiscriminatorColumn이 없어도 자동으로 DTYPE을 만든다.

### @DiscriminatorValue

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

- @DiscriminatorValue("XXX")
    - `DTYPE`에 들어갈 자식의 값을 정해줄 수 있다.
    - ex. 자식인 `movie`가 `M`으로 표시되게 한다.