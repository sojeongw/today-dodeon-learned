# 영속성 관리

![](../../.gitbook/assets/kimyounghan-orm-jpa/03/스크린샷%202021-03-13%20오후%205.12.20.png)

- 요청이 올 때마다 EntityManagerFactory를 통해 EntityManager를 생성한다.
- EntityManager는 내부적으로 DB 커넥션을 통해 DB를 사용한다.

## 영속성 컨텍스트

- Entity를 영구 저장하는 환경

```
EntityManager.persist(entity);
```

이전에 위와 같은 코드로 Entity를 DB에 저장했었다.

정확히 말하자면 저장하는 게 아니라, 영속성 컨텍스트를 통해 Entity를 영속화 한다는 뜻이다. 더 정확히 말하자면 `persist()`는 Entity를 영속성 컨텍스트에 저장하는 것이다.

영속성 컨텍스트는 논리적인 개념이라 눈에 보이지 않으며, EntityManager를 통해 접근한다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/03/스크린샷%202021-03-13%20오후%205.16.34.png)

- EntityManager를 생성하면 그 안에 1:1로 영속성 컨텍스트가 생성된다.
- EntityManager 안에 영속성 컨텍스트라는 눈에 보이지 않는 공간이 생기는 것이다.

## Entity의 생명 주기

![](../../.gitbook/assets/kimyounghan-orm-jpa/03/스크린샷%202021-03-13%20오후%205.23.48.png)

- 비영속(new/transient)
    - 최초에 객체를 생성한 상태
    - 즉, 영속성 컨텍스트와 전혀 관계 없는 새로운 상태
- 영속(managed)
    - persist() 한 상태
    - 영속성 컨텍스트에 관리되는 상태
- 준영속(detached)
    - 영속성 컨텍스트에 저장되었다가 분리된 상태
- 삭제(removed)
    - 삭제된 상태

### 비영속

![](../../.gitbook/assets/kimyounghan-orm-jpa/03/스크린샷%202021-03-13%20오후%205.25.59.png)

- Member 객체를 생성만 하고 EntityManager에 아무것도 안 넣은 상태
- 즉, JPA와 전혀 관계없는 상태

### 영속

![](../../.gitbook/assets/kimyounghan-orm-jpa/03/스크린샷%202021-03-13%20오후%205.26.06.png)

- 객체 생성 후에 EntityManager를 가져와서 persist로 객체를 넣어준다.
- 그럼 EntityManager 안에 있는 영속성 컨텍스트에 member가 들어가면서 영속 상태가 된다.

```java
public class JpaMain {

    public static void main(String[] args) {
        ...

        // 여기까지는 아무 상태도 아닌 비영속 상태다.
        Member member = new Member();
        member.setId(1L);
        member.setName("HelloJPA");

        // 이때까지는 로그에 쿼리가 출력되지 않는다.
        System.out.println("=== before ===");

        // 여기서부터 영속 상태가 된다. 영속성 컨텍스트 안에서 객체가 관리된다.
        entityManager.persist(member);

        // 이 출력 뒤에 로그에 insert 쿼리가 찍힌다.
        System.out.println("=== after ===");

        // 실제 디비에 저장되는 건 커밋될 때다.
        // 이때 쿼리가 날아가는 것이다.
        tx.commit();
    
    ...
    }
}
```

영속 상태가 된다고 해서 DB에 쿼리가 날아가는 게 아니다. commit을 해야 반영된다.

### 준영속

![](../../.gitbook/assets/kimyounghan-orm-jpa/03/스크린샷%202021-03-13%20오후%205.26.14.png)

- 영속성 컨텍스트와 아무 관계가 아닌 상태

### 삭제

![](../../.gitbook/assets/kimyounghan-orm-jpa/03/스크린샷%202021-03-13%20오후%205.26.21.png)

- DB에서 데이터를 삭제하는 상태

## 영속성 컨텍스트의 이점

### 1차 캐시

- 사실상 영속성 컨텍스트라고 생각해도 된다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/03/스크린샷%202021-03-13%20오후%206.15.26.png)

- member를 만들고 set 했을 때는 단지 생성만 한 비영속 상태다.
- 영속화를 하면 PK가 키, Entity가 값이 되어 1차 캐시에 저장된다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/03/스크린샷%202021-03-13%20오후%206.15.32.png)

- 1차 캐시에 저장 해두면, 조회할 때 DB 대신 1차 캐시를 먼저 뒤져서 반환할 수 있다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/03/스크린샷%202021-03-13%20오후%206.15.37.png)

- member2처럼 DB엔 있지만 1차 캐시엔 없다면
    - DB에서 조회한 데이터를 1차 캐시에 저장한 뒤 반환한다.
- 이후에 다시 member2를 조회하면, 1차 캐시에 있는 member2가 반환된다.

EntityManager는 트랜잭션 단위로 만들고 트랜잭션이 종료되면 삭제된다. 이때 1차 캐시도 함께 날아간다. 많은 고객에게 요청이 와도 각자의 트랜잭션을 가지고 있기 때문에, 결국 아주 짧은 찰나의 순간에만 사용되어 성능상 이점은 크게 없다.

