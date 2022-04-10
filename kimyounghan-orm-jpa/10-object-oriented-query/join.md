# 조인

## 내부 조인

```sql
SELECT m
FROM Member m [INNER] JOIN m.team t
```

- team 데이터가 없으면 그 줄은 출력되지 않는다.
- 괄호 안의 키워드는 생략 가능하다.

## 외부 조인

```sql
SELECT m
FROM Member m LEFT [OUTER] JOIN m.team t
```

- 데이터가 없어도 null로 다 출력된다.

## 세타 조인

```sql
select count(m)
from Member m,
     Team t
where m.username = t.name
```

- 연관 관계가 전혀 없는 데이터를 카티시안 곱으로 모든 짝을 불러온 뒤, 그 중 조건에 맞는 걸 찾는다.

## ON 절

- JPA 2.1부터 지원하는 기능
- 조인할 대상을 미리 필터링 할 수 있다.
- 연관 관계 없는 Entity도 외부 조인할 수 있다.
    - 과거에는 내부 조인만 가능했다.
    - 하이버네이트 5.1부터 지원한다.

### 조인 대상 필터링

```jpaql
SELECT m, t FROM Member m LEFT JOIN m.team t on t.name = 'A'
```

- 회원과 팀을 대상으로 팀 이름이 A인 팀만 조인한다.

```sql
SELECT m.*, t.*
FROM Member m
         LEFT JOIN Team t ON m.TEAM_ID = t.id and t.name = 'A'
```

- 실제 나가는 SQL

### 연관 관계 없는 Entity 외부 조인

```jpaql
SELECT m, t FROM Member m LEFT JOIN Team t on m.username = t.name
```

- 회원의 이름과 팀의 이름이 같은 데이터를 대상으로 외부 조인한다.

```sql
SELECT m.*, t.*
FROM Member m
         LEFT JOIN Team t ON m.username = t.name
```

- 실제 나가는 SQL