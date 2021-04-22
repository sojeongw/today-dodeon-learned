# MappedSuperclass

![](../../.gitbook/assets/kimyounghan-orm-jpa/07/screenshot%202021-05-16%20오후%207.50.31.png)

![](../../.gitbook/assets/kimyounghan-orm-jpa/07/screenshot%202021-05-16%20오후%207.50.38.png)

공통 매핑 정보가 필요할 떄 사용한다.

```java
public class Member {

  ...
  
  private String createdBy;
  
  private LocalDateTime createdDate;
  
  private String lastModifiedBy;
  
  private LocalDateTime lastModifiedDate;
}
```

예를 들어 생성, 수정 정보를 모든 클래스에 추가해야 한다고 해보자.

{% tabs %} {% tab title=".java" %}

```java
@MappedSuperclass
public abstract class BaseEntity {
  private String createdBy;
  private LocalDateTime createdDate;
  private String lastModifiedBy;
  private LocalDateTime lastModifiedDate;
}
```

{% endtab %} {% tab title=".java" %}

```java
public class Member extends BaseEntity {

}
```

{% endtab %} {% tab title=".java" %}

```java
public class JpaMain {

  public static void main(String[] args) {
    ...
    
    try {
      Member member = new Member();
      member.setCreatedBy("kim");
      member.setCreatedDate(LocalDateTime.now());

      em.persist(member);

      tx.commit();
    } catch (Exception e) {
      tx.rollback();
    } finally {
      em.close();
    }

    entityManagerFactory.close();
  }
}

```

{% endtab %} {% endtabs %}

공통된 속성을 클래스로 만들어 상속을 받는다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/07/screenshot%202021-05-16%20오후%208.13.42.png)

그럼 테이블 생성시에도 자동으로 컬럼이 들어간 것을 볼 수 있다.

- 상속 관계를 매핑하는 것이 아니다.
- 테이블과 관계없이 단순히 엔티티가 공통으로 사용하는 매핑 정보를 모으기만 한다.
    - 즉, `@MappedSuperclass`가 달린 클래스는 엔티티가 아니며 테이블과 매핑되지도 않는다.
    - 부모 클래스를 상속받는 자식 클래스에 매핑 정보만 제공한다.
    - 따라서 em.find(BaseEntity)와 같은 조회나 검색이 불가능하다.
- 직접 생성해서 사용할 일이 없기 때문에 추상 클래스로 만들길 권장한다.
- 등록일, 수정일 등 전체 엔티티에서 공통으로 적용하는 정보를 모을 때 사용한다.
- `@Entity` 클래스는 엔티티나 `@MappedSuperclass`로 지정한 클래스만 상속할 수 있다.