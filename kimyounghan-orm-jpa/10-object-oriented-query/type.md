# JPQL 타입 표현과 기타 식
## 타입
### 문자

- 'hello'
- 'she"s'

### 숫자

- 10L
- 10D
- 10F

### Boolean

- true
- false

### ENUM

- jpabook.MemberType.Admin

패키지명을 포함해서 넣어야 한다.

```sql
select m.username, 'hello', true from Member m where m.type = jpql.MemberType.ADMIN
```

만약 패키지 이름이 길어진다면

```java
String query = "select m.username, 'hello', true from Member m where m.type = :userType";

List<Object[]> result = em.createQuery(query).setParameter("userType", MemberType.ADMIN).getResulstList();
```

이렇게 표현할 수 있다.

### Entity

- TYPE(m) = Member

상속 관계에서 사용한다.

```java
TypedQuery<Item> query = em.createQuery("select i from Item i where type(i) = 'Book'", Item.class);
```

`Book`이 `Item`을 상속할 때 엔티티 타입이 `Book`인 경우를 조회

## 기타 식

SQL과 문법이 같은 식

- EXIST, IN
- AND, OR, NOT
- =, >, >=, <, <=, <>
- BETWEEN, LIKE, IS NULL