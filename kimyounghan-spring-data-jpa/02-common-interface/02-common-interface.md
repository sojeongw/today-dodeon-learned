# 공통 인터페이스 설정

```java

@Configuration
@EnableJpaRepositories(basePackages = "jpabook.jpashop.repository")
public class AppConfig {
}
```

- 설정 파일 위치를 지정해줘야 한다.
- 스프링 부트는 @SpringBootApplication 위치부터 자동으로 빈을 찾아가기 때문에 설정이 필요 없다.

![](../../.gitbook/assets/kimyounghan-spring-data-jpa/02/screenshot%202022-05-07%20오후%204.39.29.png)

```java
public interface MemberRepository extends JpaRepository<Member, Long> {
}
```

```text
memberRepository.getClass(): class com.sun.proxy.$ProxyXXX
```

- MemberRepository
    - `org.springframework.data.repository.Repository`를 구현한 클래스를 자동으로 스캔한다.
    - 스프링 데이터 JPA 관련 인터페이스를 발견하면 proxy에 구현체를 채워넣는다.
- @Repository의 역할
    - 컴포넌트 스캔
        - 생략해도 스프링 데이터 JPA가 자동으로 처리한다.
    - JPA 예외를 스프링 예외로 자동 변환

## 공통 인터페이스 분석

![](../../.gitbook/assets/kimyounghan-spring-data-jpa/02/screenshot%202022-05-07%20오후%204.53.09.png)

```java
public interface JpaRepository<T, ID>
        extends PagingAndSortingRepository<T, ID>, QueryByExampleExecutor<T> {
  ...
}
```

- JpaRepository
    - 공통 CRUD 제공
- 제네릭
    - T
        - 엔티티 타입
    - ID
        - 식별자 타입
    - S
        - 엔티티와 그 자식 타입

## 주요 메서드

- save(S)
    - 새로운 엔티티는 저장하고 이미 있는 엔티티는 병합한다.
- delete(T)
    - 엔티티 하나를 삭제한다.
    - 내부에서 EntityManager.remove()를 호출한다.
- findById(ID)
    - 엔티티 하나를 조회한다.
    - 내부에서 EntityManager.find()를 호출한다.
- getOne(ID)
    - 엔티티를 프록시로 조회한다.
        - 실제 값을 꺼낼 때 DB에 쿼리를 날려 가져온다.
    - 내부에서 EntityManager.getReference()를 호출한다.
- findAll()
    - 모든 엔티티를 조회한다.
    - 정렬이나 페이징 조건을 파라미터로 제공할 수 있다.