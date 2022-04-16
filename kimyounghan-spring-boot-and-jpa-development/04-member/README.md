# 회원 도메인 개발

## 회원 리포지토리 개발

```java
// 컴포넌트 스캔에 의해 자동으로 빈으로 관리된다.
@Repository
public class MemberRepository {

    // 스프링이 EntityManager를 주입해준다.
    @PersistenceContext
    private EntityManager em;

    public void save(Member member) {
        em.persist(member);
    }

    public Member findOne(Long id) {
        return em.find(Member.class, id);
    }

    public List<Member> findAll() {
        // 리스트는 jpql을 사용해야 한다.
        // sql은 테이블을 대상으로, jpql은 객체를 대상으로 쿼리한다는 점이 다르다.
        return em.createQuery("select m from Member m", Member.class)
                .getResultList();
    }

    public List<Member> findByName(String name) {
        return em.createQuery("select m from Member m where m.name = :name", Member.class)
                .setParameter("name", name) // :name 파라미터 바인딩
                .getResultList();
    }
}
```

### @Repository

- 스프링 빈으로 등록된다.
- JPA 예외를 스프링 기반 예외로 변환한다.

### @PersistenceContext

- EntityManager를 주입한다.
- 직접 긴 코드를 쓸 필요가 없어졌다.

### @PersistenceUnit

```java

@Repository
public class MemberRepository {

    // EntityManagerFactory를 직접 주입받고 싶다면 아래를 사용한다.
    @PersistenceUnit
    private EntityManagerFactory emf;
}
```

- EntityManagerFactory를 주입한다.

## 회원 서비스 개발

```java
// 컴포넌트 스캔에 의해 자동으로 빈으로 등록된다.
@Service
// JPA의 모든 데이터 변경과 로직은 트랜잭션 내에서 실행되어야 한다.
@Transactional(readOnly = true)
public class MemberService {

    @Autowired
    private MemberRepository memberRepository;

    @Transactional
    public Long join(Member member) {
        // 중복 회원 검증
        validateDuplicateMember(member);
        memberRepository.save(member);

        return member.getId();
    }

    private void validateDuplicateMember(Member member) {
        // WAS가 동시에 여러 개 떠서 동시에 validate를 시도하면 문제가 생긴다.
        // 실무에서는 이런 멀티 스레드 문제를 해결해줘야 한다.
        List<Member> findMembers = memberRepository.findByName(member.getName());

        if (!findMembers.isEmpty()) {
            throw new IllegalStateException("이미 존재하는 회원입니다.");
        }
    }

    public List<Member> findMembers() {
        return memberRepository.findAll();
    }


    public Member findOne(Long memberId) {
        return memberRepository.findOne(memberId);
    }
}

```

### @Transactional

- javax와 springframework 두 가지 종류가 있다.
    - 이미 스프링에 의존이 많이 되어있기 때문에 javax보다는 springframework를 import하는 게 좋다.
    - 그래야 쓸 수 있는 옵션도 더 다양하다.
- `readOnly` = true
    - 더티 체킹을 안하거나 DB 옵션에 따라 읽기 전용 모드로 읽는 등 성능상 이점이 있다.
    - 기본을 true로 두고 변경이 필요한 곳에만 false로 달아두면 된다.

조회가 아니라면 true일 때 데이터 변경이 안되므로 주의하자. 커맨드성이 강해서 조회가 거의 없다면 기본 값으로 두는 게 더 좋다.

### @Autowired

- `@Autowired`로 필드 주입을 하면 엑세스할 방법이 없어서 다른 의존성으로 바꿔치기 할 수가 없다.

```java

@Service
@Transactional(readOnly = true)
public class MemberService {

    private MemberRepository memberRepository;

    public void setMemberRepository(MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
    }

}
```

- setter 주입을 쓰면 직접 mock 등을 주입해줄 수 있는 장점이 있다.
- 애플리케이션 동작 시점에 누군가가 setter에 접근해서 바꿀 수 있는 위험이 있다.

```java

@Service
@Transactional(readOnly = true)
public class MemberService {

    private final MemberRepository memberRepository;

    public MemberService(MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
    }
}
```

- 생성자 주입을 하면 생성된 이후에는 바꿀 수 없어서 좋다.
- 객체 생성 시 의존성을 넣어주도록 컴파일 타임에 강제한다.
    - 주입해야 한다는 사실을 안 놓치고 안전하게 개발할 수 있다.
- 스프링 최신 버전에서는 생성자가 하나만 있는 경우 `@Autowired`가 없어도 알아서 생성해준다.
- 이때 `private final`로 선언해주는 것이 좋다.
    - 값 세팅을 안하면 컴파일 시점에 체크해주기 때문이다.

```java

@Service
@Transactional(readOnly = true)
@AllArgsConstructor
public class MemberService {

    private final MemberRepository memberRepository;
}
```

- 롬복 `@AllArgsConstructor`를 쓰면 생성자를 대체할 수 있다.

```java

@Service
@Transactional(readOnly = true)
@RequiredArgsConstructor
public class MemberService {

    private final MemberRepository memberRepository;
}
```

- `@RequiredArgsConstructor`는 final인 필드만 생성자를 만들어준다.
- `@AllArgsConstructor`보다 추천한다.

```java

@Repository
@RequiredArgsConstructor
public class MemberRepository {

    private final EntityManager em;
}
```

- 스프링 데이터 JPA 덕분에 EntityManager도 생성자 주입으로 받을 수 있다.

