# 페이징과 정렬

- 검색 조건
    - age = 10
- 정렬 조건
    - 이름으로 내림차순
- 페이징 조건
    - 첫 번째 페이지
    - 페이지 당 보여줄 데이터는 3건

## 순수 JPA

{% tabs %} {% tab title="MemberJpaRepository.java" %}

```java

@Repository
public class MemberJpaRepository {

    public List<Member> findByPage(int age, int offset, int limit) {
        return em.createQuery("select m from Member m where m.age = :age order by m.username desc")
                .setParameter("age", age)
                // 어디서 부터 가져올 것인지
                .setFirstResult(offset)
                // 몇 개를 가져올 것인지
                .setMaxResults(limit)
                .getResultList();
    }

    public long totalCount(int age) {
        // 단순 count니까 sort 조건은 빠진다.
        return em.createQuery("select count(m) from Member  m where m.age = :age", Long.class)
                .setParameter("age", age)
                .getSingleResult();
    }
}
```

{% endtab %} {% tab title="MemberJpaRepositoryTest.java" %}

```java

@SpringBootTest
@Transactional
@Rollback(value = false)
class MemberJpaRepositoryTest {

    @Autowired
    MemberJpaRepository memberJpaRepository;

    @Test
    void paging() {
        memberJpaRepository.save(new Member("member1", 10));
        memberJpaRepository.save(new Member("member2", 10));
        memberJpaRepository.save(new Member("member3", 10));
        memberJpaRepository.save(new Member("member4", 10));
        memberJpaRepository.save(new Member("member5", 10));

        int age = 10;
        int offset = 0;
        int limit = 3;

        List<Member> members = memberJpaRepository.findByPage(age, offset, limit);
        long totalCount = memberJpaRepository.totalCount(age);

        assertThat(members.size()).isEqualTo(3);
        assertThat(totalCount).isEqualTo(5);
    }
}
```

{% endtab %} {% endtabs %}

- DB가 달라져도 JPA가 그 DB에 맞는 방언으로 쿼리를 날린다.

## 스프링 데이터 JPA

### 파라미터

- org.springframework.data.domain.Sort
    - 정렬
- org.springframework.data.domain.Pageable
    - 페이징
    - 내부에 sort가 포함되어 있다.

### 반환 타입

```java
interface MemberRepository extends JpaRepository<Member, Long> {
    // count 쿼리 사용
    Page<Member> findByUsername(String name, Pageable pageable);

    // count 쿼리 사용 안함
    Slice<Member> findByUsername(String name, Pageable pageable);

    // count 쿼리 사용 안함
    List<Member> findByUsername(String name, Pageable pageable);

    // count 쿼리 사용 안함
    List<Member> findByUsername(String name, Sort sort);
}
```

- org.springframework.data.domain.Page
    - 페이징과 total count 쿼리가 같이 나간다.
- org.springframework.data.domain.Slice
    - total count 없이 해당 페이지만 가져온다.
    - 내부적으로 limit + 1만큼 조회한다.
- List
    - total count 쿼리 없이 결과만 반환한다.

## 예제

### Page

{% tabs %} {% tab title="MemberRepository.java" %}

```java
public interface MemberRepository extends JpaRepository<Member, Long> {

    Page<Member> findByAge(int age, Pageable pageable);

}

```

{% endtab %} {% tab title="MemberRepositoryTest.java" %}

```java

@SpringBootTest
@Transactional
@Rollback(value = false)
class MemberRepositoryTest {

    @Test
    void paging() {
        memberRepository.save(new Member("member1", 10));
        memberRepository.save(new Member("member2", 10));
        memberRepository.save(new Member("member3", 10));
        memberRepository.save(new Member("member4", 10));
        memberRepository.save(new Member("member5", 10));

        // 페이징은 0부터 시작한다.
        PageRequest pageRequest = PageRequest.of(0, 3, Sort.by(DESC, "username"));
        int age = 10;

        Page<Member> page = memberRepository.findByAge(age, pageRequest);

        // 조회된 데이터
        List<Member> content = page.getContent();
        // 조회된 데이터 수
        assertThat(content.size()).isEqualTo(3);
        // 전체 데이터 수
        assertThat(page.getTotalElements()).isEqualTo(5);
        // 페이지 번호
        assertThat(page.getNumber()).isEqualTo(0);
        // 전체 페이지 번호
        assertThat(page.getTotalPages()).isEqualTo(2);
        // 첫번째 항목인가?
        assertThat(page.isFirst()).isTrue();
        // 다음 페이지가 있는가?
        assertThat(page.hasNext()).isTrue();
    }
}
```

{% endtab %} {% endtabs %}

```sql
select member0_.member_id as member_i1_0_,
       member0_.age       as age2_0_,
       member0_.team_id   as team_id4_0_,
       member0_.username  as username3_0_
from member member0_
where member0_.age = 10
order by member0_.username desc limit 3;

select count(member0_.member_id) as col_0_0_
from member member0_
where member0_.age = 10;
```

