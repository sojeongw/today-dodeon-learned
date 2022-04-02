# 데이터베이스 스키마 자동 생성

- JPA는 애플리케이션 실행 시점에 DDL을 자동 생성하게 할 수 있다.
- 데이터베이스 방언을 활용해 DB에 맞는 DDL을 생성한다.
- 미리 테이블을 만들지 않아도 되므로 객체 중심의 개발을 할 수 있는 장점이 있다.
- 운영에선 쓰면 안되고 개발 단계에서 쓰면 좋다.
- 각 SQL 방언에 맞춰 생성해준다.


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

- create
    - 시작 시에 drop을 먼저 실행한다.
- create-drop
    - 마지막에 drop으로 날려버린다.
    - 테스트 같이 결과가 남지 않아야 할 때 사용한다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/04/스크린샷%202021-03-14%20오후%206.48.31.png)

- update
    - 새로 칼럼을 추가했을 때 그 내용만 반영한다.
- 만약 age 칼럼을 추가했는데 테이블을 새로 만들고 싶지 않다면 추가한 칼럼에 대해 alter를 수행한다.
- 칼럼을 지운 것에 대해서는 아무 쿼리도 동작하지 않는다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/04/스크린샷%202021-03-14%20오후%206.51.03.png)

- validate
    - 테이블에 없는 칼럼을 클래스에 필드로 추가했을 경우 에러가 난다.
- none
    - 사실상 무의미한 값을 옵션에 넣는 것과 같다. 관례상 집어넣는 값이다.

## 주의 사항

- 운영 장비에는 절대 create, create-drop, update를 사용하면 안된다.
    - 테이블을 drop이나 update 하면 alter를 치면서 락이 걸려 서비스가 중단된다.
- 개발 초기 단계
    - create
    - update
- 테스트 서버
    - update
    - validate
- 스테이징, 운영 서버
    - validate
    - none

사실 로컬을 제외한 개발 등의 서버는 직접 스크립트 짜서 반영하는 걸 권장한다.

## DDL 생성 기능

### 제약 조건 추가

```text
@Column(nullable = false, length = 10)
```

### 유니크 제약 조건 추가

```text
@Table(uniqueConstraints = {@UniqueConstraint( name = "NAME_AGE_UNIQUE", columnNames = {"NAME", "AGE"} )})
```

- 위 조건들은 JPA 실행 자체는 영향을 주지 않고 DB에 영향을 준다.
- JPA가 실행될 때 이 애너테이션을 보고 런타임 때 뭔가 바뀌거나 하는 게 아니다.
- 즉, DDL 생성에만 관여한다. 
