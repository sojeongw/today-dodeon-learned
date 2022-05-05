# 반환 타입

```java
public interface MemberRepository extends JpaRepository<Member, Long> {

    // 컬렉션
    List<Member> findByUsername(String name);

    // 단건
    Member findByUsername(String name);

    // 단건 Optional
    Optional<Member> findByUsername(String name);
}
```

## 단건

### 결과가 없을 때

- null 반환
- 단건을 조회하면 내부에서 JPQL의 Query.getSingleResult()를 호출한다.
    - 결과가 없으면 NoResultException를 던진다.
    - 스프링 데이터 JPA가 예외를 무시하고 null을 반환한다.
- 없을 수 있으면 Optional을 쓴다.

### 결과가 2건 이상일 때

- javax.persistence.NonUniqueResultException

## 컬렉션

### 결과가 없을 때

- 빈 컬렉션 반환

**Reference**

[쿼리 반환 타입](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#repository-query-return-types)