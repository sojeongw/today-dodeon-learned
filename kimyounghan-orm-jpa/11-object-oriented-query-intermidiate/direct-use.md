# Entity 직접 사용

![](../../.gitbook/assets/kimyounghan-orm-jpa/11/screenshot%202021-05-23%20오후%204.36.37.png)

JPQL에서 Entity를 직접 사용하면 SQL에서 해당 Entity의 기본키 값을 사용한다. 예제를 보면 둘 다 동일한 쿼리를 실행한다.

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

이렇게 Entity를 파라미터로 넘기는 방식과

```java
public class JpaMain {

  public static void main(String[] args) {
    String jpql = "select m from Member m where m.id = :memberId"; 
    
    List resultList = em.createQuery(jpql)
        .setParameter("memberId", memberId) .getResultList();
  }
}
```

식별자를 직접 전달하는 방식은

```sql
select m.* from Member m where m.id=?
```

같은 SQL을 실행한다.

## 외래키

```java
public class JpaMain {

  public static void main(String[] args) {
    Team team = em.find(Team.class, 1L);
    String qlString = "select m from Member m where m.team = :team"; 
    
    List resultList = em.createQuery(qlString)
        .setParameter("team", team) .getResultList();
  }
}
```

```java
public class JpaMain {

  public static void main(String[] args) {
    String qlString = "select m from Member m where m.team.id = :teamId"; 
    
    List resultList = em.createQuery(qlString)
        .setParameter("teamId", teamId) .getResultList();
  }
}
```

```sql
select m.* from Member m where m.team_id=?
```

외래키 역시 Entity와 키 값 모두 사용할 수 있다.