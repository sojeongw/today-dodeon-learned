# 프로젝트 생성 및 애플리케이션 개발

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>jpa-basic</groupId>
  <artifactId>hello-jpa</artifactId>
  <version>1.0.0</version>

  <dependencies>
    <!--  Java 11 사용 시 추가 필요  -->
    <dependency>
      <groupId>javax.xml.bind</groupId>
      <artifactId>jaxb-api</artifactId>
      <version>2.3.1</version>
    </dependency>
    <!-- JPA 하이버네이트 -->
    <dependency>
      <groupId>org.hibernate</groupId>
      <artifactId>hibernate-entitymanager</artifactId>
      <version>5.4.1.Final</version>
    </dependency>
    <!-- H2 데이터베이스 -->
    <dependency>
      <groupId>com.h2database</groupId>
      <artifactId>h2</artifactId>
      <version>1.4.200</version>
    </dependency>
  </dependencies>

  <properties>
    <maven.compiler.source>11</maven.compiler.source>
    <maven.compiler.target>11</maven.compiler.target>
  </properties>

</project>
```

- Java 8
- Maven
    - groupId: jpa-basic
    - artifactId: ex1-hello-jpa
    - version: 1.0.0

## JPA 설정하기

### persistence.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<persistence version="2.2"
  xmlns="http://xmlns.jcp.org/xml/ns/persistence"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/persistence http://xmlns.jcp.org/xml/ns/persistence/persistence_2_2.xsd">

  <!-- JPA 이름 설정: 데이터베이스 당 하나를 부여한다. -->
  <persistence-unit name="hello">
    <properties>
      <!-- 필수 속성: DB 접속 정보 -->
      <property name="javax.persistence.jdbc.driver" value="org.h2.Driver"/>
      <property name="javax.persistence.jdbc.user" value="sa"/>
      <property name="javax.persistence.jdbc.password" value=""/>
      <property name="javax.persistence.jdbc.url" value="jdbc:h2:tcp://localhost/~/test"/>
      <!-- JPA는 특정 DB에 종속적이지 않아 hibernate 전용 옵션을 추가해줘야 한다.  -->
      <property name="hibernate.dialect" value="org.hibernate.dialect.H2Dialect"/>
      <!-- 옵션 -->
      <property name="hibernate.show_sql" value="true"/>
      <property name="hibernate.format_sql" value="true"/>
      <property name="hibernate.use_sql_comments" value="true"/>
      <!--<property name="hibernate.hbm2ddl.auto" value="create" />-->
    </properties>
  </persistence-unit>
</persistence>
```

- resources/META-INF/persistence.xml
- persistence-unit
    - 이름 지정
- javax.persistence
    - JPA 표준 속성
- hibernate
    - 하이버네이트 전용 속성

![](../../.gitbook/assets/kimyounghan-orm-jpa/02/스크린샷%202021-03-13%20오후%203.20.17.png)

- H2Dialect
    - JPA는 특정 DB에 종속되지 않는다.
    - 각각의 DB가 제공하는 SQL 문법과 함수가 조금씩 다르다.
    - 이렇게 SQL 표준을 지키지 않는 특정 DB만의 고유한 기능을 방언이라고 한다.

## 애플리케이션 개발

### JPA 구동 방식

![](../../.gitbook/assets/kimyounghan-orm-jpa/02/스크린샷%202021-03-13%20오후%203.32.17.png)

JPA의 `Persistence` 클래스가 `META-INF/persistence.xml` 설정 파일을 읽어서 `EntityManagerFactory`라는 클래스를 생성한다.
여기서 필요할 때마다 `EntityManager`를 만든다.

### 회원 생성

```sql
create table Member
(
    id   bigint not null,
    name varchar(255),
    primary key (id)
);
```

{% tabs %} {% tab title="JpaMain.java" %}

```java
public class JpaMain {

  public static void main(String[] args) {
    // persistence.xml에서 설정했던 unit-name을 넘긴다.
    // 엔티티 매니저 팩토리는 애플리케이션 로딩 시점에 딱 하나만 만들어야 한다.
    EntityManagerFactory entityManagerFactory = Persistence.createEntityManagerFactory("hello");

    // 엔티티 매니저를 꺼낸다.
    // 엔티티 매니저는 트랜잭션마다 만들어줘야 한다.
    EntityManager entityManager = entityManagerFactory.createEntityManager();

    // JPA의 모든 작업은 트랜잭션 내에서 해야한다.
    EntityTransaction tx = entityManager.getTransaction();
    tx.begin();

    try {
      // 실제 동작 코드를 작성한다.
      Member member = new Member();
      member.setId(1L);
      member.setName("helloA");

      // 저장한다.
      entityManager.persist(member);

      // 커밋한다.
      tx.commit();
    } catch (Exception e) {
      tx.rollback();
    } finally {
      // 엔티티 매니저가 내부적으로 데이터베이스 커넥션을 물고 동작하기 때문에 쓰고 나서 꼭 닫아줘야 한다.
      entityManager.close();
    }

    // 전체 애플리케이션이 끝나면 팩토리까지 닫아준다.
    entityManagerFactory.close();
  }
}
```

