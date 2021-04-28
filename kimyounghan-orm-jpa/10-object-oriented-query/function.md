# JPQL 함수

## 기본 함수

- CONCAT
- SUBSTRING
- TRIM
- LOWER, UPPER
- LENGTH
- LOCATE
- ABS, SQRT, MOD
- SIZE, INDEX(JPA 용도)

위의 함수는 기본 스펙에 정의된 함수로 DB 종류에 상관없이 자유롭게 사용할 수 있다.

```java
public class JpaMain {

  public static void main(String[] args) {
    Member member = new Member();
    member.setName("관리자");
    em.persist(member);

    em.flush();
    em.clear();

    String query = "select concat ('a', 'b') From Member m";

    List<String> result = em.createQuery(query, String.class).getResultList();

    for (String s : result) {
      System.out.println("s = " + s);
    }

    tx.commit();
  }
}
```

![](../../.gitbook/assets/kimyounghan-orm-jpa/10/screenshot%202021-05-23%20오후%201.17.28.png)

## 사용자 정의 함수

```sql
select function('group_concat', i.name)
from Item i
```

하이버네이트는 사용하기 전에 방언에 추가해줘야 한다. 사용하는 DB 방언을 상속받고 사용자 정의 함수를 등록한다.

MySQL의 경우 `registerFunction()`으로 커스텀 함수를 정의하고 있다. DB에 종속적으로 되어버리지만 미리 등록된 걸 사용할 수 있다는 이점이 있다.

{% tabs %} {% tab title="MyH2Dialect.java" %}

```java
public class MyH2Dialect extends H2Dialect {

  public MyH2Dialect() {
    // 등록하는 방법은 라이브러리 소스 코드를 참고해야 한다.
    registerFunction("group_concat", new StandardSQLFunction("group_concat", StandardBasicTypes.STRING));
  }
}

```

{% endtab %} {% tab title="persistence.xml" %}

```xml
<?xml version="1.0" encoding="UTF-8"?>
<persistence version="2.2"
  xmlns="http://xmlns.jcp.org/xml/ns/persistence"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/persistence http://xmlns.jcp.org/xml/ns/persistence/persistence_2_2.xsd">

  <persistence-unit name="hello">
    <properties>
      <property name="javax.persistence.jdbc.driver" value="org.h2.Driver"/>
      <property name="javax.persistence.jdbc.user" value="sa"/>
      <property name="javax.persistence.jdbc.password" value=""/>
      <property name="javax.persistence.jdbc.url" value="jdbc:h2:tcp://localhost/~/jpashop"/>
      <!-- 커스텀한 dialect를 추가한다.  -->
      <property name="hibernate.dialect" value="jpabook.jpashop.dialect.MyH2Dialect"/>
      <property name="hibernate.show_sql" value="true"/>
      <property name="hibernate.format_sql" value="true"/>
      <property name="hibernate.use_sql_comments" value="true"/>
      <property name="hibernate.jdbc.batch_size" value="10"/>
      <property name="hibernate.hbm2ddl.auto" value="create"/>
    </properties>
  </persistence-unit>
</persistence>
```

{% endtab %} {% tab title="JpaMain.java" %}

```java
public class JpaMain {

  public static void main(String[] args) {
      Member member1 = new Member();
      member1.setName("관리자1");
      em.persist(member1);

      Member member2 = new Member();
      member2.setName("관리자2");
      em.persist(member2);

      em.flush();
      em.clear();

      String query = "select function('group_concat', m.name) From Member m";

      List<String> result = em.createQuery(query, String.class).getResultList();

      for (String s : result) {
        System.out.println("s = " + s);
      }

      tx.commit();
  }
}

```

{% endtab %} {% endtabs %}

![](../../.gitbook/assets/kimyounghan-orm-jpa/10/screenshot%202021-05-23%20오후%201.25.48.png)

커스텀한 함수를 생성, 등록한 뒤 두 데이터를 한 줄로 붙여서 출력했다.
