# 영속성 관리

![](../../.gitbook/assets/kimyounghan-orm-jpa/03/스크린샷%202021-03-13%20오후%205.12.20.png)

엔티티 매니저 팩토리를 통해 고객의 요청이 올 때마다 엔티티 매니저를 생성한다. 엔티티 매니저는 내부적으로 DB 커넥션을 통해 DB를 사용한다.

## 영속성 컨텍스트

엔티티를 영구 저장하는 환경을 뜻한다. 이전에 `EntityManager.persist(entity);`를 통해 DB에 저장했었는데 정확히 말하자면 저장하는 게 아니라, 영속성
컨텍스트를 통해 엔티티를 영속화 한다는 뜻이다. 더 정확히 말하자면 `persist()`는 엔티티를 영속성 컨텍스트에 저장하는 것이다.

영속성 컨텍스트는 논리적인 개념이며 눈에 보이지 않는다. 엔티티 매니저를 통해 영속성 컨텍스트에 접근한다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/03/스크린샷%202021-03-13%20오후%205.16.34.png)

엔티티 매니저를 생성하면 그 안에 1:1로 영속성 컨텍스트가 생성된다. 엔티티 매니저 안에 영속성 컨텍스트라는 눈에 보이지 않는 공간이 생기는 것이다.

## 엔티티의 생명 주기

![](../../.gitbook/assets/kimyounghan-orm-jpa/03/스크린샷%202021-03-13%20오후%205.23.48.png)

- 비영속(new/transient)
    - 영속성 컨텍스트와 전혀 관계 없는 새로운 상태
- 영속(managed)
    - 영속성 컨텍스트에 관리되는 상태
- 준영속(detached)
    - 영속성 컨텍스트에 저장되었다가 분리된 상태
- 삭제(removed)
    - 삭제된 상태

### 비영속

![](../../.gitbook/assets/kimyounghan-orm-jpa/03/스크린샷%202021-03-13%20오후%205.25.59.png)

member 객체를 생성만 하고 엔티티 매니저에 아무것도 안 넣은 상태다. JPA와 전혀 관계없는 상태라고 할 수 있다.

### 영속

![](../../.gitbook/assets/kimyounghan-orm-jpa/03/스크린샷%202021-03-13%20오후%205.26.06.png)

객체 생성 후에 엔티티 매니저를 가져와서 persist로 객체를 넣어주면, 엔티티 매니저 안에 있는 영속성 컨텍스트에 member가 들어가면서 영속 상태가 된다.

```java
public class JpaMain {

  public static void main(String[] args) {
      ...

    // 여기까지는 아무 상태도 아닌 비영속 상태다.
    Member member = new Member();
    member.setId(1L);
    member.setName("HelloJPA");

    // 여기서부터 영속 상태가 된다. 영속성 컨텍스트 안에서 객체가 관리된다.
    System.out.println("=== before ==="); // 쿼리가 출력되지 않는다.
    entityManager.persist(member);
    System.out.println("=== after ===");  // insert 쿼리가 출력된다.

    // 실제 디비에 저장되는 건 커밋될 때다.
    tx.commit();
    
    ...
  }
}
```

영속 상태가 된다고 해서 DB에 쿼리가 날아가는 게 아니다. commit을 해야 반영된다.

### 준영속

![](../../.gitbook/assets/kimyounghan-orm-jpa/03/스크린샷%202021-03-13%20오후%205.26.14.png)

영속성 컨텍스트와 아무 관계가 아니게 된다.

### 삭제

![](../../.gitbook/assets/kimyounghan-orm-jpa/03/스크린샷%202021-03-13%20오후%205.26.21.png)

실제 DB에 데이터를 삭제하는 상태다.

## 영속성 컨텍스트의 이점
### 1차 캐시

![](../../.gitbook/assets/kimyounghan-orm-jpa/03/스크린샷%202021-03-13%20오후%206.15.26.png)

1차 캐시를 사실상 영속석 컨텍스트라고 생각해도 된다. 영속화를 하게 되면 PK가 키, 엔티티가 값이 되어 1차 캐시에 저장된다. 

![](../../.gitbook/assets/kimyounghan-orm-jpa/03/스크린샷%202021-03-13%20오후%206.15.32.png)

1차 캐시에 저장을 해두면 조회할 때 DB 대신 먼저 1차 캐시를 뒤져서 반환할 수 있다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/03/스크린샷%202021-03-13%20오후%206.15.37.png)

member2가 DB엔 있지만 1차 캐시엔 없다면 DB에서 조회한 데이터를 1차 캐시에 저장한 뒤 반환한다. 이후에 다시 member2를 조회하면 1차 캐시에 있는 member2가 반환된다.

엔티티 매니저는 트랜잭션 단위로 만들고 종료되면 삭제되기 때문에 1차 캐시도 날아간다. 따라서 아주 짧은 찰나의 순간에만 사용된다. 그래서 크게 성능상 이점은 없다.

