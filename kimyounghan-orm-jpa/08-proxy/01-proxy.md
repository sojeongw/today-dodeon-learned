# 프록시

![](../../.gitbook/assets/kimyounghan-orm-jpa/08/스크린샷%202020-07-08%20오후%203.09.37.png)

이러한 구조에서 Member를 조회할 때 Team도 함께 조회해야 할까?

```java
public class App {
    public void printUserAndTeam(String memberId) {
    Member member = em.find(Member.class, memberId);
    Team team = member.getTeam();
    System.out.println("회원 이름: " + member.getUsername());
    System.out.println("소속팀: " + team.getName());
    }
}
```

```java
public class App {
    public void printUser(String memberId) {
    Member member = em.find(Member.class, memberId);
    Team team = member.getTeam();
    System.out.println("회원 이름: " + member.getUsername());
    }
}
```

두 번째 코드처럼 team을 굳이 조회할 필요가 없다면 가져올 필요도 없다.

## 프록시 기초

이 문제를 정확히 해결하려면 프록시를 이해해야 한다.

### em.find()

데이터베이스를 통해서 실제 엔티티 객체를 조회한다.

```java
public class App {
    public class App {
        public void printUserAndTeam(String memberId) {
        Member member = em.find(Member.class, memberId);
        }
    }
}
```

위의 코드를 실행하면,

![](../../.gitbook/assets/kimyounghan-orm-jpa/08/스크린샷%202020-07-08%20오후%203.42.46.png)

`find()`만 했을 때도 `select` 쿼리가 실행되었다.

### em.getReference()

데이터베이스 조회를 미루는 가짜(프록시) 엔티티 객체를 조회한다. DB에 쿼리가 안 날아가는데 객체가 조회되는 것이다.

```java
public class App {
    public void printUserAndTeam(String memberId) {
    Member member = em.getReference(Member.class, memberId);
    }
}
```

기존의 `em.find()` 대신에 `em.getReference()`로 조회하면,

![](../../.gitbook/assets/kimyounghan-orm-jpa/08/스크린샷%202020-07-08%20오후%203.32.27.png)

`select` 쿼리가 나가지 않는 걸 확인할 수 있다.

```java
public class App {
    public void printUserAndTeam(String memberId) {
    Member member = em.getReference(Member.class, memberId);
    // 데이터 조회
    System.out.println("회원 이름: " + member.getUsername());
    }
}
```

![](../../.gitbook/assets/kimyounghan-orm-jpa/08/스크린샷%202020-07-08%20오후%203.40.40.png)

하지만 `member`의 데이터를 실제 호출하는 순간엔 `select` 쿼리가 출력된다.

```java
public class App {
    public void printUserAndTeam(String memberId) {
    Member member = em.getReference(Member.class, memberId);
    // 클래스 정보 조회
    System.out.println("회원 이름: " + member.getClass());
    }
}
```

![](../../.gitbook/assets/kimyounghan-orm-jpa/08/스크린샷%202020-07-08%20오후%204.11.33.png)

클래스를 출력해보면 `Proxy`라는 글자가 보인다. 하이버네이트가 강제로 만든 가짜 클래스라는 뜻이다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/08/스크린샷%202020-07-08%20오후%203.22.25.png)

프록시는 껍데기는 같지만 안은 텅텅 빈 객체다.

## 프록시 특징

![](../../.gitbook/assets/kimyounghan-orm-jpa/08/스크린샷%202020-07-08%20오후%204.21.18.png)

- 실제 클래스를 상속 받아서 만들어지기 때문에 실제 클래스와 겉모습이 같다.
    - 하이버네이트가 내부적으로 라이브러리를 사용해 상속한다.
- 이론상, 사용하는 입장에서는 진짜 객체인지 프록시 객체인지 구분하지 않고 사용한다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/08/스크린샷%202020-07-08%20오후%204.28.11.png)

- 프록시 객체는 실제 객체의 참조(target)을 보관한다.
- 프록시 객체에 있는 메서드를 호출하면 프록시 객체는 실제 객체의 메서드를 호출한다.

만약 `getId()`를 호출하면 프록시는 `target`에 있는 `getId()`를 대신 호출한다.