```java
public class JpaMain {

    public static void main(String[] args) {
        
        ...

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
    }
}
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

- find() 한 지점에는 select 쿼리가 나가지 않았다.
    - member가 1차 캐시에 먼저 저장됐기 때문이다.

```java
public class JpaMain {

    public static void main(String[] args) {
        
        ...

        Member findMember1 = entityManager.find(Member.class, 101L);
        Member findMember2 = entityManager.find(Member.class, 101L);

        tx.commit();
    }
}
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

- 1차 캐시 때문에 같은 데이터를 2번 조회하면 select 쿼리가 한 번만 나간다.

### 동일성 보장

![](../../.gitbook/assets/kimyounghan-orm-jpa/03/스크린샷%202021-03-13%20오후%206.31.01.png)

- 자바 컬렉션에서 꺼낸 데이터는 레퍼런스가 같은데, JPA도 이와 같은 동일성을 보장해준다.
- 반복 가능한 읽기 등급(REPEATABLE READ)의 트랜잭션 격리 수준을 제공한다.
    - 1차 캐시를 이용해 DB가 아닌 애플리케이션 차원에서 제공할 수 있게 되었다.

### 트랜잭션을 지원하는 쓰기 지연

![](../../.gitbook/assets/kimyounghan-orm-jpa/03/스크린샷%202021-03-13%20오후%206.33.58.png)

- persist() 한다고 바로 쿼리를 보내지 않는다.
- JPA가 메모리에 쿼리를 쌓고 있다가 트랜잭션을 커밋하는 순간 DB에 보낸다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/03/스크린샷%202021-03-13%20오후%206.35.49.png)

- 영속성 컨텍스트에는 쓰기 지연 SQL 저장소가 있다.
- memberA를 persist 하면 일단 1차 캐시에 들어간다.
    - 이와 동시에 JPA가 Entity를 분석해서 insert 쿼리를 생성한다.
- memberB를 persist 할 때도 1차 캐시에 일단 넣는다.
    - 이때도 insert 쿼리를 생성해 쓰기 지연 저장소에 쌓아둔다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/03/스크린샷%202021-03-13%20오후%206.37.58.png)

- 트랜잭션을 커밋하는 시점이 되어서야 쓰기 지연 SQL 저장소에 있던 쿼리가 flush 되면서 DB에 날아간다.

{% tabs %} {% tab title="Member.java" %}

```java

@Entity
public class Member {

    @Id
    private Long id;
    private String name;

    // JPA는 내부적으로 reflection을 이용해 동적으로
    // 객체를 생성해내기 때문에 기본 생성자가 필요하다. 
    // public일 필요는 없다.
    public Member() {

    }

    // 객체 생성을 쉽게 하기 위해 새로운 생성자를 만든다.
    public Member(Long id, String name) {
        this.id = id;
        this.name = name;
    }

  ...
}
```

{% endtab %} {% tab title="JpaMain.java" %}

```java
public class JpaMain {

    public static void main(String[] args) {
        
        ...

        Member member1 = new Member(150L, "A");
        Member member2 = new Member(160L, "B");

        // 쓰기 지연 SQL 저장소에 저장된다.
        entityManager.persist(member1);
        entityManager.persist(member2);
        System.out.println("------------");

        // 실제 쿼리가 날아간다.
        tx.commit();
    }
}
```

{% endtab %} {% endtabs %}

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

memberA와 B에 대한 두 쿼리가 구분선 아래 즉, 커밋 시점에 날아가는 걸 확인할 수 있다.

```xml

<property name="hibernate.jdbc.batch_size" value="10"/>
```

value 값 만큼의 쿼리를 모아서 DB에 날릴 수 있는 옵션도 있다.

만약 persist() 할 때마다 DB에 쿼리를 날리면 최적화 하기가 어렵다. 이렇게 버퍼링처럼 여러 개를 한 번에 모아서 보낸다면, 옵션 하나로 성능 최적화를 할 수 있다.

### 변경 감지(더티 체킹)

![](../../.gitbook/assets/kimyounghan-orm-jpa/03/스크린샷%202021-03-13%20오후%206.54.34.png)

- JPA는 자바 컬렉션에 넣은 것처럼 값을 다루는 게 목적이다.
    - 컬렉션에서 꺼낸 값을 변경했다고 다시 컬렉션에 집어넣지 않는다.
- 마찬가지로 JPA는 데이터 변경 후에 `persist()`를 쓰지 않아도 된다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/03/스크린샷%202021-03-13%20오후%206.54.23.png)

비밀은 영속성 컨텍스트에 있다.

- 1차 캐시에는 키(id)와 값(entity) 외에도 스냅샷이 있다.
    - 스냅샷은 최초로 영속성 컨텍스트에 들어온 상태를 본 떠둔 것이다.
- 커밋하는 시점에 내부적으로 `flush()` 되면서 Entity와 스냅샷을 비교한다.
    - 스냅샷과 일일이 비교해서 바뀐 게 있으면 update 쿼리를 만든다.
