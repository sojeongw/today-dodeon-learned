# 벌크성 수정 쿼리

## 순수 JPA

{% tabs %} {% tab title="MemberJpaRepository.java" %}

```java

@Repository
public class MemberJpaRepository {

    public int bulkAgePlus(int age) {
        int resultCount = em.createQuery("update Member m set m.age = m.age + 1 where m.age >= :age")
                .setParameter("age", age)
                .executeUpdate();
        return resultCount;
    }
}
```

{% endtab %} {% tab title="MemberJpaRepositoryTest.java" %}

```java

@SpringBootTest
@Transactional
@Rollback(value = false)
class MemberJpaRepositoryTest {

    @Test
    public void bulkUpdate() throws Exception {
        memberJpaRepository.save(new Member("member1", 10));
        memberJpaRepository.save(new Member("member2", 19));
        memberJpaRepository.save(new Member("member3", 20));
        memberJpaRepository.save(new Member("member4", 21));
        memberJpaRepository.save(new Member("member5", 40));

        int resultCount = memberJpaRepository.bulkAgePlus(20);
        assertThat(resultCount).isEqualTo(3);
    }
}
```

{% endtab %} {% endtabs %}

```sql
update member
set age=age + 1
where age >= 20;
```

## 스프링 데이터 JPA

{% tabs %} {% tab title="MemberRepository.java" %}

```java
public interface MemberRepository extends JpaRepository<Member, Long> {

    @Modifying
    @Query("update Member m set m.age = m.age + 1 where m.age >= :age")
    int bulkAgePlus(@Param("age") int age);
}
```

{% endtab %} {% tab title="MemberRepositoryTest.java" %}

```java

@SpringBootTest
@Transactional
@Rollback(value = false)
class MemberRepositoryTest {

    @Test
    public void bulkUpdate() throws Exception {
        memberRepository.save(new Member("member1", 10));
        memberRepository.save(new Member("member2", 19));
        memberRepository.save(new Member("member3", 20));
        memberRepository.save(new Member("member4", 21));
        memberRepository.save(new Member("member5", 40));

        int resultCount = memberRepository.bulkAgePlus(20);
        assertThat(resultCount).isEqualTo(3);
    }
}
```

{% endtab %} {% endtabs %}

```sql
update member
set age=age + 1
where age >= 20;
```

- @Modifying
    - executeUpdate()를 실행한다.
    - 붙이지 않으면 getSingleResult(), getResultList()를 호출하면서 에러가 발생한다.

## 주의사항

- JPA는 영속성 컨텍스트에서 엔티티를 관리한다.
- 벌크형 연산은 이걸 무시하고 진행하기 때문에 문제가 발생할 수 있다.

```java

@SpringBootTest
@Transactional
@Rollback(value = false)
class MemberRepositoryTest {

    @Test
    public void bulkUpdate() throws Exception {
        memberRepository.save(new Member("member1", 10));
        memberRepository.save(new Member("member2", 19));
        memberRepository.save(new Member("member3", 20));
        memberRepository.save(new Member("member4", 21));
        memberRepository.save(new Member("member5", 40));

        // 나이를 한 살 더한다.
        int resultCount = memberRepository.bulkAgePlus(20);
        assertThat(resultCount).isEqualTo(3);

        // JPA 영속성 컨텍스트에서는 여전히 40살이다.
        Member member5 = memberRepository.findMembers("member5");
        // fail
        assertThat(member5.getAge()).isEqualTo(41);
    }
}
```

- 아직 DB에 반영이 되지 않은 상태에서 쿼리를 날려서 데이터가 서로 안맞을 수 있다.

```java

@SpringBootTest
@Transactional
@Rollback(value = false)
class MemberRepositoryTest {

    // 같은 트랜잭션이면 둘 다 같은 엔티티 매니저를 쓴다.
    @Autowired
    MemberRepository memberRepository;

    @PersistenceContext
    private EntityManager em;

    @Test
    public void bulkUpdate() throws Exception {
        memberRepository.save(new Member("member1", 10));
        memberRepository.save(new Member("member2", 19));
        memberRepository.save(new Member("member3", 20));
        memberRepository.save(new Member("member4", 21));
        memberRepository.save(new Member("member5", 40));

        int resultCount = memberRepository.bulkAgePlus(20);
        assertThat(resultCount).isEqualTo(3);

        // DB에 반영되지 않은 데이터를 반영한다.
        em.flush();
        // 영속성 컨텍스트의 데이터를 날려버린다.
        em.clear();

        Member member5 = memberRepository.findMembers("member5");
        assertThat(member5.getAge()).isEqualTo(41);
    }
}
```

- 벌크 연산 직후에는 꼭 영속성 컨텍스트를 날려준다.

```java
public interface MemberRepository extends JpaRepository<Member, Long> {

    @Modifying(clearAutomatically = true)
    @Query("update Member m set m.age = m.age + 1 where m.age >= :age")
    int bulkAgePlus(@Param("age") int age);
}
```

- @Modifying에 옵션을 주면 clear()를 자동으로 해준다.

### 참고

- JPA는 update 등의 쿼리가 있으면 영속성 컨텍스트에 있는 데이터를 먼저 flush 하고 JPQL을 실행한다.
    - memberRepository.bulkAgePlus(20)도 결국 JPQL이므로 똑같이 동작한다.
    - 따라서 memberRepository.save가 반영된 뒤 memberRepository.bulkAgePlus(20)를 실행한다.
- JPA 대신 JDBC Template을 직접 사용하는 경우는 해당되지 않는다.
