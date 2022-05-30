# 기본 Q-Type 활용

## 별칭 직접 지정

## 기본 인스턴스 활용

{% tabs %} {% tab title="QMember.java" %}

```java

@Generated("com.querydsl.codegen.DefaultEntitySerializer")
public class QMember extends EntityPathBase<Member> {

    public static final QMember member = new QMember("member1");

    ...

}
```

{% endtab %} {% tab title=".java" %}

```java
import static study.querydsl.entity.QMember.*;

@SpringBootTest
@Transactional
public class QuerydslBasicTest {
    
    ...

    @Test
    public void startQuerydsl3() {
        Member findMember = queryFactory
                .select(member)
                .from(member)
                .where(member.username.eq("member1"))
                .fetchOne();

        assertThat(findMember.getUsername()).isEqualTo("member1");
    }

}
```

{% endtab %} {% endtabs %}

- Q 클래스에서 만들어놓은 QMember를 static import해서 사용할 수 있다.

### JPQL 로그 확인

```yaml
spring.jpa.properties.hibernate.use_sql_comments: true
```

- Querydsl은 결국 JPQL을 만드는 빌더 역할을 한다.
    - 따라서 JPQL 로그를 보려면 위와 같이 추가한다.

```sql
select member1
from Member member1
where member1.username = ?1 
```

- QMember에서 variable 값(JPQL의 alias)을 member1으로 자동 생성했기 때문에 쿼리에도 member1로 나간다.

```java
import static study.querydsl.entity.QMember.*;

@SpringBootTest
@Transactional
public class QuerydslBasicTest {
    
    ...

    @Test
    public void startQuerydsl3() {
        QMember m1 = new QMember("m1");

        Member findMember = queryFactory
                .select(m1)
                .from(m1)
                .where(m1.username.eq("member1"))
                .fetchOne();

        assertThat(findMember.getUsername()).isEqualTo("member1");
    }

}
```

- 직접 지정하면 m1이란 이름으로 jpql이 나간다.
- 같은 테이블을 조인해야 하는 경우는 이름이 같으면 안되니까 이렇게 따로 선언해 사용한다.
    - 이런 경우가 아니면 기본 인스턴스를 사용한다.