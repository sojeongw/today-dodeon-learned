# 스프링 데이터 JPA 분석

## 구현체

- org.springframework.data.jpa.repository.support.SimpleJpaRepository
- 스프링 데이터 JPA가 제공하는 공통 인터페이스의 구현체

```java

@Repository
@Transactional(readOnly = true)
public class SimpleJpaRepository<T, ID> implements JpaRepositoryImplementation<T, ID> {

}
```

### @Repository

- JPA 예외를 스프링이 추상화 한 예외로 변환한다.
    - 기술이 바뀌어도 영향을 주지 않도록 분리할 수 있다.

### @Transactional

```java

@Repository
@Transactional(readOnly = true)
public class SimpleJpaRepository<T, ID> implements JpaRepositoryImplementation<T, ID> {

    // 변경 사항은 읽기 전용이 아닌 트랜잭션을 사용하도록 내부적으로 설정되어 있다.
    @Override
    @Transactional
    public void deleteAll(Iterable<? extends T> entities) {

        Assert.notNull(entities, "Entities must not be null!");

        for (T entity : entities) {
            delete(entity);
        }
    }
}
```

- 트랜잭션을 적용한다.
    - JPA의 모든 변경은 트랜잭션 안에서 동작한다.
    - 스프링 데이터 JPA는 등록, 수정, 삭제 등 변경 메서드를 트랜잭션 처리한다.
- 서비스 계층에서 트랜잭션을 시작하지 않으면
    - 리파지토리에서 트랜잭션을 시작한다.
    - 그래서 예제에서 @Transactional 애너테이션이 없어도 데이터 등록, 변경이 가능했다.
    - 트랜잭션이 리포지토리 계층에 이미 걸려있는 것이다.
- 서비스 계층에서 트랜잭션을 시작하면
    - 리파지토리는 해당 트랜잭션을 전파 받아서 사용한다.

### @Transactional(readOnly = true)

- 단순 조회만 하고 변경은 하지 않는 트랜잭션에 사용한다.
    - flush를 생략해서 약간의 성능 향상을 얻을 수 있다.
    - JPA 책 15.4.2를 참고한다.

### save()

```java

@Repository
@Transactional(readOnly = true)
public class SimpleJpaRepository<T, ID> implements JpaRepositoryImplementation<T, ID> {

    @Transactional
    @Override
    public <S extends T> S save(S entity) {

        Assert.notNull(entity, "Entity must not be null.");

        if (entityInformation.isNew(entity)) {
            em.persist(entity);
            return entity;
        } else {
            return em.merge(entity);
        }
    }
}
```

- 새로운 엔티티는 저장한다.
    - 객체의 식별자가 null 일 때
    - 기본 타입의 식별자가 0일 때
    - Persistable 인터페이스를 구현해서 판단 로직을 변경할 수 있다.
- 새로운 엔티티가 아니면 병합한다.
    - 기존 값을 새로 들어온 데이터로 교체해버린다.
    - DB에서 select를 무조건 한 번 한다는 단점이 있다.
    - 따라서 데이터 변경은 병합보다는 변경 감지를 활용하는 게 좋다.
    - 병합은 영속 상태 엔티티가 잠시 영속 상태를 벗어났다가 다시 영속 상태가 되어야할 때 사용한다.