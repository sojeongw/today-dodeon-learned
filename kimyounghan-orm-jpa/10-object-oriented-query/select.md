# 프로젝션

- select 절에 조회할 대상을 지정하는 것
- Entity, 임베디드 타입, 스칼라 타입(숫자, 문자 등 기본 데이터 타입)이 대상이다.
- `distinct`를 붙여서 중복 제거를 할 수 있다.

### Entity 프로젝션

```jpaql
select m from Member m
select m.team from Member m
```

- `m`이 Member Entity의 alias이므로 Entity 프로젝션이다.
- `m.team`도 결과가 `team` Entity이므로 Entity 프로젝션에 해당한다.

```java
public class JpaMain {

    public static void main(String[] args) {
        Member member = new Member();
        member.setUsername("member");
        em.persist(member);

        em.flush();
        em.clear();

        // Entity 프로젝션으로 조회한 데이터는 영속성 컨텍스트에서 관리된다.
        List<Member> result = em.createQuery("select m from Member m", Member.class).getResultList();

        Member findMember = result.get(0);
        // 영속성 컨텍스트에서 관리되기 때문에 update 쿼리가 나간다.
        findMember.setAge(30);

        tx.commit();
    }
}
```

- Entity 프로젝션을 하면 select에 들어가는 모든 데이터가 영속성 컨텍스트에서 관리된다.

```java
public class JpaMain {

    public static void main(String[] args) {
        Member member = new Member();
        member.setUsername("member");
        em.persist(member);

        em.flush();
        em.clear();

        List<Team> result = em.createQuery("select m.team from Member m", Team.class).getResultList();

        tx.commit();
    }
}
```

![](../../.gitbook/assets/kimyounghan-orm-jpa/10/screenshot%202021-04-03%20오후%207.05.55.png)

- 코드는 단순 select 문이지만 `m.team`이 다른 테이블이므로 실제로는 join으로 나간다.
    - 이런 경우에는 그냥 join으로 명확히 써주는 것이 좋다.
        - `select t from Member m join m.team t`
    - 안 그러면 join이 발생하는지 예측이 안된다.

### 임베디드 타입 프로젝션

```jpaql
select m.address from Member m
```

- `address` 같은 임베디드 타입도 가능하다.

```java
public class JpaMain {

    public static void main(String[] args) {
        Member member = new Member();
        member.setUsername("member");
        em.persist(member);

        em.flush();
        em.clear();

        // 임베디드 타입 조회
        em.createQuery("select o.address from Order o", Address.class).getResultList();

        tx.commit();
    }
}
```

![](../../.gitbook/assets/kimyounghan-orm-jpa/10/screenshot%202021-04-03%20오후%207.11.02.png)

- 임베디드 타입은 그 Entity에 속해있으므로 join 없이 조회한다.
- 임베디드 타입은 소속된 타입이므로 Entity 조회하듯 할 수 없다.
    - ex. select address from Address (X)

### 스칼라 타입 프로젝션

```jpaql
select m.username, m.age from Member m
```

- String, int 등의 기본 데이터 타입이 해당한다.

## 여러 값 조회

```sql
select m.username, m.age
from Member m
```

- 조회하는 데이터들의 타입이 다르거나 여러 개를 가져와야 할 때는 3가지 방법을 사용할 수 있다.

### Query 타입으로 조회

```java
public class JpaMain {

    public static void main(String[] args) {
        // String과 int 값을 반환한다.
        Query query = em.createQuery("select m.username, m.age from Member m");
    }
}
```

### Object[] 타입으로 조회

```java
public class JpaMain {

    public static void main(String[] args) {
        Member member = new Member();
        member.setUsername("member");
        member.setAge(30);
        em.persist(member);

        List resultList = em.createQuery("select m.username, m.age from Member m")
                .getResultList();

        Object o = resultList.get(0);
        Object[] result = (Object[]) o;

        System.out.println("result = " + result[0]);
        System.out.println("result = " + result[1]);
    }
}
```

![](../../.gitbook/assets/kimyounghan-orm-jpa/10/screenshot%202021-04-03%20오후%207.18.10.png)

`Object`타입으로 바꿔서 조회한다.

```java
public class JpaMain {

    public static void main(String[] args) {
        Member member = new Member();
        member.setUsername("member");
        member.setAge(30);
        em.persist(member);

        List<Object[]> resultList = em.createQuery("select m.username, m.age from Member m")
                .getResultList();
        Object[] result = resultList.get(0);

        System.out.println("result = " + result[0]);
        System.out.println("result = " + result[1]);
    }
}
```

이렇게 제네릭을 이용해 바로 받아올 수도 있다.

### new 명령어로 조회

{% tabs %} {% tab title="JpaMain.java" %}

```java
public class JpaMain {

    public static void main(String[] args) {
        Member member = new Member();
        member.setUsername("member");
        member.setAge(30);
        em.persist(member);

        em.flush();
        em.clear();

        // 단순 값을 DTO로 바로 가져올 수도 있다.
        List<MemberDto> resultList = em
                // 패키지명.생성자로 불러온다.
                .createQuery("select new jpql.MemberDto(m.username, m.age) from Member m",
                        MemberDto.class).getResultList();

        MemberDto memberDto = resultList.get(0);

        System.out.println("result = " + memberDto.getUsername());
        System.out.println("result = " + memberDto.getAge());

        tx.commit();
    }
}
```

{% endtab %}{% tab title="MemberDto.java" %}

```java
public class MemberDto {
    private String username;
    private int age;

    // 생성자의 순서와 타입이 일치해야 가져올 수 있다.
    public MemberDto(String username, int age) {
        this.username = username;
        this.age = age;
    }
}

```

{% endtab %} {% endtabs %}

- Entity가 아닌 타입에 생성자로 매핑해서 가져오는 방법
- 순서와 타입이 일치하는 생성자가 필요하다.
- `패키지명.클래스명`으로 가져오기 때문에 패키지명이 길어질수록 사용하기 힘든 단점이 있다.