- 쿼리를 쓰기 지연 SQL 저장소에 저장한 뒤 DB에 반영한다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/03/스크린샷%202021-03-13%20오후%207.02.39.png)

- 삭제는 `remove()`만 해주면 된다.
- 다른 변경과 똑같이 쓰기 지연 저장소에 쿼리를 모았다가 커밋 시점에 실행한다.

그래도 수정할 때 `persist()`를 호출해야 하지 않냐고 하는 사람도 있다. 안 하는 것이 답이다.

```java
public class JpaMain {
    public static void main(String[] args) {
        ...

        if (member.getName().equals("younghan")) {
            em.update(member);
        }
    }
}
```

만약 이렇게 특정 조건일 때만 update() 하기 위해 명시적으로 사용해도, 어차피 commit을 하면 update 쿼리가 무조건 날아간다. 괜한 로직상의 오류가 발생하지 않게 그냥 생략하자.

## 플러시

- 영속성 컨텍스트의 변경 내용을 DB에 반영하는 것
    - 변경을 감지
    - 수정된 Entity를 쓰기 지연 SQL 저장소에 등록
    - 쿼리를 DB에 전송
- 커밋되면 플러시가 자동 발생한다.

### 영속성 컨텍스트를 플러시 하는 방법

- 직접 호출
    - em.flush()
- 자동 호출
    - 트랜잭션 커밋
    - JPQL 쿼리 실행

```java
public class JpaMain {

    public static void main(String[] args) {
        
        ...

        Member member = new Member(200L, "A");
        entityManager.persist(member);

        entityManager.flush();
        System.out.println("-----");

        tx.commit();
    }
}
```

```text
Hibernate: 
    /* insert hellojpa.Member
        */ insert 
        into
            Member
            (name, id) 
        values
            (?, ?)
-----
```

- 쿼리를 미리 반영하고 싶으면 커밋 전에 강제로 `flush()`를 호출할 수 있다.
- `flush()`를 해도 1차 캐시는 유지된다.
    - 1차 캐시와는 상관없이 쓰기 지연 SQL 저장소에 쌓인 쿼리나 변경 감지한 내용이 DB에 반영되는 것이다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/03/스크린샷%202021-03-13%20오후%207.17.05.png)

- JPQL을 중간에 실행했는데 flush 되지 않는다면
    - 아직 커밋하지 않은 persist 대상으로 JPQL 쿼리를 썼다면 이 데이터를 불러 올 수 없다.
- 그래서 JPQL 실행 시에는 자동으로 무조건 `flush()`를 날리는 것이다.

### 플러시 모드 옵션

- FlushModeType.AUTO
    - 기본값
    - 커밋이나 쿼리를 실행할 때 실행된다.
- FlushModeType.COMMIT
    - 커밋할 때만 실행된다.
    - 쿼리를 할 때는 실행되지 않는다.
        - 중간에 실행하는 JPQL 쿼리가 앞과는 전혀 다른 데이터를 써서 굳이 당장 플러시할 필요가 없을 때 사용하면 된다.

### 정리

- 영속성 컨텍스트의 변경 내용을 데이터베이스에 동기화하는 작업
    - 영속성 컨텍스트를 비우는 개념이 아니다.
- 트랜잭션이라는 개념이 있기 때문에 동작 가능한 매커니즘이다.
    - 트랜잭션 작업 단위로 동작하므로 커밋 직전에만 변경 내역을 DB에 날려주면 된다.

## 준영속 상태

- 영속
    - 영속성 컨텍스트에서 관리되는 상태
    - insert뿐만 아니라 조회할 때 1차 캐시에 없어서 DB에서 가져와 1차 캐시에 올리는 상태도 포함된다.
- 준영속
    - 영속 상태의 Entity가 영속성 컨텍스트에서 분리되는 상태
    - `detach()`를 실행하면 트랜잭션을 커밋해도 영향을 받지 않는다.
    - 영속성 컨텍스트가 더 이상 관리하지 않는 상태가 된다.
    - 즉, 영속성 컨텍스트가 제공하는 기능을 사용할 수 없다.

### 준영속 상태로 만드는 법

- em.detach(entity)
    - 특정 Entity만 준영속 상태로 전환한다.
- em.clear()
    - 영속성 컨텍스트를 통으로 지운다.
    - 즉, 완전히 초기화한다.
    - 1차 캐시 등이 다 사라졌기 때문에 같은 데이터를 조회해도 다시 DB에서 가져온다.
- em.close()
    - 영속성 컨텍스트를 종료한다.

```java
public class JpaMain {

    public static void main(String[] args) {
        
        ...

        Member member1 = entityManager.find(Member.class, 150L);
        member1.setName("AAAAA");

        entityManager.clear();

        // 영속성 컨텍스트를 지워서 1차 캐시에 없으므로 다시 올리기 위해 select 쿼리가 다시 실행된다.
        Member member2 = entityManager.find(Member.class, 150L);

        System.out.println("-----");

        tx.commit();
    }
}
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
Hibernate: 
    select
        member0_.id as id1_0_0_,
        member0_.name as name2_0_0_ 
    from
        Member member0_ 
    where
        member0_.id=?
-----
```