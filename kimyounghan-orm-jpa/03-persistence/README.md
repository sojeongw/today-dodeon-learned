# 영속성 관리

![](../../.gitbook/assets/kimyounghan-orm-jpa/03/스크린샷%202021-03-13%20오후%205.12.20.png)

- 요청이 올 때마다 EntityManagerFactory를 통해 EntityManager를 생성한다.
- EntityManager는 내부적으로 DB 커넥션을 통해 DB를 사용한다.