```java
Member member = new Member();
member.setId(101L);
member.setName("HelloJPA");

System.out.println("=== before ===");
entityManager.persist(member);
System.out.println("=== after ===");

Member findMember = entityManager.find(Member.class, 101L);

System.out.println("findMember.id: " + findMember.getId());
System.out.println("findMember.name: " + findMember.getName());

tx.commit();
```

```text
=== before ===
=== after ===
findMember.id: 101
findMember.name: HelloJPA
Hibernate: 
    /* insert hellojpa.Member
        */ insert 
        into
            Member
            (name, id) 
        values
            (?, ?)
```

member가 1차 캐시에 저장되면서 조회 시 select 쿼리가 나가지 않는 것을 볼 수 있다.

```java
Member findMember1 = entityManager.find(Member.class, 101L);
Member findMember2 = entityManager.find(Member.class, 101L);

tx.commit();
```

```text
Hibernate: 
    select
        member0_.id as id1_0_0_,
        member0_.name as name2_0_0_ 
    from
        Member member0_ 
    where
        member0_.id=?
```

같은 데이터를 2번 조회하면 select 쿼리가 한 번만 나간 걸 볼 수 있다.

### 동일성 보장

![](../../.gitbook/assets/kimyounghan-orm-jpa/03/스크린샷%202021-03-13%20오후%206.31.01.png)

자바 컬렉션에서 가져오면 주소가 같은 것처럼 동일성을 보장해준다. 어렵게 얘기하면 1차 캐시로 반복 가능한 읽기 등급의 트랜잭션 격리 수준을 데이터베이스가 아닌 애플리케이션 차원에서 제공한다고 할 수 있다.

### 트랜잭션을 지원하는 쓰기 지연

![](../../.gitbook/assets/kimyounghan-orm-jpa/03/스크린샷%202021-03-13%20오후%206.33.58.png)

JPA가 쿼리를 쭉쭉 쌓고 있다가 트랜잭션을 커밋하는 순간에 DB에 SQL을 보낸다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/03/스크린샷%202021-03-13%20오후%206.35.49.png)

member를 저장하면 영속성 컨텍스트 안의 1차 캐시에 저장함과 동시에 JPA가 엔티티를 분석해서 SQL을 생성한다. 그리고 그 쿼리를 쓰기 지연 SQL 저장소에 저장한다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/03/스크린샷%202021-03-13%20오후%206.37.58.png)

트랜잭션을 커밋하는 시점에 쓰기 지연 SQL 저장소에 있던 쿼리가 flush 되면서 DB에 날아간다.

```java
Member member1 = new Member(150L, "A");
Member member2 = new Member(160L, "B");

// 쓰기 지연 SQL 저장소에 저장된다.
entityManager.persist(member1);
entityManager.persist(member2);
System.out.println("------------");

// 실제 쿼리가 날아간다.
tx.commit();
```

```text
------------
Hibernate: 
    /* insert hellojpa.Member
        */ insert 
        into
            Member
            (name, id) 
        values
            (?, ?)
Hibernate: 
    /* insert hellojpa.Member
        */ insert 
        into
            Member
            (name, id) 
        values
            (?, ?)
```

쿼리가 구분선 아래에서 날아가는 걸 확인할 수 있다. 이렇게 하면 데이터베이스에 한 번만 전송할 수 있다.

```xml
<property name="hibernate.jdbc.batch_size" value="10"/>
```

value 값 만큼의 쿼리를 모아서 DB에 날릴 수 있는 옵션도 있다.

이렇게 버퍼링처럼 여러 개를 한 번에 모아서 할 수 있어서 옵션 하나로 성능 최적화를 할 수 있다.

### 변경 감지(더티 체킹)

![](../../.gitbook/assets/kimyounghan-orm-jpa/03/스크린샷%202021-03-13%20오후%206.54.34.png)

JPA는 자바 컬렉션에 넣은 것처럼 값을 다루는 게 목적이다. 컬렉션에서 꺼낸 값을 변경했다고 다시 컬렉션에 집어넣지 않는다. 마찬가지로 데이터 변경 후에 `persist()`는 쓰지 않는다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/03/스크린샷%202021-03-13%20오후%206.54.23.png)

커밋하는 시점에 내부적으로 `flush()`되면서 엔티티와 스냅샷을 비교한다. 

1차 캐시에는 키(id)와 값(entity) 외에도 스냅샷이 있다. 스냅샷은 최초로 영속성 컨텍스트에 들어온 상태를 본 떠둔 것이다. 커밋이 되면 스냅샷과 일일이 비교해서 바뀐 게 있으면 쿼리를 만들어서 쓰기 지연 SQL 저장소에 저장해서 DB에 반영한다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/03/스크린샷%202021-03-13%20오후%207.02.39.png)

삭제는 `remove()`만 해주면 된다. 메커니즘은 앞과 같이 쓰기 지연 저장소에 쿼리를 만들어뒀다가 커밋 시점에 실행된다.

그래도 수정할 때 `persist()`를 호출해야 하지 않냐고 하는 사람도 있다. 안 하는 것이 답이다. 어차피 업데이트 쿼리는 날아가기 때문에 괜한 로직상의 오류가 발생하지 않도록 주의하자.