# 객체 지향 쿼리 언어

JPA는 다양한 쿼리 방법을 지원한다.

- JPQL
- JPA Criteria
- QueryDSL
- 네이티브 SQL
- JDBC API
- MyBatis, SpringJdbcTemplate

그동안은 `EntityManager.find()`로 찾아서 `a.getB().getC()`로 객체 그래프를 탐색하는 가장 단순한 조회를 했다. 하지만 복잡한 조회 조건이 붙는다면
어떻게 해야할까?

JPA를 사용하면 Entity 객체를 중심으로 개발할 수 있다. 검색을 할 때도 테이블이 아니라 Entity 객체를 대상으로 실행된다. 하지만 모든 DB 데이터를 객체로 변환해서 검색하는
것은 불가능하다. 애플리케이션이 필요한 데이터만 DB에서 불러오려면 결국 검색 조건이 포함된 SQL이 필요하다.

그래서 JPA는 SQL을 추상화한 JPQL이라는 객체 지향 쿼리 언어를 제공한다. 추상화했기 때문에 특정 DB SQL에 의존하지 않는다. SQL 문법과 유사해서 select,
from, where, group by, having, join을 사용할 수 있다.

정리하자면 JPQL은 Entity 객체를 대상으로 쿼리하고, SQL은 DB 테이블을 대상으로 쿼리한다.

{% tabs %} {% tab title="JpaMain.java" %}

```java
public class JpaMain {

  public static void main(String[] args) {
    List<Member> resultList = em
        .createQuery("select m from Member m where m.username like '%kim%'", Member.class)
        .getResultList();

    tx.commit();
  }
}
```

{% endtab %} {% endtabs %}

위와 같이 Entity 대상으로 쿼리를 하면,

![](../../.gitbook/assets/kimyounghan-orm-jpa/10/screenshot%202021-04-03%20오후%204.21.46.png)

Entity 매핑 정보를 읽어서 SQL을 만들어 보낸다.

## Criteria

JPQL은 단순 String이기 때문에 동적 쿼리를 짜기가 힘들다.

- JPA 공식 기능
- 문자 대신 자바 코드로 JPQL을 작성할 수 있다.
- JPQL 빌더 역할을 한다.
- 너무 복잡하고 실용성이 없어 QueryDSL 사용을 권장한다.

{% tabs %} {% tab title="JpaMain.java" %}

```java
public class JpaMain {

  public static void main(String[] args) {
    // Criteria 사용 준비
    CriteriaBuilder cb = em.getCriteriaBuilder();
    CriteriaQuery<Member> query = cb.createQuery(Member.class);

    // 루트 클래스 (조회를 시작할 클래스)
    Root<Member> m = query.from(Member.class);

    //  쿼리 생성
    CriteriaQuery<Member> cq = query.select(m).where(cb.equal(m.get("username"), "kim")); 
    List<Member> resultList = em.createQuery(cq).getResultList();
  }
}
```

{% endtab %} {% endtabs %}

## QueryDSL

- 문자 대신 자바 코드로 JPQL을 작성할 수 있다.
- JPQL 빌더 역할을 한다.
- 컴파일 시점에 문법 오류를 찾을 수 있다.
- 동적 쿼리 작성이 편리하다.
- 단순하고 쉬워 실무 사용을 권장한다.

{% tabs %} {% tab title="JpaMain.java" %}

```java
public class JpaMain {

  public static void main(String[] args) {
    JPAFactoryQuery query = new JPAQueryFactory(em);
    QMember m = QMember.member;
    
    List<Member> list = query.selectFrom(m)
        .where(m.age.gt(18))
        .orderBy(m.name.desc())
        .fetch();
  }
}
```

{% endtab %} {% endtabs %}

JPQL만 잘하면 QueryDSL 도큐먼트만 보고 쉽게 적용할 수 있다.

## 네이티브 SQL

- JPQ가 제공하는 SQL을 직접 사용하는 기능
- JPQL로 해결할 수 없는 특정 DB에 의존적인 기능에 사용한다.
    - ex. 오라클 `connect by`
    
```java
public class JpaMain {

  public static void main(String[] args) {
    String sql = "SELECT ID, AGE, TEAM_ID, NAME FROM MEMBER WHERE NAME = ‘kim’";
    
    List<Member> resultList =
        em.createNativeQuery(sql, Member.class).getResultList();
  }
}
```

## JDBC, SpringJdbcTemplate

- JPA를 사용하면서 JDBC 커넥션을 직접 사용하는 방법이다.
- 스프링 JdbcTemplate이나 Mybatis 등을 함께 사용할 수도 있다.
- 영속성 컨텍스트를 적절한 시점에 강제로 `flush()` 해야 한다.
    - ex. JPA를 우회해서 SQL 실행 직전에 영속성 컨텍스트를 수동 `flush()`한다.
