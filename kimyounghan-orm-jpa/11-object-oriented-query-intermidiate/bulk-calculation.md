# 벌크 연산

- PK를 지정하지 않고 실행하는 update, delete 문

재고가 10개 미만인 모든 상품의 가격을 10% 상승하려면 어떻게 해야할까?

1. 재고가 10개 미만인 상품을 리스트로 조회한다.
2. 상품 Entity의 가격을 10% 증가시킨다.
3. 트랜잭션 커밋 시점에 변경 감지가 동작한다.

JPA 변경 감지 기능으로 이 모든 과정을 실행하려면 너무 많은 SQL이 나간다. 변경할 데이터가 100건이라면 100번의 update sql을 실행해야 한다.

```java
public class JpaMain {

    public static void main(String[] args) {
        String qlString = "update Product p "
                + "set p.price = p.price * 1.1 "
                + "where p.stockAmount < :stockAmount";

        int resultCount = em.createQuery(qlString)
                .setParameter("stockAmount", 10)
                // 영향받은 Entity 수를 반환한다.
                .executeUpdate();
    }
}
```

- 벌크 연산을 사용하면 쿼리 한 번으로 여러 row를 변경할 수 있다.
- 즉, 엔티티 여러 개를 수정할 수 있다.

## 지원 쿼리

- update
- delete
- 하이버네이트는 insert를 지원한다.

## 주의사항

- 벌크 연산은 영속성 컨텍스트를 무시하고 DB에 직접 쿼리한다.

## 해결 방안

- 영속성 컨텍스트 작업을 하기 전에 벌크 연산을 먼저 실행한다.
- 혹은 벌크 연산 수행 후에 영속성 컨텍스트를 초기화한다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/11/screenshot%202021-05-23%20오후%205.01.52.png)

- member 객체 생성 후 직접 flush()를 하지 않아도 벌크 연산 전에 insert 문이 나간다.
    - JPA 기본 flush 모드인 auto는 commit이나 query가 나가면 flush()를 실행하기 때문이다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/11/screenshot%202021-05-23%20오후%205.06.22.png)

- 문제는 벌크 연산으로 강제 업데이트 한 후, 영속성 컨텍스트에 남아있는 기존 데이터엔 반영이 안된다는 것이다.
    - member.getAge()를 하면 반영이 되지 않은 상태다.
- flush()는 단지 DB에 반영하는 것이지 영속성 컨텍스트에서 지워지는 건 아니다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/11/screenshot%202021-05-23%20오후%205.08.55.png)

- 따라서 **영속성 컨텍스트 초기화를 한 뒤** 새롭게 객체를 가져와야 한다.
- clear() 한 뒤 기존 데이터 대신 다시 member를 가져오면 제대로 보인다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/11/screenshot%202021-05-23%20오후%205.09.30.png)

- 초기화를 안 하고 가져오면 여전히 이전 데이터를 가져온다.

## @Modifying

![](../../.gitbook/assets/kimyounghan-orm-jpa/11/screenshot%202021-05-23%20오후%205.11.30.png)

- Spring Data JPA에서도 비슷한 기능을 다양한 옵션과 함께 지원한다.