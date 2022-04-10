# 준영속 상태

- 영속
    - 영속성 컨텍스트에서 관리되는 상태
    - insert뿐만 아니라 조회할 때 1차 캐시에 없어서 DB에서 가져와 1차 캐시에 올리는 상태도 포함된다.
- 준영속
    - 영속 상태의 Entity가 영속성 컨텍스트에서 분리되는 상태
    - `detach()`를 실행하면 트랜잭션을 커밋해도 영향을 받지 않는다.
    - 영속성 컨텍스트가 더 이상 관리하지 않는 상태가 된다.
    - 즉, 영속성 컨텍스트가 제공하는 기능을 사용할 수 없다.

## 준영속 상태로 만드는 법

- em.detach(entity)
    - 특정 Entity만 준영속 상태로 전환한다.
- em.clear()
    - 영속성 컨텍스트를 통으로 지운다.
    - 즉, 완전히 초기화한다.
    - 1차 캐시 등이 다 사라졌기 때문에 같은 데이터를 조회해도 다시 DB에서 가져온다.
- em.close()
    - 영속성 컨텍스트를 종료한다.

```java
public class JpaMain {

    public static void main(String[] args) {
        
        ...

        Member member1 = entityManager.find(Member.class, 150L);
        member1.setName("AAAAA");

        entityManager.clear();

        // 영속성 컨텍스트를 지워서 1차 캐시에 없으므로 다시 올리기 위해 select 쿼리가 다시 실행된다.
        Member member2 = entityManager.find(Member.class, 150L);

        System.out.println("-----");

        tx.commit();
    }
}
```

```text
Hibernate: 
    select
        member0_.id as id1_0_0_,
        member0_.name as name2_0_0_ 
    from
        Member member0_ 
    where
        member0_.id=?
Hibernate: 
    select
        member0_.id as id1_0_0_,
        member0_.name as name2_0_0_ 
    from
        Member member0_ 
    where
        member0_.id=?
-----
```