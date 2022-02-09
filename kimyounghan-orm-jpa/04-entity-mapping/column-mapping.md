# 필드와 컬럼 매핑

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

## @Column

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
- 주로 메모리 상에서만 임시로 어떤 값을 보관하고 싶을 때 사용한다.