하지만 맨 처음에는 DB 조회가 되지 않은 상태이므로 `target`이 비어있을 것이다. 이때는 어떻게 될까?

### 프록시 객체의 초기화

```java
public class App {
    public void getMemberName() {
        // 실제 객체가 아닌 프록시 객체를 가져온다.
        Member member = em.getReference(Member.class, "id");
        // 처음에 target이 없으면 아래의 그림과 같은 과정을 거친다.
        member.getName();
    }
}
```

![](../../.gitbook/assets/kimyounghan-orm-jpa/08/스크린샷%202020-07-08%20오후%204.35.16.png)

1. 데이터를 요청했는데 `Member`의 `target`이 없으면 JPA가 진짜 `Member` 객체를 가져오라고 영속성 컨텍스트에 요청한다.
2. 영속성 컨텍스트는 DB를 조회해서 실제 Entity를 보내준다.
3. `target`과 진짜 객체인 Entity를 연결시켜준다.
4. `target`의 진짜 `getName()`을 통해서 값을 반환한다.

이제 target에 값이 있기 때문에 다시 DB를 조회하지 않아도 된다.

```java
public class App {
    public void printUserAndTeam(String memberId) {
    Member member = em.getReference(Member.class, memberId);
    // 실제 레퍼런스 조회
    System.out.println("회원 이름: " + member.getUsername());
    }
}
```

결국 `userName`을 실제 가져다 쓰는 시점에 내부적으로 영속성 컨텍스트로 `Member`를 요청해서 실제 레퍼런스를 가지게 된다.

```java
public class App {
    public void printUserAndTeam(String memberId) {
    Member member = em.getReference(Member.class, memberId);
    // 실제 레퍼런스 조회
    System.out.println("회원 이름: " + member.getUsername());
    // 다시 하면 프록시에서 조회
    System.out.println("회원 이름: " + member.getUsername());
    }
}
```

그리고 같은 내용을 다시 실행하면,

![](../../.gitbook/assets/kimyounghan-orm-jpa/08/스크린샷%202020-07-08%20오후%204.56.02.png)

프록시에서 가져다 쓰기 때문에 첫번째 `username`과는 달리 쿼리를 치지 않는다.

## 주의 사항

- 프록시 객체는 처음 사용할 때 한 번만 초기화 된다.
    - 한 번 초기화하면 그 내용을 그대로 사용한다.
    
```java
public class App {
    public void printUserAndTeam(String memberId) {
    Member member = em.getReference(Member.class, memberId);

    System.out.println("before: " + member.getClass());
    System.out.println("회원 이름: " + member.getUsername());
    System.out.println("after: " + member.getClass());  // before와 같은 값 출력
    }
}
```
    
- 프록시 객체를 초기화 할 때, 프록시 객체가 실제 엔티티로 바뀌는 것은 아니다.
    - 그래서 프록시를 통해 데이터를 가져온 뒤에도 `getClass()`의 값은 동일하다.
    - 초기화 되면 프록시 객체를 통해서 실제 엔티티에 접근할 수 있다.
    
```java
public class App {
    public void printUserAndTeam(String memberId) {
        Member member1 = new Member();
        member1.setUsername("member1");
        em.persist(member1);

        Member member2 = new Member();
        member2.setUsername("member2");
        em.persist(member2);

        em.flush();
        em.clear();

        Member m1 = em.find(Member.class, member1.getId());
        Member m2 = em.find(Member.class, member2.getId());
       
        // find()로 가져왔고 타입을 정확하게 비교하는 것이므로 true를 출력한다.
        System.out.println("m1 == m2: " + (m1.getClass() == m2.getClass()));

        Member m3 = em.getReference(Member.class, member1.getId());
        Member m4 = em.find(Member.class, member2.getId());
       
        // m3는 getReference()이므로 false를 출력한다.
        System.out.println("m1 == m2: " + (m1.getClass() == m2.getClass()));

        tx.commit();
    }
}
```

```java
public class App {
    // 실제 로직 상에서는 이렇게 프록시인지 아닌지 알기가 어려우므로 instance of를 습관적으로 사용해야 한다.
    private static void logic(Member m1, Member m2) {
        // true
        System.out.println("m1 == Member: " + (m1 instanceof Member));
        System.out.println("m2 == Member: " + (m2 instanceof Member));
    }
}
```
    
