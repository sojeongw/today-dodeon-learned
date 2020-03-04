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

[Inversion of Control Containers and the Dependency Injection pattern](https://martinfowler.com/articles/injection.html)

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

[Interface BeanFactory](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/beans/factory/BeanFactory.html)

[Interface ApplicationContext](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/ApplicationContext.html)

## 빈

스프링 IoC 컨테이너가 관리하는 객체

```java
OwnerController ownerController = new OwnerController();
```

위 코드는 직접 new를 해서 만든 인스턴스이므로 빈이 아니다. 

```java
OwnerController ownerController = applicationContext.getBean(OwnerController.class);
```

위 객체는 애플리케이션 컨텍스트에 등록된 빈이다. 스프링이 얘기하는 빈에 해당한다.

그럼 어떻게 하면 객체를 빈으로 등록할 수 있을까?

## 빈 등록하기

### 1. Component Scanning

애너테이션으로 등록하는 방식. 애너테이션은 그 자체가 애너테이션을 처리하는 것은 아니다. 그냥 애너테이션일 뿐이다.

- @Component
    - @Repository
    - @Service
    - @Controller
    - @Configuration
    
애너테이션 프로세서 중에 스프링 IoC 컨테이너가 사용하는 여러가지 인터페이스가 있다. 이 인터페이스를 `라이프 사이클 콜백`이라고 부른다.

`라이프 사이클 콜백`에는 `@Component` 애너테이션이 붙어있는 모든 클래스를 찾아서 그 클래스의 인스턴스를 빈으로 등록하는 애너테이션 프로세서가 등록되어 있다.

```java
@SpringBootApplication(proxyBeanMethods = false)
public class PetClinicApplication {
	public static void main(String[] args) {
		SpringApplication.run(PetClinicApplication.class, args);
	}
}
```

`@SpringBootApplication`에는 `@ComponentScan`이 있다. `@ComponentScan`은 어디부터 어디까지 찾아보라고 알려준다. 그러면 `@ComponentScan`의 모든 하위 클래스를 훑어보면서 `@Repository`, `@Service`, `@Controller` 등등 다양한 애너테이션을 찾는다.

즉, 우리가 하나하나 빈으로 등록하지 않아도 스프링이 알아서 IoC 컨테이너가 만들어질 때 `@ComponentScan`에 따라 해당 클래스를 빈으로 등록해준다.

```java
public interface OwnerRepository extends Repository<Owner, Integer> {
    ...
}
```

`Repository` 는 조금 특이한 방식으로 등록된다. 스프링 JPA에 의해 등록되기 때문에 애너테이션이 없어도 된다. 

특정 인터페이스(Repository)를 상속하면, 해당 인터페이스를 상속받는 인터페이스(OwnerRepository)를 찾는다. 그리고 그 인터페이스의 구현체를 내부적으로 생성해서 빈으로 등록한다.

### 2. 직접 등록하기

XML이나 자바 설정 파일이 직접 하나하나 등록하는 방식. 

```java
@Configuration
public class SampleConfig {
    @Bean
    public SampleController sampleController() {
        // 리턴하는 이 객체 자체가 IoC 컨테이너의 빈으로 등록된다.
        return new SampleController();
    }
}

// 그럼 더 이상 이 클래스에 @Controller를 붙일 필요가 없다.
public class SampleController {
    ...
}
```

요즘 추세는 자바 설정 파일을 사용하는 것이다. `@Configuration` 애너테이션을 붙인 클래스에 빈으로 등록하고 싶은 내용을 추가하면 된다.

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class SampleControllerTest {
    @Autowired
    ApplicationContext applicationContext;

    @Test
    public void testDI() {
        SampleController bean = applicationContext.getBean(SampleController.class);
        assertThat(bean).isNotNull();
    }
}
```

따라서 위의 테스트는 성공적으로 실행된다. 직접 설정한 빈도 애플리케이션 컨텍스트에 등록되어 있는 것이다.

## 빈 사용하기

```java
class OwnerController {
    @Autowired
    private OwnerRepository owners;
}
```

생성자로 주입받는 대신 `@Autowired`를 사용하면 IoC 컨테이너에 들어있는 빈을 주입받아 사용할 수 있다.

```java
SampleController bean = applicationContext.getBean(SampleController.class);
```

물론 이렇게 직접 주입받을 수도 있지만 `@Autowired`를 자주 쓰게 될 것이다.