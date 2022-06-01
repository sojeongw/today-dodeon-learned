# 상수, 문자 더하기

## 상수

```java

@SpringBootTest
@Transactional
public class QuerydslBasicTest {
    
    @Test
    public void constant() {
        Tuple result = queryFactory
                .select(member.username, Expressions.constant("A"))
                .from(member)
                .fetchFirst();
    }
}
```

```text
result = [member1, A]
```

- Expressions.constant()를 사용한다.

## 문자 더하기

```java

@SpringBootTest
@Transactional
public class QuerydslBasicTest {
    
    @Test
    public void concat() {
        String result = queryFactory
                .select(member.username.concat("_").concat(member.age.stringValue()))
                .from(member)
                .where(member.username.eq("member1"))
                .fetchOne();
    }
}
```

```text
member1_10
```

- ENUM과 문자가 아닌 타입은 stringValue()로 변환한다.