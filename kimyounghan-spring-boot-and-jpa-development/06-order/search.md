# 주문 검색 기능 개발

- JPA에서 동적 쿼리를 어떻게 해결할지 알아본다.

```java

@Getter
@Setter
public class OrderSearch {

    private String memberName;
    private OrderStatus orderStatus;

}
```

- memberName, orderStatus 중 검색 조건에 해당하지 않는 항목은 쿼리에 포함하지 않게 만든다.

## JPQL로 처리

```java

@Repository
@RequiredArgsConstructor
public class OrderRepository {

    public List<Order> findAllByString(OrderSearch orderSearch) {
        String jpql = "select o From Order o join o.member m";
        boolean isFirstCondition = true;

        // 주문 상태 검색
        if (orderSearch.getOrderStatus() != null) {
            if (isFirstCondition) {
                jpql += " where";
                isFirstCondition = false;
            } else {
                jpql += " and";
            }
            jpql += " o.status = :status";
        }

        // 회원 이름 검색
        if (StringUtils.hasText(orderSearch.getMemberName())) {
            if (isFirstCondition) {
                jpql += " where";
                isFirstCondition = false;
            } else {
                jpql += " and";
            }
            jpql += " m.name like :name";
        }

        TypedQuery<Order> query = em.createQuery(jpql, Order.class).setMaxResults(1000); // 최대 1000건

        if (orderSearch.getOrderStatus() != null) {
            query = query.setParameter("status", orderSearch.getOrderStatus());
        }

        if (StringUtils.hasText(orderSearch.getMemberName())) {
            query = query.setParameter("name", orderSearch.getMemberName());
        }

        return query.getResultList();
    }
}
```

굉장히 번거로운 작업이 된다.

## JPA Criteria로 처리

```java

@Repository
@RequiredArgsConstructor
public class OrderRepository {

    public List<Order> findAllByCriteria(OrderSearch orderSearch) {
        CriteriaBuilder cb = em.getCriteriaBuilder();
        CriteriaQuery<Order> cq = cb.createQuery(Order.class);
        Root<Order> o = cq.from(Order.class);
        Join<Order, Member> m = o.join("member", JoinType.INNER); // 회원과 조인
        List<Predicate> criteria = new ArrayList<>();

        // 주문 상태 검색
        if (orderSearch.getOrderStatus() != null) {
            Predicate status = cb.equal(o.get("status"),
                    orderSearch.getOrderStatus());
            criteria.add(status);
        }

        // 회원 이름 검색
        if (StringUtils.hasText(orderSearch.getMemberName())) {
            Predicate name =
                    cb.like(m.<String>get("name"), "%" +
                            orderSearch.getMemberName() + "%");
            criteria.add(name);

        }

        cq.where(cb.and(criteria.toArray(new Predicate[criteria.size()])));
        TypedQuery<Order> query = em.createQuery(cq).setMaxResults(1000); // 최대 1000건

        return query.getResultList();
    }
}
```

JPA Criteria는 JPA 표준 스펙이지만 너무 복잡하다. 대안은 QueryDSL이다. 지금은 일단 이대로 진행한다.