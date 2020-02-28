# 스프링 IoC
---
description: https://martinfowler.com/articles/injection.html
---

### 일반적인 (의존성에 대한) 제어권

> 내가 사용할 의존성은 내가 만든다.

### IoC

> 내가 사용할 의존성은 누군가 알아서 해주겠지.

내가 사용할 의존성의 타입(또는 인터페이스)만 맞으면 어떤 것이든 상관없다. 그래야 내 코드 테스트 하기도 편하지.

의존성 주입도 IoC의 일종이다. 의존성을 관리하는 것 자체를 다른 곳에 위임했기 때문이다.

```java
class OwnerController {
   private OwnerRepository repo;

   // 사용하고 싶은 것을 파라미터로 받아온다.
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

의존성 주입과 IoC는 스프링만의 개념은 아니다. 스프링을 사용하면 스프링이 제공하는 IoC 기능을 사용할 수 있다.

## 스프링 IoC 컨테이너

