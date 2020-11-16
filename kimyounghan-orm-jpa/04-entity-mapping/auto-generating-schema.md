# 데이터베이스 스키마 자동 생성

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

## 주의 사항

- 운영 장비에는 절대 create, create-drop, update를 사용하면 안된다.
    - 테이블이 날아가거나 update 시에는 alter를 치면서 락이 걸려 서비스가 중단된다.
- 개발 초기 단계에는 create, update
- 테스트 서버는 update, validate
- 스테이징과 운영 서버는 validate, none

## DDL 생성 기능

`@Column(nullable = false, length = 10)`

`@Table(uniqueConstraints = {@UniqueConstraint( name = "NAME_AGE_UNIQUE", columnNames = {"NAME", "AGE"} )})`

이런 조건들은 DDL 생성에만 관여한다. 이런 DDL 생성 기능은 DDL을 자동 생성할 때만 사용되고 JPA의 실행 로직에는 영향을 주지 않는다.