- 프록시 객체는 원본 엔티티를 상속받는다. 즉, 프록시인 멤버와 아닌 멤버가 타입이 맞지 않는다.
    - 따라서 타입 체크 시 주의해야 한다.
    - `==`로 비교하면 실패하고, `instance of`를 사용해야 한다.
    - 프록시를 쓸지 안 쓸지 모르므로 JPA에서는 웬만하면 `instance of`를 사용하는 것이 좋다.
    
```java
public class App {
    public void printUserAndTeam(String memberId) {
        Member member1 = new Member();
        member1.setUsername("member1");
        em.persist(member1);

        em.flush();
        em.clear();

        Member m1 = em.find(Member.class, member1.getId());
        // find이므로 'class app.Member'라고 정확한 해당 객체가 출력된다.
        System.out.println("m1 = " + m1.getClass());

        Member m2 = em.getReference(Member.class, member1.getId());
        // 레퍼런스도 프록시가 아니라 'Member'로 출력된다.
        System.out.println("m2 = " + m2.getClass());

        tx.commit();
    }
}
```
    
- 영속성 컨텍스트가 이미 있는 엔티티를 찾게 되면 `em.getReference()`를 호출해도 실제 엔티티를 반환한다.

JPA에서는 같은 인스턴스인지 비교할 때 같은 영속성 컨텍스트 안에 있으면 항상 같다고 나온다. 이미 멤버를 영속성 컨텍스트 1차 캐시에 넣어뒀기 때문에 굳이 프록시로 가져올 필요가 없어서 성능 최적화에 대한 이점이 없다.

더 중요한 이유가 있다. JPA에서는 원본과 레퍼런스를 `==` 비교 시 항상 `true`가 나온다. 한 트랜잭션 안에서는 값이 같다고 보장해주기 때문에, 하나의 영속성 컨텍스트 안에 있고 PK가 같으면 항상 `true`를 반환한다.

```java
public class App {
    public void printUserAndTeam(String memberId) {
        Member member1 = new Member();
        member1.setUsername("member1");
        em.persist(member1);

        em.flush();
        em.clear();

        Member m1 = em.getReference(Member.class, member1.getId());
        System.out.println("m1 = " + m1.getClass());

        Member reference = em.getReference(Member.class, member1.getId());
        System.out.println("reference = " + reference.getClass());
   
        System.out.println("a == a: " + (m1 == reference));

        tx.commit();
    }
}
```

![](../../.gitbook/assets/kimyounghan-orm-jpa/08/스크린샷%202020-07-09%20오전%201.08.28.png)

둘 다 레퍼런스로 받을 때도 같은 프록시로 출력되는 것을 확인할 수 있다. 하나의 영속성 컨텍스트에 있을 때 같다는 점을 보장해줘야 하기 때문이다.

```java
public class App {
    public void printUserAndTeam(String memberId) {
        Member member1 = new Member();
        member1.setUsername("member1");
        em.persist(member1);

        em.flush();
        em.clear();

        Member refMember = em.getReference(Member.class, member1.getId());
        // 프록시값 출력
        System.out.println("refMember = " + refMember.getClass());

        Member findMember = em.find(Member.class, member1.getId());
        // 당연히 Member 타입이 출력되어야 하는 것 아닌가?
        // 같음을 보장해줘야 하기 때문에 프록시로 나온다.
        System.out.println("findMember = " + findMember.getClass());
   
        // JPA에서는 무조건 이게 참이 되도록 맞춘다!
        System.out.println("a == a: " + (refMember == findMember));

        tx.commit();
    }
}
```

프록시가 초기화 된 상태에서 `find()`를 하면 어떻게 될까? JPA는 기본적으로 `refMember == findMember`의 값이 true임을 보장해야 한다. 하지만 코드를 보면 `refMember`는 프록시, `findMember`는 `Member`가 나올 것처럼 보인다. 정말 그럴까?

![](../../.gitbook/assets/kimyounghan-orm-jpa/08/스크린샷%202020-07-09%20오전%201.15.34.png)

