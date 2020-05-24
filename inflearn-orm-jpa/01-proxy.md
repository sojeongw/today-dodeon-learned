# 프록시

![](../.gitbook/assets/inflearn-orm-jpa/01/스크린샷%202020-07-08%20오후%203.09.37.png)

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

![](../.gitbook/assets/inflearn-orm-jpa/01/스크린샷%202020-07-08%20오후%203.42.46.png)

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

![](../.gitbook/assets/inflearn-orm-jpa/01/스크린샷%202020-07-08%20오후%203.32.27.png)

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

![](../.gitbook/assets/inflearn-orm-jpa/01/스크린샷%202020-07-08%20오후%203.40.40.png)

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

![](../.gitbook/assets/inflearn-orm-jpa/01/스크린샷%202020-07-08%20오후%204.11.33.png)

클래스를 출력해보면 `Proxy`라는 글자가 보인다. 하이버네이트가 강제로 만든 가짜 클래스라는 뜻이다.

![](../.gitbook/assets/inflearn-orm-jpa/01/스크린샷%202020-07-08%20오후%203.22.25.png)

프록시는 껍데기는 같지만 안은 텅텅 빈 객체다.

## 프록시 특징

![](../.gitbook/assets/inflearn-orm-jpa/01/스크린샷%202020-07-08%20오후%204.21.18.png)

- 실제 클래스를 상속 받아서 만들어지기 때문에 실제 클래스와 겉모습이 같음
    - 하이버네이트가 내부적으로 라이브러리를 사용해 상속함
- 이론상, 사용하는 입장에서는 진짜 객체인지 프록시 객체인지 구분하지 않고 사용함

![](../.gitbook/assets/inflearn-orm-jpa/01/스크린샷%202020-07-08%20오후%204.28.11.png)

- 프록시 객체는 실제 객체의 참조(target)을 보관함
- 프록시 객체를 호출하면 프록시 객체는 실제 객체의 메서드를 호출함

만약 `getId()`를 호출하면 프록시는 `target`에 있는 `getId()`를 대신 호출한다.

하지만 맨 처음에는 DB 조회가 되지 않은 상태이므로 `target`이 비어있을 것이다. 이때는 어떻게 될까?

## 프록시 객체의 초기화

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

![](../.gitbook/assets/inflearn-orm-jpa/01/스크린샷%202020-07-08%20오후%204.35.16.png)

1. 데이터를 요청했는데 `Member`의 `target`이 없으면 JPA가 진짜 `Member` 객체를 가져오라고 영속성 컨텍스트에 요청한다.
2. 영속성 컨텍스트는 DB를 조회해서 실제 Entity를 보내준다.
3. `target`과 진짜 객체인 Entity를 연결시켜준다.
4. `target`의 진짜 `getName()`을 통해서 값을 반환한다.