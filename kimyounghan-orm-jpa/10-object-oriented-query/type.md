# JPQL 타입 표현과 기타 식

## 타입

### 문자

- 'hello'
- 'she"s'
    - single quotation은 두 개를 넣어주면 된다.

### 숫자

- 10L
- 10D
- 10F

### Boolean

- true
- false

### ENUM

```sql
select m.username, 'hello', true
from Member m
where m.type = jpql.MemberType.ADMIN
```

```java
class JpaMain {
    public static void main(String[] args) {
        String query = "select m.username, 'hello', true from Member m " +
                // 특정 enum 타입 조회
                "where m.type = jpql.MemberType.USER";

        List<Object[]> result = em.createQuery(query)
                .getResulstList();

        for (Object[] objects : result) {
            System.out.println("objects " + objects[0]);
            System.out.println("objects " + objects[1]);
            System.out.println("objects " + objects[2]);
        }
    }
}
```

```text
object = teamA
// 그냥 이렇게 입력한 문자를 그대로 뽑을 수도 있다.
object = hello
object = true
```

- jpabook.MemberType.Admin
    - 패키지명을 포함해서 넣어야 한다.

```java
class JpaMain {
    public static void main(String[] args) {
        String query = "select m.username, 'hello', true from Member m " +
                // 패키지 이름이 너무 길면 이렇게 표현할 수 있다.
                "where m.type = :userType";


        List<Object[]> result = em.createQuery(query)
                // userType에 원하는 enum을 세팅한다.
                .setParameter("userType", MemberType.ADMIN)
                .getResulstList();

        for (Object[] objects : result) {
            System.out.println("objects " + objects[0]);
            System.out.println("objects " + objects[1]);
            System.out.println("objects " + objects[2]);
        }
    }
}
```

- 패키지 이름이 길어지면 setParameter()에 설정해준다.

### Entity

- TYPE(m) = Member
    - 타입을 확인할 때 사용한다.
    - ex. 상속 관계

```java
TypedQuery<Item> query=em.createQuery("select i from Item i where type(i) = 'Book'",Item.class);
```

- `Book`이 `Item`을 상속할 때 Entity 타입이 `Book`인 경우를 조회한다.

## 기타 식

- SQL과 문법이 같은 식
    - EXIST, IN
    - AND, OR, NOT
    - =, >, >=, <, <=, <>
    - BETWEEN, LIKE, IS NULL