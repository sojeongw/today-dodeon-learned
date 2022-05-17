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

## 새로운 엔티티를 구별하는 방법

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

{% tabs %} {% tab title="Item.java" %}

```java

@Entity
@Getter
public class Item {

    @Id
    @GeneratedValue
    private Long id;
}
```

{% endtab %} {% tab title="ItemRepositoryTest.java" %}

```java

@SpringBootTest
class ItemRepositoryTest {

    @Autowired
    ItemRepository itemRepository;

    @Test
    void save() {
        Item item = new Item();
        // id는 JPA에 persist를 할 때 들어간다.
        itemRepository.save(item);
    }
}
```

{% endtab %} {% endtabs %}

![](../../.gitbook/assets/kimyounghan-spring-data-jpa/05/screenshot%202022-05-22%20오후%203.52.35.png)

- 새로운 객체이므로 id가 null이다.

{% tabs %} {% tab title="Item.java" %}

```java

@Entity
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class Item {

    @Id
    private String id;

    public Item(String id) {
        this.id = id;
    }
}
```

{% endtab %} {% tab title="ItemRepositoryTest.java" %}

```java

@SpringBootTest
class ItemRepositoryTest {

    @Autowired
    ItemRepository itemRepository;

    @Test
    void save() {
        Item item = new Item("A");
        itemRepository.save(item);
    }
}
```

![](../../.gitbook/assets/kimyounghan-spring-data-jpa/05/screenshot%202022-05-22%20오후%203.58.46.png)

```sql
select item0_.id as id1_0_0_
from item item0_
where item0_.id = 'A';
insert into item (id)
values ('A');
```

- 식별자에 값이 있으므로 병합한다.
- DB에 값이 있다는 걸 가정하는 것이기 때문에 일단 있는지 쿼리한다.

### Persistable

```java
package org.springframework.data.domain;

public interface Persistable<ID> {
    ID getId();

    boolean isNew();
}
```

- 어떤 이슈로 임의의 ID를 직접 생성해야 한다면 Persistable를 implement 한다.

{% tabs %} {% tab title="Item.java" %}

```java

@Entity
@EntityListeners(AuditingEntityListener.class)
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class Item implements Persistable<String> {
    @Id
    private String id;

    @CreatedDate
    private LocalDateTime createdDate;

    public Item(String id) {
        this.id = id;
    }

    @Override
    public String getId() {
        return id;
    }

    // 새 엔티티인지 확인하는 조건을 직접 구현한다.
    @Override
    public boolean isNew() {
        return createdDate == null;
    }
}
```

{% endtab %} {% endtabs %}

```sql
insert into item (created_date, id)
values ('2022-05-22T16:23:28.873+0900', 'A');
```

- 등록 시간인 @CreatedDate를 조합해서 사용하면 새 객체인지 쉽게 구별할 수 있다.
- 똑같은 테스트를 돌리면 select 없이 insert만 치는 것을 확인할 수 있다.