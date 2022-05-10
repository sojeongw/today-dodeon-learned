# 사용자 정의 리포지토리

- 스프링 데이터 JPA의 리포지토리는 인터페이스만 정의하면 스프링이 구현체를 자동으로 생성한다.
    - 인터페이스를 직접 구현하면 부모의 기능까지 합쳐서 구현해야 하는 기능이 너무 많아진다.
- 그럼에도 불구하고 직접 구현하고 싶다면
    - EntityManager로 JPA 직접 사용
    - 스프링 JDBC Template 사용
    - MyBatis 사용
    - 데이터베이서 커넥션 직접 사용
    - QueryDsl 사용

## 예제

{% tabs %} {% tab title="MemberRepositoryCustom.java" %}

```java
public interface MemberRepositoryCustom {
    List<Member> findMemberCustom();
}
```

{% endtab %} {% tab title="MemberRepositoryImpl.java" %}

```java

@RequiredArgsConstructor
public class MemberRepositoryImpl implements MemberRepositoryCustom {

    // 생성자에 하나만 있으면 알아서 injection 해준다.
    private final EntityManager em;

    @Override
    public List<Member> findMemberCustom() {
        return em.createQuery("select m from Member m")
                .getResultList();
    }
}

```

{% endtab %} {% tab title="MemberRepository.java" %}

```java
public interface MemberRepository extends JpaRepository<Member, Long>, MemberRepositoryCustom {
}
```

{% endtab %} {% endtabs %}

- 인터페이스에 대한 실제 구현은 MemberRepositoryImpl에서 한다.
- MemberRepository가 인터페이스를 상속한다.

```java
class MemberRepositoryTest {
    
    ...

    @Test
    void callCustom() {
        List<Member> result = memberRepository.findMemberCustom();
    }
}
```

- 호출될 때는 실제 구현인 MemberRepositoryImpl의 로직을 실행한다.

## 규칙

- 구현 클래스 이름
    - 리포지토리 인터페이스 이름 + Impl
    - 스프링 데이터 JPA가 이 이름으로 인식해서 스프링 빈으로 자동 등록한다.
- MemberRepositoryCustom등 인터페이스는 자유롭게 지을 수 있다.

### 규칙 변경

- 억지로 바꾸면 다른 개발자가 혼란스러울 수 있으니 웬만하면 바꾸지 않는다.

**XML**

```xml

<repositories base-package="study.datajpa.repository"
              repository-impl-postfix="Impl"/>
```

**JavaConfig**

```java
@EnableJpaRepositories(basePackages = "study.datajpa.repository",
        repositoryImplementationPostfix = "Impl")
```

## 참고

- 실무에서 QueryDSL이나 SpringJDBCTemplate을 사용할 때 사용한다.
- 항상 사용자 정의 리포지토리가 필요한 것은 아니다.
    - 핵심 비즈니스를 처리하는 로직과 화면에 맞춘 복잡한 쿼리는 클래스를 분리하는 게 좋다.
    - 화면에 맞춘 쿼리는 이해만 하기도 힘들기 때문에 로직이 한 군데에 몰리면 개발자가 혼란스러워 한다.
    - 커스텀 리포지토리를 배웠다고 해서 여기에 다 몰아넣는 실수를 하지 말자.

```java

@Repository
@RequiredArgsConstructor
public class MemberQueryRepository {

    private final EntityManager entityManager;

    List<Member> findAllMembers() {
        List result = entityManager.createQuery("")
                .getResultList();

        return result;
    }
}
```

- 인터페이스가 아닌 클래스로 만들어 빈으로 등록하고 직접 사용해도 된다.
    - 물론 이때는 스프링 데이터 JPA와는 아무 관계 없이 별도로 동작한다.
    - 핵심은 비즈니스 로직과 그렇지 않은 것을 분리해야 한다는 것!

## 최신 구현 방식

```java

@RequiredArgsConstructor
public class MemberRepositoryCustomImpl implements MemberRepositoryCustom {

    private final EntityManager em;

    @Override
    public List<Member> findMemberCustom() {
        return em.createQuery("select m from Member m")
                .getResultList();
    }
}
```

- 스프링 데이터 2.x 이상에서 사용자 정의 구현 클래스 이름
    - MemberRepositoryImpl 등 리포지토리 인터페이스 이름 + Impl 대신
    - MemberRepositoryCustomImpl 등 사용자 정의 인터페이스 이름 + Impl도 지원한다.
- 사용자 정의 인터페이스와 구현 클래스 이름이 비슷하므로 기존 방식보다 직관적이다.
- 여러 인터페이스를 분리해서 구현하는 것도 가능해지므로 이 방식을 권장한다.
        