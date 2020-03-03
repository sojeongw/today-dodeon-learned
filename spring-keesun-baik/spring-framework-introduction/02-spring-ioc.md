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

특정 생성자나 애너테이션 등을 통해 필요한 의존성을 주입해주는 것. 

## 스프링 IoC 컨테이너

빈을 만들고, 서로의 의존성을 엮어주고, 그렇게 만들어진 빈을 제공한다. 프로젝트에 있는 모든 클래스가 빈으로 등록되는 것은 아니다. 

의존성 주입은 빈끼리만 가능하다. 즉, IoC 컨테이너 안에 들어있는 객체끼리만 의존성 주입이 가능하다.

### BeanFactory

사실상 IoC 컨테이너

### ApplicationContext

BeanFactory를 상속하면서 그밖의 다양한 일들도 함께 수행한다.

```java
public class BeanTest {
    @Test
    public void getBean() {
        // 애플리케이션 컨텍스트에 있는 모든 빈의 이름을 가져올 수 있다.
        applicationContext.getBeanDefinitionNames();
        // 존재하는 빈 이름을 넣고 체크해보면 null이 아닌 것을 알 수 있다.
        OwnerController bean = applicationContext.getBean(OwnerController.class);
        assertThat(bean).isNotNull();
    }   
} 
```

하지만 이렇게 직접 `applicationContext`를 사용할 일은 없다. `application context`가 가지고 있는 빈을 알아서 주입해주기 때문이다.

### 빈 스코프

```java
@Controller
class OnwerController{
    private final OwnerRepository owners;

    @GetMapping("/bean")
    @ResponseBody
    public String bean() {
        // 둘은 같은 해시 값을 출력한다.
        return applicationContext.getBean(OwnerRepository.class) + "\n" + this.owners;
     }
}
```

스프링은 빈을 매번 새로 만들지 않고 재사용 한다. 멀티 스레드 상황에서 싱글턴을 사용하는 건 번거롭고 조심스럽다. 하지만 IoC를 이용하면 특별한 코드를 넣지 않아도 IoC에 등록된 빈으로 편하게 싱글턴 스코프를 구현할 수 있다.