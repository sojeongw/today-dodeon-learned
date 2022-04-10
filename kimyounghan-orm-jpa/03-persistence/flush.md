# 플러시

- 영속성 컨텍스트의 변경 내용을 DB에 반영하는 것
    - 변경을 감지
    - 수정된 Entity를 쓰기 지연 SQL 저장소에 등록
    - 쿼리를 DB에 전송
- 커밋되면 플러시가 자동 발생한다.

## 영속성 컨텍스트를 플러시 하는 방법

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

- JPQL 실행 시에는 자동으로 무조건 `flush()`를 날린다.
    - 만약 JPQL을 중간에 실행했는데 flush 되지 않는다면?
        - 아직 커밋하지 않은 persist 대상에 JPQL 쿼리를 날릴 경우 해당 데이터를 불러 올 수 없다.

## 플러시 모드 옵션

- FlushModeType.AUTO
    - 기본값
    - 커밋이나 쿼리를 실행할 때 실행된다.
- FlushModeType.COMMIT
    - 커밋할 때만 실행된다.
    - 쿼리를 할 때는 실행되지 않는다.
        - 중간에 실행하는 JPQL 쿼리가 앞과는 전혀 다른 데이터를 써서 굳이 당장 플러시할 필요가 없을 때 사용하면 된다.

## 정리

- 영속성 컨텍스트의 변경 내용을 데이터베이스에 동기화하는 작업
    - 영속성 컨텍스트를 비우는 개념이 아니다.
- 트랜잭션이라는 개념이 있기 때문에 동작 가능한 매커니즘이다.
    - 트랜잭션 작업 단위로 동작하므로 커밋 직전에만 변경 내역을 DB에 날려주면 된다.