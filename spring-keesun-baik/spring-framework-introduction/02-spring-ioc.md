# 스프링 IoC
---
description: https://martinfowler.com/articles/injection.html
---

## 일반적인 (의존성에 대한) 제어권

> 내가 사용할 의존성은 내가 만든다.

```java
class OwnerController {
    private OwnerRepository repository = new OwnerRepository();
}
```

일반적으로는 이렇게 쓰고 싶은 의존성 `new OwnerRepository()`를 자기가 직접 만들어 사용한다. 제어권이 자기 자신인 `OwnerController`에 있어서 자기가 관리한다.

## IoC

> 내가 사용할 의존성은 누군가 알아서 해주겠지.

내가 사용할 의존성의 타입(또는 인터페이스)만 맞으면 어떤 것이든 상관없다. 그래야 내 코드 테스트 하기도 편하지.

의존성 주입도 IoC의 일종이다. 의존성을 관리하는 것 자체를 다른 곳에 위임했기 때문이다.

```java
class OwnerController {
   // OwnerController는 OwnerRepository를 사용은 하지만 만들진 않는다.
   // OwnerController 밖에서 누군가가 주입시켜준다.
   private OwnerRepository repo;

   // 즉, 생성자를 통해 그것을 파라미터로 받아온다.
   // 더 이상 의존성을 만드는 일은 OwnerController가 하는 게 아니다.
   public OwnerController(OwnerRepository repo) {
       this.repo = repo;
   } 

   // repo를 사용한다.
}

class OwnerControllerTest {
   @Test
   public void create() {
         OwnerRepository repo = new OwnerRepository();
         OwnerController controller = new OwnerController(repo);
   }
}
```

이제 의존성은 `OwnerController` 밖에서 만들어 주입(Dependency Injection) 해준다. DI는 일종의 Inversion of Control이다. 의존성을 관리하는 그 자체, 제어권이 역전되었기 때문이다. 자기 자신이 아니라 외부의 누군가가 제어하기 때문이다.

의존성 주입과 IoC는 스프링만의 개념은 아니다. 스프링을 사용하면 스프링이 제공하는 IoC 기능을 사용할 수 있다.

### 빈

스프링이 관리하는 객체

### 의존성 관리

특정 생성자나 애너테이션 등을 통해 필요한 의존성을 주입해주는 것

## 스프링 IoC 컨테이너

