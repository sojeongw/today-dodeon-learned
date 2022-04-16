# Named 쿼리

- 미리 정의해서 이름을 부여해두고 사용하는 쿼리
- 정적 쿼리
- 애너테이션, XML에 정의한다.

## 특징

- 애플리케이션 로딩 시점에 초기화 후 재사용한다.
    - 정적 쿼리라 변하지 않으니 JPA나 하이버네이트가 파싱할 때 캐싱해서 사용한다.
    - 즉, 로딩 시점에 한 번 캐싱을 해두므로 코스트가 줄어든다.
- 애플리케이션 로딩 시점에 쿼리를 검증한다.
    - 실행 시점에 문법이 안 맞으면 예외를 던진다.

## 애너테이션에 정의하는 법

```java

@Entity
// 쿼리에 미리 이름을 선언해 놓는다.
@NamedQuery(
        name = "Member.findByUsername",
        query = "select m from Member m where m.username = :username")
public class Member {
  ...
}

public class JpaMain {

    public static void main(String[] args) {
        List<Member> resultList =
                // 정의해둔 이름으로 쿼리를 사용한다.
                em.createNamedQuery("Member.findByUsername", Member.class)
                        .setParameter("username", "회원1")
                        .getResultList();
    }
}
```

## XML에 정의하는 법

{% tabs %} {% tab title="persistence.xml" %}

```xml

<persistence-unit name="jpabook">
    <mapping-file>META-INF/ormMember.xml</mapping-file>
```

{% endtab %} {% tab title="ormMember.xml" %}

```xml
<?xml version="1.0" encoding="UTF-8"?>
<entity-mappings xmlns="http://xmlns.jcp.org/xml/ns/persistence/orm" version="2.1">
    <named-query name="Member.findByUsername">
        <query><![CDATA[
select m
from Member m
where m.username = :username
]]></query>
    </named-query>
    <named-query name="Member.count">
        <query>select count(m) from Member m</query>
    </named-query>
</entity-mappings>
```

{% endtab %} {% endtabs %}

- XML 설정이 항상 우선권을 가진다.
- 애플리케이션 운영 환경에 따라 다른 XML을 배포할 수 있다.

## @Query

![](../../.gitbook/assets/kimyounghan-orm-jpa/11/screenshot%202021-05-23%20오후%204.52.58.png)

- Spring Data JPA의 `@Query` 기능이 바로 Named Query다. 
- `@NamedQuery`로 Entity에 직접 쓰면 지저분하니까 Spring Data JPA를 쓰는 게 좋다.