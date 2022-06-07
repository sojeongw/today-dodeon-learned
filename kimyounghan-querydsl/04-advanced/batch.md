# 수정, 삭제 벌크 연산

## 대량 데이터 수정

- 개별로 보내는 것보다 한번에 보내는게 성능상 좋다.

```java

@SpringBootTest
@Transactional
public class QuerydslBasicTest {

    @Test
    void bulkUpdate() {
        long count = queryFactory
                .update(member)
                .set(member.username, "비회원")
                .where(member.age.lt(28))
                .execute();
    }
}
```

```sql
update member
set username=?
where age < ?

update member
set username=NULL
where age < ?
```

```text
m = Member(id=3, username=비회원, age=10)
m = Member(id=4, username=비회원, age=20)
m = Member(id=5, username=member3, age=30)
m = Member(id=6, username=member4, age=40)
```

- JPA는 영속성 컨텍스트에 엔티티를 올려놓는다.
- 벌크 연산을 하게 되면 모든 데이터가 영속성 컨텍스트에 올라가게 된다.
- 그런데 벌크 연산은 영속성 컨텍스트를 무시하고 바로 쿼리를 보낸다.
- 따라서 DB 상태와 영속성 컨텍스트의 상태가 달라진다.

```java

@SpringBootTest
@Transactional
public class QuerydslBasicTest {

    @Test
    void bulkUpdate() {
        long count = queryFactory
                .update(member)
                .set(member.username, "비회원")
                .where(member.age.lt(28))
                .execute();

        List<Member> result = queryFactory
                .selectFrom(member)
                .fetch();

        for (Member m : result) {
            System.out.println("m = " + m);
        }
    }
}
```

```text
m = Member(id=3, username=member1, age=10)
m = Member(id=4, username=member2, age=20)
m = Member(id=5, username=member3, age=30)
m = Member(id=6, username=member4, age=40)
```

- 만약 벌크 연산 후 조회 로직을 넣는다면?
- 이미 영속성 컨텍스트에 해당 id로 값이 있기 때문에 영속성 컨텍스트의 데이터가 우선적으로 보여진다.
- 즉, DB에 비회원이라고 업데이트된 것과는 다른 값이 출력된다.

```java

@SpringBootTest
@Transactional
public class QuerydslBasicTest {

    @Test
    void bulkUpdate() {
        long count = queryFactory
                .update(member)
                .set(member.username, "비회원")
                .where(member.age.lt(28))
                .execute();

        em.flush();
        em.clear();

        List<Member> result = queryFactory
                .selectFrom(member)
                .fetch();

        for (Member m : result) {
            System.out.println("m = " + member1);
        }
    }
}
```

```text
m = Member(id=3, username=비회원, age=10)
m = Member(id=4, username=비회원, age=20)
m = Member(id=5, username=member3, age=30)
m = Member(id=6, username=member4, age=40)
```

- flush를 해줘서 영속성 컨텍스트와 DB를 맞춘 뒤, clear로 초기화 한다.

## 대량 데이터 1씩 더하기

```java

@SpringBootTest
@Transactional
public class QuerydslBasicTest {

    @Test
    void bulkUpdate() {
        long count = queryFactory
                .update(member)
                .set(member.age, member.age.add(1))
                .execute();
    }
}
```

```text
update member set age=age+?
update member set age=age+NULL
```

- 곱하기는 multiply()를 사용할 수 있다.

## 대량 데이터 삭제

```java

@SpringBootTest
@Transactional
public class QuerydslBasicTest {

    @Test
    void bulkUpdate() {
        long count = queryFactory
                .delete(member)
                .where(member.age.gt(18))
                .execute();
    }
}
```

```text
delete from member where age>?
delete from member where age>NULL
```