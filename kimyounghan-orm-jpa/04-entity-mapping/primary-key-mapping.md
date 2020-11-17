# 기본 키 매핑

## 직접 할당

`@Id`만 사용한다.

## 자동 생성

`@GeneratedValue`를 추가한다.
  
### IDENTITY

JPA는 보통 트랜잭션 커밋 시점에 insert SQL을 실행한다. AUTO_INCREMENT는 데이터베이스에 insert SQL을 실행한 뒤에 ID 값을 알 수 있다. em.persist() 시점에 즉시 insert SQL을 실행하고 DB에서 식별자를 조회한다.

- 기본 키 생성을 데이터베이스에 위임한다.
- 주로 MySQL, PostgreSQL, SQL Server, DB2에서 사용한다.
    - ex. MySQL의 AUTO_INCREMENT
