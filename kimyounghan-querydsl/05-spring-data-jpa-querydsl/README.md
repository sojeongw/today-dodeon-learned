# 스프링 데이터 JPA와 Querydsl

## 스프링 데이터 JPA 리포지토리로 변경

```java
public interface MemberRepository extends JpaRepository<Member, Long> {
    List<Member> findByUsername(String username);
}
```

- findByUsername()처럼 Spring Data JPA가 자동으로 제공하지 않는 기능은 직접 선언한다.

## 사용자 정의 리포지토리

![](../../.gitbook/assets/kimyounghan-querydsl/05/스크린샷%202022-07-23%20오후%207.58.28.png)

