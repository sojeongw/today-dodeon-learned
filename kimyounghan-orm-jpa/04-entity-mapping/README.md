# 엔티티 매핑

## 객체와 테이블 매핑

### @Entity

JPA가 관리하는 객체. 엔티티라고 한다. JPA를 사용해서 테이블과 매핑할 클래스는 `@Entity`가 필수다.

- 기본 생성자를 필수로 구현해야 한다.
    - 파라미터가 없는 public 또는 protected 생성자
- final 클래스, enum, interface, inner 클래스에 사용할 수 없다.
- DB 저장할 필드에는 final을 사용할 수 없다.
- name 속성
    - JPA에서 사용할 엔티티 이름을 지정한다.
    - 기본값으로 클래스 이름을 그대로 사용한다.
    - 같은 이름이 있는 게 아니라면 가급적 기본값을 사용한다.

### @Table

엔티티와 매핑할 테이블을 지정한다.

- name 속성
    - 매핑할 테이블 이름을 지정한다.
    - 실제 쿼리도 name에 지정된 테이블로 나간다.
    - 엔티티 이름을 기본값으로 사용한다.
- catalog 속성
    - 데이터베이스 catalog 매핑
- schema 속성
    - 데이터베이스 schema 매핑
- uniqueConstraints(DDL)
    - DDL 생성 시에 유니크 제약 조건 생성

## 데이터베이스 스키마 자동 생성

JPA는 애플리케이션 실행 시점에 DDL을 자동 생성하게 할 수 있다. 데이터베이스 방언을 활용해 DB에 맞는 DDL을 생성한다.

미리 테이블을 만들지 않아도 되므로 객체 중심의 개발을 할 수 있는 장점이 있다. 이 기능은 운영에선 쓰면 안되고 개발 단계에서 쓰면 좋다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<persistence version="2.2"
  xmlns="http://xmlns.jcp.org/xml/ns/persistence"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/persistence http://xmlns.jcp.org/xml/ns/persistence/persistence_2_2.xsd">

  <persistence-unit name="hello">
    <properties>
      ...
      <property name="hibernate.hbm2ddl.auto" value="create"/>
    </properties>
  </persistence-unit>
</persistence>
```

`hibernate.hbm2ddl.auto`에 자동 생성과 관련된 옵션을 설정할 수 있다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/04/스크린샷%202021-03-14%20오후%206.42.03.png)

create는 시작 시에 drop을 하고 drop-table은 마지막에 drop으로 날려버린다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/04/스크린샷%202021-03-14%20오후%206.48.31.png)

update는 새로 컬럼을 추가했을 때 그 내용만 반영한다. 실제 쿼리도 위와 같이 동작한다. 컬림을 지운 것에 대해서는 아무 쿼리도 동작하지 않는다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/04/스크린샷%202021-03-14%20오후%206.51.03.png)

validate 옵션을 적용하면 테이블에 없는 칼럼을 클래스에 필드로 추가했을 경우 에러가 난다.

none은 사실상 무의미한 값을 옵션에 넣는 것과 같다. 관례상 집어넣는 값이다.

### 주의 사항

- 운영 장비에는 절대 create, create-drop, update를 사용하면 안된다.
    - 테이블이 날아가거나 update 시에는 alter를 치면서 락이 걸려 서비스가 중단된다.
- 개발 초기 단계에는 create, update
- 테스트 서버는 update, validate
- 스테이징과 운영 서버는 validate, none

### DDL 생성 기능

`@Column(nullable = false, length = 10)`

`@Table(uniqueConstraints = {@UniqueConstraint( name = "NAME_AGE_UNIQUE", columnNames = {"NAME", "AGE"} )})`

이런 조건들은 DDL 생성에만 관여한다. 이런 DDL 생성 기능은 DDL을 자동 생성할 때만 사용되고 JPA의 실행 로직에는 영향을 주지 않는다.

## 필드와 컬럼 매핑

아래와 같은 요구 사항이 추가되었다고 해보자.

- 회원은 일반 회원과 관리자로 구분해야 한다.
- 회원 가입일과 수정일이 있어야 한다.
- 회원을 설명할 수 있는 필드가 있어야 한다. 이 필드는 길이 제한이 없다.

```java

@Entity
public class Member {

  @Id
  private Long id;

  @Column(name = "name")
  private String username;

  private Integer age;

  // DB에는 enum 타입이 없어서 이 애너테이션을 달아줘야 한다.
  @Enumerated(EnumType.STRING)
  private RoleType roleType;

  // 날짜 타입은 @Temporal을 달아준다.
  // DB는 DATE, TIME, TIMESTAMP로 나뉘기 때문에 정보를 줘야 한다.
  @Temporal(TemporalType.TIMESTAMP)
  private Date createdDate;

  @Temporal(TemporalType.TIMESTAMP)
  private Date lastModifiedDate;

  // varchar를 넘어서는 큰 컨텐츠를 넣고 싶을 때 사용한다.
  // String 타입이면 DB에서 clob으로 생성된다.
  @Lob
  private String description;
  
  // getter, setter
}
```

![](../../.gitbook/assets/kimyounghan-orm-jpa/04/스크린샷%202021-03-15%20오후%2012.39.50.png)
  
### @Column

![](../../.gitbook/assets/kimyounghan-orm-jpa/04/스크린샷%202021-03-15%20오후%2012.40.02.png)

- insertable, updatable
  - 해당 column이 수정됐을 때 DB에 insert, update를 할 건지 결정한다.
  - 즉 insert, update 문이 나갈 때 반영할 것인지를 의미한다.
  - 기본값이 true로 되어있다.
  
![](../../.gitbook/assets/kimyounghan-orm-jpa/04/스크린샷%202021-03-16%20오전%209.13.54.png)

- unique
  - 잘 안 쓴다. unique 키가 랜덤한 이름으로 나와서 알아보기 힘들다.
  - `@Table`에 uniqueConstrains 옵션을 주면 이름을 설정할 수 있어서 이 방법을 더 많이 사용한다.

## @Enumerated

![](../../.gitbook/assets/kimyounghan-orm-jpa/04/스크린샷%202021-03-15%20오후%2012.40.10.png)

자바 enum 타입을 매핑할 때 사용한다. `ORDINAL`의 경우, 요구 사항이 추가되어 맨 앞에 다른 enum이 추가되면 그 값이 0이 되는 등 문제가 많아서 사용하지 않는다.

## @Temporal

![](../../.gitbook/assets/kimyounghan-orm-jpa/04/스크린샷%202021-03-15%20오후%2012.40.18.png)

날짜 타입 `java.util.Date`, `java.util.Calendar`를 매핑할 때 사용한다. `LocalDate`, `LocalDateTime`을 사용할 때는 생략 가능하다.

## @Lob

데이터베이스 blob과 clob 타입을 매핑한다. 지정할 수 있는 속성은 없으며 문자면 clob, 나머지는 blob으로 매핑된다.

- clob
  - String, char[], java.sql.CLOB
- blob
  - byte[], java.sql.BLOB
  
## @Transient

- 필드 매핑을 하지 않을 때 사용한다. 
- 데이터베이스에 저장이나 조회가 되지 않는다.
- 주로 메모리 상에서만 임시로 어떤 값을 보관하고 싶을 떄 사용한다.