{% endtab %} {% tab title="Member.java" %}

```java
// JPA가 사용하는 데이터라고 인식한다.
@Entity
public class Member {

  @Id
  private Long id;
  private String name;

  public Long getId() {
    return id;
  }

  public void setId(Long id) {
    this.id = id;
  }

  public String getName() {
    return name;
  }

  public void setName(String name) {
    this.name = name;
  }
}
```

{% endtab %} {% endtabs %}

### 회원 수정

{% tabs %} {% tab title="JpaMain.java" %}

```java
public class JpaMain {

  public static void main(String[] args) {
    EntityManagerFactory entityManagerFactory = Persistence.createEntityManagerFactory("hello");
    EntityManager entityManager = entityManagerFactory.createEntityManager();
    EntityTransaction tx = entityManager.getTransaction();
    tx.begin();

    try {
      // 수정할 대상을 가져온다.
      Member findMember = entityManager.find(Member.class, 1L);
      System.out.println("id: " + findMember.getId());
      System.out.println("name: " + findMember.getName());

      findMember.setName("helloJPA");

      // 수정한 객체를 따로 저장하지 않아도 된다. 
      // 데이터를 JPA를 통해 가져오면 변경 여부를 트랜잭션 커밋 시점에
      // 다 체크해서 바뀐 내용에 대해 업데이트 쿼리를 만들어 날린다.

      tx.commit();
    } catch (Exception e) {
      tx.rollback();
    } finally {
      entityManager.close();
    }

    entityManagerFactory.close();
  }
}
```

{% endtab %} {% endtabs %}

## 주의할 점

- 엔티티 매니저 팩토리는 애플리케이션 실행 시점에 하나만 생성해서 애플리케이션 전체에 걸쳐 공유한다.
- 엔티티 매니저는 요청이 왔을 때 썼다가 끝나면 버리는 사이클을 반복한다.
    - 그래서 절대 스레드 간에 공유해서는 안된다. 사용하고 바로 버려야 한다.
- JPA의 모든 데이터 변경은 트랜잭션 안에서 실행해야 한다.

## JPQL

`EntityManger.find()`는 가장 단순한 조회 방법이다. 만약 나이가 18살 이상인 회원을 모두 검색하고 싶다면 JPQL을 사용해야 한다.

{% tabs %} {% tab title="JpaMain.java" %}

```java
public class JpaMain {

  public static void main(String[] args) {
    EntityManagerFactory entityManagerFactory = Persistence.createEntityManagerFactory("hello");
    EntityManager entityManager = entityManagerFactory.createEntityManager();
    EntityTransaction tx = entityManager.getTransaction();
    tx.begin();

    try {
      // JPA는 테이블 대상이 아니라 객체를 대상으로 쿼리를 짠다.
      // 따라서 Member는 테이블이 아니라 객체를 가리킨다.
      List<Member> result = entityManager
          .createQuery("select m from Member as m", Member.class)
          // 페이지네이션을 할 수도 있다. 아래는 1번부터 10개 가져온다.
          .setFirstResult(1)
          .setMaxResults(10)
          .getResultList();

      for (Member member : result) {
        System.out.println("name: " + member.getName());
      }

      tx.commit();
    } catch (Exception e) {
      tx.rollback();
    } finally {
      entityManager.close();
    }

    entityManagerFactory.close();
  }
}
```

{% endtab %} {% endtabs %}

JPA를 사용하면 엔티티 객체를 중심으로 개발하게 되는데 문제는 검색 쿼리다. 검색할 때 테이블이 아닌 엔티티 객체를 대상으로 검색하는데, 모든 DB 데이터를 객체로 변환해서
검색하는 것은 불가능하다.

애플리케이션이 필요한 데이터만 DB에서 불러오려면 결국 검색 조건이 포함된 SQL이 필요하다. 데이터베이스에서 데이터를 최대한 필터링이 필요한데 쿼리를 쓰면 SQL에 종속된다.
이때 객체를 대상으로 검색할 수 있게 하는 기술이 JPQL이다.

### 특징

- JPA가 제공하는 SQL을 추상화한 객체 지향 쿼리 언어
    - 한마디로 객체지향 SQL이다.
    - 추상화했기 때문에 특정 데이터베이스에 의존하지 않는다.
- SQL과 문법이 유사하다.
    - select, from, where, group by, having, join을 지원한다.
- 엔티티 객체를 대상으로 쿼리한다.
    - SQL은 데이터베이스 테이블을 대상으로 쿼리한다.