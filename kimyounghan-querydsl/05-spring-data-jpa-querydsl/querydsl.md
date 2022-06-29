# 스프링 데이터 JPA가 제공하는 Querydsl 기능

- 실무에서 사용하기에는 제약이 크기 때문에 간단하게 설명한다.

## QuerydslPredicateExecutor 인터페이스

[Reference](https://docs.spring.io/spring-data/jpa/docs/2.2.3.RELEASE/reference/html/#core.extensions.querydsl)

{% tabs %} {% tab title="QuerydslPredicateExecutor.java" %}

```java
public interface QuerydslPredicateExecutor<T> {
    Optional<T> findById(Predicate predicate);

    Iterable<T> findAll(Predicate predicate);

    long count(Predicate predicate);

    boolean exists(Predicate predicate);
}
```

{% endtab %} {% tab title="MemberRepository.java" %}

```java
 interface MemberRepository extends JpaRepository<User, Long>,
        QuerydslPredicateExecutor<User> {
}
```

{% endtab %}{% tab title="TestService.java" %}

```java
class TestService {

    public void test() {
        Iterable result = memberRepository.findAll(
                member.age.between(10, 40)
                        .and(member.username.eq("member1"))
        );
    }
}
```

{% endtab %} {% endtabs %}

- 묵시적 조인은 가능하지만 left 조인은 불가하다.
- 클라이언트가 Querydsl에 의존해야 한다.
    - 서비스 클래스가 Querydsl이라는 구현 기술에 의존하게 된다.
- 복잡한 실무 환경에서 쓰기엔 한계가 명확하다.
- Pageable, Sort를 모두 지원한다.

## Querydsl Web

[Reference](https://docs.spring.io/spring-data/jpa/docs/2.2.3.RELEASE/reference/html/#core.web.type-safe)

- 파라미터로 들어오는 값을 Querydsl에 매핑한다.
- 단순한 조건만 가능하다.
- 조건을 커스텀하는 기능이 복잡하고 명시적이지 않다.
- 컨트롤러가 Querydsl에 의존한다.
- 복잡한 실무에서 사용하기 힘들다.

## QuerydslRepositorySupport 리포지토리

- 스프링 데이터 JPA가 제공하는 페이징을 Querydsl로 편리하게 변환 가능하다.
    - Sort는 오류가 발생한다.
- from()으로 시작할 수 있다.
    - 최근엔 select()로 시작하는 것이 더 명시적이다.
- EntityManager를 제공한다.
- Querydsl 3.x만 대상이다.
- QueryFactory를 제공하지 않는다.