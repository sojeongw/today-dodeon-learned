# Entity 직접 사용

![](../../.gitbook/assets/kimyounghan-orm-jpa/11/screenshot%202021-05-23%20오후%204.36.37.png)

- JPQL에서 id 대신 Entity를 넣으면 SQL에서 자동으로 해당 Entity의 기본키를 사용한다.

## 기본키

```java
public class JpaMain {

    public static void main(String[] args) {
        String jpql = "select m from Member m where m = :member";

        List resultList = em.createQuery(jpql)
                .setParameter("member", member).getResultList();
    }
}
```

- Entity를 파라미터로 넘기는 방식

```java
public class JpaMain {

    public static void main(String[] args) {
        String jpql = "select m from Member m where m.id = :memberId";

        List resultList = em.createQuery(jpql)
                .setParameter("memberId", memberId).getResultList();
    }
}
```

- 식별자를 직접 전달하는 방식

```sql
select m.*
from Member m
where m.id = ?
```

- 둘 다 같은 SQL을 실행한다.

## 외래키

```java
public class JpaMain {

    public static void main(String[] args) {
        Team team = em.find(Team.class, 1L);
        // 엔티티를 파라미터로 바로 전달
        // 연관 관계 매핑 설정에서 적어둔 FK를 사용한다.
        String qlString = "select m from Member m where m.team = :team";

        List resultList = em.createQuery(qlString)
                .setParameter("team", team).getResultList();
    }
}
```

```java
public class JpaMain {

    public static void main(String[] args) {
        // 키를 파라미터로 전달
        String qlString = "select m from Member m where m.team.id = :teamId";

        List resultList = em.createQuery(qlString)
                .setParameter("teamId", teamId).getResultList();
    }
}
```

```sql
select m.*
from Member m
where m.team_id = ?
```

- 외래 키 역시 Entity를 사용할 수 있다.