![](../../.gitbook/assets/kimyounghan-orm-jpa/08/스크린샷%202020-07-09%20오전%201.15.44.png)

`find()`를 했기 때문에 실제 DB를 조회하면서 select 쿼리는 찍힌다. 하지만 프록시를 한 번 조회한 뒤에는 `find()`를 한 객체에도 프록시로 반환을 하도록 되어있다. 그래야 JPA의 룰을 보장할 수 있기 때문이다.

어쨌든 제일 중요한 건 프록시든 아니든 개발에 문제가 없도록 짜는 것이다. `instance of`를 기억하자.

- 영속성 컨텍스트의 도움을 받을 수 없는 준영속 상태이면, 프록시를 초기화할 때 문제가 발생한다.
    - 하이버테이트는 `org.hibernate.LazyInitializationException` 예외를 터뜨린다.

```java
public class App {
    public void printUserAndTeam(String memberId) {
        Member member1 = new Member();
        member1.setUsername("member1");
        em.persist(member1);

        em.flush();
        em.clear();

        // 프록시 생성
        Member refMember = em.getReference(Member.class, member1.getId());
        System.out.println("refMember = " + refMember.getClass());

        // 실제 DB를 조회하면서 쿼리를 날리고 프록시 초기화
        refMember.getUsername();
        System.out.println("refMember = " + refMember.getClass());

        tx.commit();
    }
}
```

이때는 앞뒤 결과가 같은 프록시를 출력하지만,

```java
public class App {
    public void printUserAndTeam(String memberId) {
        Member member1 = new Member();
        member1.setUsername("member1");
        em.persist(member1);

        em.flush();
        em.clear();

        // 프록시 생성
        Member refMember = em.getReference(Member.class, member1.getId());
        System.out.println("refMember = " + refMember.getClass());

        // 영속성 컨텍스트를 준영속으로 만든다.
//        em.detach(refMember);
//        em.clear();
        em.close();

        // 영속성 컨텍스트로 관리하지 않게 되면서 exception이 떨어진다.
        refMember.getUsername();
        System.out.println("refMember = " + refMember.getClass());

        tx.commit();
    }
}
```

![](../../.gitbook/assets/kimyounghan-orm-jpa/08/스크린샷%202020-07-09%20오전%201.26.22.png)

`detach()`나 `clear()`, 혹은 `close()`로 준영속 상태가 되면 `org.hibernate.LazyInitializationException`을 날린다. 앞에서 살펴봤던 것 처럼 프록시를 초기화 할 때 영속성 컨텍스트를 통하기 때문이다.

트랜잭션이 끝났는데 조회를 하려고 할 때 많이 만나는 에러이므로 기억해두도록 하자.

## 프록시 유틸리티 메서드

#### 프록시 인스턴스의 초기화 여부 확인

해당 프록시가 초기화 됐는지 확인하는 메서드

`PersistenceUnitUtil.isLoaded(Object entity)`

```java
public class App {
    public static void main(String[] args) {
        EntityManagerFactory emf = Persistence.createEntityManagerFactory();
        ...
        // 앞에서 초기화 했다면 true, 아니라면 false
        System.out.println("isLoaded: " + emf.getPersistenceUnitUtil().isLoaded(refMember));
    }
}
```

#### 프록시 클래스 확인

`entity.getClass().getName()`

#### 프록시 강제 초기화

`org.hibernate.Hibernate.initialize(entity)`

```java
public class App {
    public void printUserAndTeam(String memberId) {
        Member member1 = new Member();
        member1.setUsername("member1");
        em.persist(member1);

        em.flush();
        em.clear();

        Member refMember = em.getReference(Member.class, member1.getId());
        System.out.println("refMember = " + refMember.getClass());

        // 이런 방식으로 강제 호출하는 것 보다는
        refMember.getUsername();
        // 이 방식을 더 권유한다.
        Hibernate.initialize(refMember);

        tx.commit();
    }
}
```

참고로 이런 초기화 방식은 Hibernate에서 제공하는 것이며, JPA 표준은 강제 초기화가 없다. 따라서 `member.getName()`처럼 강제 호출 해야 한다.