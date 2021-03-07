# 페이징

JPA는 복잡한 페이징을 두 가지 API로 추상화했다.

## setFirstResult()

- setFirstResult(int startPosition)
- 조회 시작 위치(0부터 시작)를 입력한다.

## setMaxResults()

- setMaxResults(int maxResult)
- 조회할 데이터 수를 입력한다.

```java

public class JpaMain {

  public static void main(String[] args) {
    List<Member> resultList = em
        .createQuery("select m from Member m order by m.age desc", Member.class)
        .setFirstResult(1).setMaxResults(10).getResultList();
  }
}
```

![](../../.gitbook/assets/kimyounghan-orm-jpa/10/screenshot%202021-04-03%20오후%207.41.51.png)

만약 오라클이었다면 위처럼 복잡한 쿼리를 짜야하는데 API로 쉽게 사용할 수 있다.