- count를 구하는 별도의 메서드 없이 자동으로 날린다.
- PageRequest
    - Pageable 인터페이스를 구현한 객체

### Slice

{% tabs %} {% tab title="MemberRepository.java" %}

```java
public interface MemberRepository extends JpaRepository<Member, Long> {

    Slice<Member> findByAge(int age, Pageable pageable);

}
```

{% endtab %} {% tab title="MemberRepositoryTest.java" %}

```java

@SpringBootTest
@Transactional
@Rollback(value = false)
class MemberRepositoryTest {

    @Test
    void slice() {
        memberRepository.save(new Member("member1", 10));
        memberRepository.save(new Member("member2", 10));
        memberRepository.save(new Member("member3", 10));
        memberRepository.save(new Member("member4", 10));
        memberRepository.save(new Member("member5", 10));

        PageRequest pageRequest = PageRequest.of(0, 3, Sort.by(DESC, "username"));
        int age = 10;

        Slice<Member> page = memberRepository.findByAge(age, pageRequest);

        // 조회된 데이터
        List<Member> content = page.getContent();
        // 조회된 데이터 수
        assertThat(content.size()).isEqualTo(3);
        // 페이지 번호
        assertThat(page.getNumber()).isEqualTo(0);
        // 첫번째 항목인가?
        assertThat(page.isFirst()).isTrue();
        // 다음 페이지가 있는가?
        assertThat(page.hasNext()).isTrue();
    }
}
```

{% endtab %} {% endtabs %}

```sql
select member0_.member_id as member_i1_0_,
       member0_.age       as age2_0_,
       member0_.team_id   as team_id4_0_,
       member0_.username  as username3_0_
from member member0_
where member0_.age = 10
order by member0_.username desc limit 4;
```

- count를 가져오지 않는다.
- 컨텐츠만 limit + 1만큼 가져온다.
- 더보기 방식으로 개발할 때 사용한다.

### List

{% tabs %} {% tab title="MemberRepository.java" %}

```java
public interface MemberRepository extends JpaRepository<Member, Long> {

    List<Member> findByAge(int age, Pageable pageable);

}
```

{% endtab %} {% tab title="MemberRepositoryTest.java" %}

```java

@SpringBootTest
@Transactional
@Rollback(value = false)
class MemberRepositoryTest {

    @Test
    void list() {
        memberRepository.save(new Member("member1", 10));
        memberRepository.save(new Member("member2", 10));
        memberRepository.save(new Member("member3", 10));
        memberRepository.save(new Member("member4", 10));
        memberRepository.save(new Member("member5", 10));

        PageRequest pageRequest = PageRequest.of(0, 3, Sort.by(DESC, "username"));
        int age = 10;

        List<Member> page = memberRepository.findByAge(age, pageRequest);
    }
}
```

{% endtab %} {% endtabs %}

```sql
select member0_.member_id as member_i1_0_,
       member0_.age       as age2_0_,
       member0_.team_id   as team_id4_0_,
       member0_.username  as username3_0_
from member member0_
where member0_.age = 10
order by member0_.username desc limit 3;
```

## Count 최적화

```java
public interface MemberRepository extends JpaRepository<Member, Long> {

    @Query(value = "select m from Member m left join m.team t")
    Page<Member> findByAge(int age, Pageable pageable);

}
```

```sql
select member0_.member_id as member_i1_0_,
       member0_.age       as age2_0_,
       member0_.team_id   as team_id4_0_,
       member0_.username  as username3_0_
from member member0_
         left outer join team team1_ on member0_.team_id = team1_.team_id
order by member0_.username desc limit 3;

select count(member0_.member_id) as col_0_0_
from member member0_
         left outer join team team1_ on member0_.team_id = team1_.team_id;
```

- count 쿼리는 매번 총 개수를 세야 해서 부하가 생긴다.
- join으로 가져오는 데이터라면 count 쿼리에 불필요한 join이 나간다.

```java
public interface MemberRepository extends JpaRepository<Member, Long> {

    @Query(value = "select m from Member m left join m.team t", countQuery = "select count(m.username) from Member m")
    Page<Member> findByAge(int age, Pageable pageable);

}
```

```sql
select member0_.member_id as member_i1_0_,
       member0_.age       as age2_0_,
       member0_.team_id   as team_id4_0_,
       member0_.username  as username3_0_
from member member0_
         left outer join team team1_ on member0_.team_id = team1_.team_id
order by member0_.username desc limit 3;

select count(member0_.username) as col_0_0_
from member member0_;
```

- countQuery를 사용하면 count 쿼리가 심플하게 나간다.

## 페이지를 유지하면서 엔티티를 DTO로 변환

```java

@SpringBootTest
@Transactional
@Rollback(value = false)
class MemberRepositoryTest {
    @Test
    void paging() {
        
        ...

        Page<Member> page = memberRepository.findByAge(10, pageRequest);
        Page<MemberDto> dtoPage = page.map(m -> new MemberDto(m.getId(), m.getUsername()));

      ...
    }
}
```

- map
    - 엔티티 그대로 컨트롤러에서 넘기면 안되므로 DTO로 변환한다.
