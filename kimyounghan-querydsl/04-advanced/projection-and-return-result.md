# 프로젝션과 결과 반환

- 프로젝션
    - select 할 대상을 지정하는 것

## 프로젝션 대상이 하나일 때

```java
class Projection {
    void example() {
        List<String> result = queryFactory
                .select(member.username)
                .from(member)
                .fetch();
    }
}
```

- 대상이 하나면 타입을 명확하게 지정할 수 있다.

## 프로젝션 대상이 둘일 때

### Tuple

```java
class Projection {
    void example() {
        List<Tuple> result = queryFactory
                .select(member.username, member.age)
                .from(member)
                .fetch();

        for (Tuple tuple : result) {
            // 프로젝션 한 데이터를 각각 꺼내서 사용하면 된다.
            String username = tuple.get(member.username);
            Integer age = tuple.get(member.age);

            System.out.println("username=" + username);
            System.out.println("age=" + age);
        }
    }
}
```

- 튜플
    - 결과가 여러 개일 때 담을 수 있도록 만든 Querydsl 객체
    - 리포지토리 계층에서만 사용하고 다른 레이어가 종속되지 않게 한다.
- 대상이 둘 이상이면 튜플이나 DTO로 조회해야 한다.

### 순수 JPA에서 DTO 조회

{% tabs %} {% tab title="MemberDto.java" %}

```java

@Data
public class MemberDto {
    private String username;
    private int age;

    public MemberDto() {
    }

    public MemberDto(String username, int age) {
        this.username = username;
        this.age = age;
    }
}
```

{% endtab %} {% endtabs %}

- 엔티티로 가져오면 필요하지 않은 데이터도 다 가져와야 하므로 필요한 데이터만 가져올 수 있게 DTO를 만든다.
- Querydsl이 일단 MemberDto를 만든 다음 작업하기 때문에 MemberDto에 디폴트 생성자가 꼭 필요하다.

{% tabs %} {% tab title="QuerydslBasicTest.java" %}

```java

@SpringBootTest
@Transactional
public class QuerydslBasicTest {

    @Test
    void findDtoByJPQL() {
        List<MemberDto> result = em.createQuery(
                        "select new study.querydsl.dto.MemberDto(m.username, m.age) " +
                                "from Member m", MemberDto.class)
                .getResultList();
    }
}


```

{% endtab %} {% endtabs %}

- select에 DTO 타입을 new 명령어로 명시해준다.
    - `select m from Member m` 이라고 하면 Member 엔티티를 조회하기 때문에 타입이 맞지 않는다.
- DTO의 패키지 일므을 다 적어야 해서 지저분하다.
- 생성자 방식만 지원한다.

### Querydsl 빈 생성

{% tabs %} {% tab title="프로퍼티 접근" %}

```java

@SpringBootTest
@Transactional
public class QuerydslBasicTest {

    @Test
    void findDtoBySetter() {
        List<MemberDto> result = queryFactory
                // bean이 setter로 주입해준다.
                .select(Projections.bean(
                        MemberDto.class,
                        member.username,
                        member.age))
                .from(member)
                .fetch();
    }
}


```

{% endtab %} {% tab title="필드 직접 접근" %}

```java

@SpringBootTest
@Transactional
public class QuerydslBasicTest {

    @Test
    void findDtoByField() {
        List<MemberDto> result = queryFactory
                // 필드에 바로 넣는다.
                .select(Projections.fields(MemberDto.class,
                        member.username,
                        member.age))
                .from(member)
                .fetch();
    }
}


```

{% endtab %} {% tab title="생성자 사용" %}

```java

@SpringBootTest
@Transactional
public class QuerydslBasicTest {

    @Test
    void findDtoByConstructor() {
        List<MemberDto> result = queryFactory
                // 생성자를 사용한다.
                .select(Projections.constructor(MemberDto.class,
                        member.username,
                        member.age))
                .from(member)
                .fetch();
    }
}

```

{% endtab %} {% endtabs %}

### 별칭이 다를 때

{% tabs %} {% tab title="UserDto.java" %}

```java

@Data
public class UserDto {
    private String name;
    private int age;
}
```

{% endtab %} {% tab title="QuerydslBasicTest.java" %}

```java

@SpringBootTest
@Transactional
public class QuerydslBasicTest {

    @Test
    void findDtoByConstructor() {
        QMember memberSub = new QMember("memberSub");

        List<MemberDto> result = queryFactory
                .select(Projections.constructor(MemberDto.class,
                        member.username,
                        member.age))
                .from(member)
                .fetch();
    }
}
```

{% endtab %} {% endtabs %}

- username인데 DTO에는 name으로 되어있어 일치하지 않을 경우 사용한다.

```
ExpressionUtils.as(source, alias)
```

- 필드나 서브 쿼리에 별칭을 적용한다.

```
username.as("memberName")
```

- 필드에 별칭을 적용한다.

## @QueryProjection

{% tabs %} {% tab title="MemberDto.java" %}

```java

@Data
public class MemberDto {
    private String username;
    private int age;

    public MemberDto() {
    }

    @QueryProjection
    public MemberDto(String username, int age) {
        this.username = username;
        this.age = age;
    }
}
```

{% endtab %} {% tab title="QMemberDto.java" %}

```java
/**
 * study.querydsl.dto.QMemberDto is a Querydsl Projection type for MemberDto
 */
@Generated("com.querydsl.codegen.DefaultProjectionSerializer")
public class QMemberDto extends ConstructorExpression<MemberDto> {

    private static final long serialVersionUID = 1356709634L;

    public QMemberDto(com.querydsl.core.types.Expression<String> username, com.querydsl.core.types.Expression<Integer> age) {
        super(MemberDto.class, new Class<?>[]{String.class, int.class}, username, age);
    }

}


```

{% endtab %} {% endtabs %}

- 생성자에 @QueryProjection을 붙인 뒤 compileQuerydsl 한다.
    - QMemberDto 클래스가 생성된다.

{% tabs %} {% tab title="QuerydslBasicTest.java" %}

```java

@SpringBootTest
@Transactional
public class QuerydslBasicTest {

    @Test
    void findDtoByQueryProjection() {
        List<MemberDto> result = queryFactory
                // DTO 클래스를 new로 바로 넣으면 된다.
                // 생성자로 만들기 때문에 타입과 파라미터를 다 맞춰줘서 안정적으로 만들 수 있다.
                .select(new QMemberDto(member.username, member.age))
                .from(member)
                .fetch();
    }
}


```

{% endtab %} {% endtabs %}

- 컴파일러로 타입을 체크할 수 있어 안전하다.
    - 생성자를 통한 빈 생성 방식은 필드를 다르게 넣어도 컴파일 시점에 잡을 수 없다.
- DTO에 Querydsl 애너테이션을 유지하고 Q 파일까지 만들어야 하는 단점이 있다.
    - Querydsl에 의존적인 설계가 된다.