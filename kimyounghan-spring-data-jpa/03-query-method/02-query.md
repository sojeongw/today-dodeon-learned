# @Query

## 엔티티 조회

{% tabs %} {% tab title="MemberRepository.java" %}

```java
public interface MemberRepository extends JpaRepository<Member, Long> {

    @Query("select m from Member m where m.username= :username and m.age = :age")
    List<Member> findUser(@Param("username") String username, @Param("age") int age);
}
```

{% endtab %} {% endtabs %}

- 복잡한 쿼리나 JPQL을 @Query에 넣어서 해결할 수 있다.
    - 메서드 이름에 따라 쿼리를 자동 생성하면 이름이 너무 길어진다.
- 실행할 메서드의 정적 쿼리를 직접 작성하므로 이름 없는 Named Query라고 할 수 있다.
- JPA Named Query처럼 애플리케이션 실행 시점에 문법 오류를 발견할 수 있다.

## 값 조회

{% tabs %} {% tab title="MemberRepository.java" %}

```java
public interface MemberRepository extends JpaRepository<Member, Long> {

    @Query("select m.username from Member m")
    List<String> findUsernameList();
}
```

{% endtab %} {% endtabs %}

## DTO 조회

{% tabs %} {% tab title="MemberRepository.java" %}

```java
public interface MemberRepository extends JpaRepository<Member, Long> {

    // new 명령어 필요
    @Query("select new study.datajpa.dto.MemberDto(m.id, m.username, t.name) " +
            "from Member m join m.team t")
    List<MemberDto> findMemberDto();
}

@Data
public class MemberDto {
    private Long id;
    private String username;
    private String teamName;

    public MemberDto(Long id, String username, String teamName) {
        this.id = id;
        this.username = username;
        this.teamName = teamName;
    }
}
```

{% endtab %} {% endtabs %}

- JPA의 new 명령어를 사용한다.
- 생성자가 일치하는 DTO가 필요하다.