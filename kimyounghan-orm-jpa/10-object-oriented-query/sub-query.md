# 서브 쿼리

- 쿼리 안에 서브로 넣는 쿼리

```sql
select m
from Member m
where m.age > (select avg(m2.age) from Member m2)
```

- 나이가 평균보다 많은 회원

```sql
select m
from Member m
where (select count(o) from Order o where m = o.member) > 0
```

- 한 건이라도 주문한 고객

## 서브 쿼리 지원 함수

### [NOT] EXIST + SUBQUERY

- { ALL | ANY | SOME } + SUBQUERY
- ALL
    - 모두 만족하면 참
- ANY, SOME
    - 조건을 하나라도 만족하면 참
    - 둘은 같은 의미다.

```sql
select m
from Member m
where exists(select t from m.team t where t.name = '팀A')
```

- 팀 A 소속인 회원

```sql
select o
from Order o
where o.orderAmount > ALL (select p.stockAmount from Product p)
```

- 전체 상품 각각의 재고보다 주문량이 많은 주문들

```sql
select m
from Member m
where m.team = ANY (select t from Team t)
```

- 어떤 팀이든 팀에 소속된 회원

### [NOT] IN + SUBQUERY

서브 쿼리의 결과 중 하나라도 같은 것이 있으면 참이다.

## 한계

- JPA는 where, having 절에서만 서브 쿼리를 사용할 수 있다.
- 구현체인 하이버네이트는 select 절에서도 사용할 수 있다.

```sql
select mm.age, mm.username
from (select m.age, m.username from Member m) as mm
```

- 위처럼 from 절의 서브 쿼리는 JPQL에서 불가능하다.
    - join으로 풀 수 있으면 join으로 해결하는 게 최선이다.
    - 꼭 from을 써야겠다면 네이티브 SQL을 써야 한다.
    - 하지만 애플리케이션 단에서 정리하거나, 쿼리를 각각 날려서 조합하는 식으로 해결하는 걸 추천한다.
    
