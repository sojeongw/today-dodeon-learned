# 페이징

## setFirstResult()

- 조회 시작 위치
    - 0부터 시작한다.
- setFirstResult(int startPosition)

## setMaxResults()

- 조회할 데이터 수
- setMaxResults(int maxResult)

```java

public class JpaMain {

    public static void main(String[] args) {
        List<Member> resultList = em
                // order by를 해봐야 페이징이 잘 되는지 확인할 수 있다.
                .createQuery("select m from Member m order by m.age desc", Member.class)
                .setFirstResult(1).setMaxResults(10).getResultList();
    }
}
```

![](../../.gitbook/assets/kimyounghan-orm-jpa/10/screenshot%202021-04-03%20오후%207.41.51.png)

- 위의 로그처럼 오라클이었다면 복잡했을 쿼리를 API를 통해 쉽게 적용할 수 있다.

