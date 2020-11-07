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