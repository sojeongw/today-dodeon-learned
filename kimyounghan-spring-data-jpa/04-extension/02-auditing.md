# Auditing

- 엔티티를 생성, 변경할 때 변경한 사람이나 시간을 추적하고 싶을 때 사용한다.
- 웬만하면 모든 엔티티에 다 적용해줘야 운영해서 편하다.

## 순수 JPA 사용

{% tabs %} {% tab title="JpaBaseEntity.java" %}

```java
// 상속처럼 프로퍼티를 테이블에서 내려서 쓸 수 있는 애너테이션
// 진짜 상속과는 다르다.
@MappedSuperclass
@Getter
public class JpaBaseEntity {

    // 실수로라도 DB 값이 변경되지 않게 막는다.
    @Column(updatable = false)
    private LocalDateTime createdDate;

    private LocalDateTime updatedDate;

    // 영속화 하기 전에 발생시키는 이벤트
    @PrePersist
    public void prePersist() {
        LocalDateTime now = LocalDateTime.now();
        createdDate = now;
        // 등록과 수정을 처음부터 똑같이 맞춰둔다.
        // 값에 null이 있으면 쿼리가 지저분해질 수 있고 created와 값이 같으면 최초 값이라는 것을 알 수 있어 편하다.
        updatedDate = now;
    }

    @PreUpdate
    public void preUpdate() {
        updatedDate = LocalDateTime.now();
    }
}
```

{% endtab %} {% tab title="JpaBaseEntity.java" %}

```java

@Entity
public class Member extends JpaBaseEntity {

}
```

{% endtab %} {% endtabs %}

```sql
create table member
(
    member_id    bigint  not null,
    created_date timestamp,
    updated_date timestamp,
    age          integer not null,
    username     varchar(255),
    team_id      bigint,
    primary key (member_id)
)
```

- JPA 주요 이벤트 애너테이션
    - @PrePersist
    - @PostPersist
    - @PreUpdate
    - @PostUpdate

```java
class MemberTest {
    @Test
    public void JpaEventBaseEntity() throws Exception {
        Member member = new Member("member1");
        // @PrePersist 발생
        memberRepository.save(member);

        Thread.sleep(100);
        member.setUsername("member2");

        // @PreUpdate 발생
        em.flush();
        em.clear();

        Member findMember = memberRepository.findById(member.getId()).get();

        System.out.println("findMember.createdDate = " + findMember.getCreatedDate());
        System.out.println("findMember.updatedDate = " + findMember.getUpdatedDate());
    }
}
```

```text
findMember.createdDate = 2022-05-22T11:13:25.287759
findMember.updatedDate = 2022-05-22T11:13:25.430480
```

- JpaBaseEntity만 만들어 놓으면 여러 엔티티에 공통으로 사용할 수 있어 편리하다.

## 스프링 데이터 JPA 사용

- 설정
    - 스프링 부트 설정 클래스
        - @EnableJpaAuditing
    - 엔티티
        - @EntityListeners(AuditingEntityListener.class)
- 사용 애너테이션
    - @CreatedDate
    - @LastModifiedDate
    - @CreatedBy
    - @LastModifiedBy

### 등록일, 수정일

{% tabs %} {% tab title="BaseEntity.java" %}

```java

@Getter
@EntityListeners(AuditingEntityListener.class)
@MappedSuperclass
public class BaseEntity {

    @CreatedDate
    @Column(updatable = false)
    private LocalDateTime createdDate;

    @LastModifiedDate
    private LocalDateTime lastModifiedDate;
}

```

{% endtab %} {% tab title="DataJpaApplication.java" %}

```java

@EnableJpaAuditing
@SpringBootApplication
public class DataJpaApplication {

    public static void main(String[] args) {
        SpringApplication.run(DataJpaApplication.class, args);
    }

}
```

{% endtab %} {% tab title="BaseEntity.java" %}

```java
public class Member extends BaseEntity {

}
```

{% endtab %} {% endtabs %}

```text
findMember.createdDate = 2022-05-22T11:27:49.486753
findMember.getLastModifiedDate = 2022-05-22T11:27:49.639758
```

### 등록자, 수정자

{% tabs %} {% tab title=BaseEntity.java" %}

```java

@Getter
@EntityListeners(AuditingEntityListener.class)
@MappedSuperclass
public class BaseEntity {

    ...

    @CreatedBy
    @Column(updatable = false)
    private String createdBy;

    @LastModifiedBy
    private String lastModifiedBy;
}

```

{% endtab %} {% tab title="DataJpaApplication.java" %}

```java

@EnableJpaAuditing
@SpringBootApplication
public class DataJpaApplication {

    public static void main(String[] args) {
        SpringApplication.run(DataJpaApplication.class, args);
    }

    // 등록자, 수정자를 처리해주는 AuditorAware 스프링 빈을 등록한다.
    @Bean
    public AuditorAware<String> auditorProvider() {
        // 실제로는 스프링 시큐리티 정보나 HTTP 세션에서 가져온다.
        return () -> Optional.of(UUID.randomUUID().toString());
    }
}
```

{% endtab %} {% endtabs %}

```text
findMember.getCreatedBy = 90634e5c-5270-448f-be1f-8742c96292f6
findMember.getLastModifiedBy = 7514ab5c-cdc7-4981-ac9d-4c3bd715dd34
```

- 등록이나 수정할 때마다 auditorProvider 빈을 호출한 뒤 결과물을 채운다.

### 전체 적용

{% tabs %} {% tab title="META-INF/orm.xml" %}

```xml
<?xml version=“1.0” encoding="UTF-8”?>
<entity-mappings xmlns=“http://xmlns.jcp.org/xml/ns/persistence/orm”
        xmlns:xsi=“http://www.w3.org/2001/XMLSchema-instance”
        xsi:schemaLocation=“http://xmlns.jcp.org/xml/ns/persistence/
        orm http://xmlns.jcp.org/xml/ns/persistence/orm_2_2.xsd”
        version=“2.2">
<persistence-unit-metadata>
<persistence-unit-defaults>
    <entity-listeners>
        <entity-listener
                class="org.springframework.data.jpa.domain.support.AuditingEntityListener”/>
                </entity-listeners>
            </persistence-unit-defaults>
        </persistence-unit-metadata>
    </entity-mappings>
```

{% endtab %} {% endtabs %}

- @EntityListeners(AuditingEntityListener.class)를 생략하고 엔티티 전체에 적용한다.

## 참고

```java
public class BaseTimeEntity {
    @CreatedDate
    @Column(updatable = false)
    private LocalDateTime createdDate;
    @LastModifiedDate
    private LocalDateTime lastModifiedDate;
}

public class BaseEntity extends BaseTimeEntity {
    @CreatedBy
    @Column(updatable = false)
    private String createdBy;
    @LastModifiedBy
    private String lastModifiedBy;
}
```

- 실무에서는 시간은 필요해도 등록자, 수정자는 필요없을 수 있다.
- 따라서 Base 타입을 분리하고 원하는 타입을 선택해서 상속하면 된다.