## 회원 기능 테스트

### 회원 가입

```java

@RunWith(SpringRunner.class)
@SpringBootTest
@Transactional
public class MemberServiceTest {

    @Autowired
    MemberService memberService;

    @Autowired
    MemberRepository memberRepository;

    @Test
    public void 회원가입() {
        // given
        Member member = new Member();
        member.setName("kim");

        // when
        Long savedId = memberService.join(member);

        // then
        // 같은 트랜잭션에 묶었으므로 동일한 영속성 컨텍스트에
        // 같은 PK를 가질 경우 똑같은 데이터로 취급한다.
        assertEquals(member, memberRepository.findOne(savedId));
    }
}
```

예제에서는 이해를 돕기 위해 통합적으로 테스트를 할 것이다.

- `@RunWith(SpringRunner.class)`
    - 스프링과 테스트를 통합한다.
- `@SpringBootTest`
    - 스프링 부트를 띄워서 테스트한다.
    - 이게 없으면 `@Autowired`는 다 실패한다.
- `@Transactional`
    - 반복 가능한 테스트를 지원한다.
        - 즉, 각 테스트 실행마다 테스트가 끝나면 트랜잭션을 롤백한다.
        - 자동 롤백은 테스트 케이스에서만 적용된다.

![](../../.gitbook/assets/kimyounghan-spring-boot-and-jpa-development/04/screenshot%202021-05-19%20오후%208.17.32.png)

select만 나가고 실제 회원 가입을 하면서 생겨야 할 insert 쿼리는 나가지 않았다.

```java

@Repository
@RequiredArgsConstructor
public class MemberRepository {

    public void save(Member member) {
        // insert 쿼리가 나가지 않은 이유
        em.persist(member);
    }
}
```

- DB 전략마다 다르지만 기본적으로 persist만 하면 쿼리가 나가지 않는다.
- 트랜잭션이 commit될 때 flush 되면서 쿼리가 나가는 것이기 때문이다.

```java

@RunWith(SpringRunner.class)
@SpringBootTest
// 기본적으로 롤백이 동작한다.
@Transactional
public class MemberServiceTest {

    @Test
    @Rollback(value = false)
    public void 회원가입() {
    ...
    }
}
```

- @Transactional이 자동으로 롤백을 하니까 당연히 insert 쿼리를 보내지 않는 것이다.
    - 정확하게는 영속성 컨텍스트가 flush를 하지 않는다.
- `@Rollback`에 false 옵션을 주어야 커밋이 실행된다.

![](../../.gitbook/assets/kimyounghan-spring-boot-and-jpa-development/04/screenshot%202021-05-19%20오후%208.22.32.png)

정상적으로 insert 쿼리가 나간 것을 볼 수 있다.

### 중복 회원 예외

```java

@RunWith(SpringRunner.class)
@SpringBootTest
@Transactional
public class MemberServiceTest {

    @Test(expected = IllegalStateException.class)
    public void 중복_회원_예외() {
        // given
        Member member1 = new Member();
        member1.setName("kim");

        Member member2 = new Member();
        member2.setName("kim");

        // when
        memberService.join(member1);
        // 똑같은 이름을 넣었으니 여기서 예외를 던져야 한다.
        memberService.join(member2);

        // then
        // 여기까지 오지 않고 위에서 예외가 이미 발생해야 할 때 쓴다.
        fail("예외가 발생해야 한다.");
    }
}
```

- `expected`에 터질 예정인 예외를 써준다.

## 테스트를 위한 설정

- 테스트는 완전히 격리된 환경에서 실행하는 것이 좋다. 
- 끝나면 데이터를 초기화하기 위해 메모리 DB를 사용한다.

![](../../.gitbook/assets/kimyounghan-spring-boot-and-jpa-development/04/screenshot%202021-05-19%20오후%208.38.47.png)

`test`에 `resources`를 따로 만들어두면 테스트 시에 `main`의 `resources`보다 우선권을 가진다.

![](../../.gitbook/assets/kimyounghan-spring-boot-and-jpa-development/04/screenshot%202021-05-19%20오후%208.41.11.png)

{% tabs %} {% tab title="test/resources/application.yaml" %}

```yaml
spring:
  datasource:
    url: jdbc:h2:mem:test # 인 메모리 방식으로 바꿔준다.
    username: sa
    password:
    driver-class-name: org.h2.Driver
  jpa:
    hibernate:
      ddl-auto: create
    properties:
      hibernate:
        format_sql: true
logging.level:
  org.hibernate.SQL: debug
```

{% endtab %} {% endtabs %}

하이버네이트는 jvm 위에 띄워 인 메모리 방식으로도 사용할 수 있다.

{% tabs %} {% tab title="test/resources/application.yaml" %}

```yaml
spring:
logging.level:
  org.hibernate.SQL: debug
```

{% endtab %} {% endtabs %}

심지어 이렇게 간단하게만 설정해도 돌아간다. 스프링 부트는 따로 설정이 없다면 자동으로 메모리 모드로 돌려주기 때문이다.

```yaml
spring:
  jpa:
    hibernate:
      ddl-auto: create  
```

스프링은 기본적으로 `create-drop`으로 돌아간다.

- create
    - 먼저 drop한 후 애플리케이션을 실행한다.
- create-drop
    - create와 똑같이 동작한 뒤에 종료 시점에 다시 drop 시킨다.
    - 인 메모리는 어차피 WAS가 내려가면 다 사라지기 때문에 중요하진 않다.