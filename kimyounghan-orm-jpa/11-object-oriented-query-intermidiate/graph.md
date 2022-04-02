# 경로 표현식

![](../../.gitbook/assets/kimyounghan-orm-jpa/11/screenshot%202021-05-23%20오후%201.29.36.png)

점을 찍어 객체 그래프를 탐색하는 식이다.

- 상태 필드
    - 객체가 가지고 있는 필드
    - m.username
- 단일값 연관 필드
    - 연관 관계를 맺은 것 중 단일 값인 필드
    - m.team
- 컬렉션값 연관 필드
    - 연관 관계를 맺은 것 중 컬렉션으로 된 필드
    - m.orders

어떤 필드냐에 따라 내부적으로 동작하는 방식과 결과가 달라지므로 꼭 구분해야 한다.

## 상태 필드

- 단순히 값을 저장하기 위한 필드
- 경로 탐색의 끝이라서 더 이상 탐색이 되지 않는다.
    - `m.name`에서 또 점을 찍어 다른 객체를 탐색할 수 없다.

## 연관 필드

- 연관 관계를 위한 필드

### 단일값 연관 필드

- 대상이 Entity다.
    - ManyToOne
    - OneToOne
- 탐색을 더 할 수 있다.
    - `m.team.name`처럼 team에서 또 탐색을 할 수 있다.
- 묵시적 내부 조인(inner join)이 발생한다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/11/screenshot%202021-05-23%20오후%201.58.53.png)

![](../../.gitbook/assets/kimyounghan-orm-jpa/11/screenshot%202021-05-23%20오후%201.49.15.png)

java 코드에서는 점을 찍어서 접근하면 되지만, 쿼리에서는 inner join을 통해 team의 name을 가져온다. 이것을 묵시적 내부 조인이라고 한다.

편해보이지만 실무 때는 쿼리 튜닝이 어렵다. 실무에서는 묵시적 내부 조인이 일어나지 않도록 한다. 안 그러면 성능에 지대한 영향을 주게 된다. JPQL과 쿼리를 맞춰서 명시적으로
표현하면, 직관적으로 알 수 있다.

### 컬렉션 값 연관 필드

- 대상이 컬렉션이다.
    - OneToMany
    - ManyToMany
- 묵시적 내부 조인이 발생한다.
- 탐색을 할 수 없다.
    - t.members.name으로 name을 접근할 수 없다.

```sql
select m.username
from Team t
         join t.members m
```

from 절에서 명시적 조인을 통해 별칭을 얻으면 별칭으로 탐색할 수 있다.

## 명시적 조인

```sql
select m
from Member m
         join m.team t
```

- join 키워드를 직접 사용한다.

## 묵시적 조인

```sql
select m.team from Member m
```

- 경로 표현식에 의해 묵시적으로 SQL 조인이 발생한다.
    - 경로 탐색을 사용한 묵시적 조인은 항상 내부 조인이다.
    - 외부 조인을 하고 싶다면 명시적 조인을 하면 된다.
- 컬렉션은 경로 탐색의 끝이므로 명시적 조인을 통한 별칭으로 얻어야 한다.
- 경로 탐색은 주로 select, where 절에서 사용하지만, 묵시적 조인으로 인해 SQL의 from, join 절에 영향을 준다.

## 싦무 조언

- 가급적 묵시적 조인 대신 명시적 조인을 사용한다.
- 조인은 SQL 튜닝에 있어 중요 포인트이기 때문이다.
- 묵시적 조인은 조인이 일어나는 상황을 한 눈에 파악하기 어렵다.