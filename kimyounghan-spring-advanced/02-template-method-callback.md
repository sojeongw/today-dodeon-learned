# 템플릿 메서드 패턴과 콜백 패턴

## 템플릿 매서드 패턴

```java

@Slf4j
public class TemplateMethodTest {

    @Test
    void templateMethodV0() {
        logic1();
        logic2();
    }

    private void logic1() {
        long startTime = System.currentTimeMillis();
        // 비즈니스 로직 실행
        log.info("비즈니스 로직1 실행");
        // 비즈니스 로직 종료
        long endTime = System.currentTimeMillis();
        long resultTIme = endTime - startTime;
        log.info("resultTIme = {}", resultTIme);
    }

    private void logic2() {
        long startTime = System.currentTimeMillis();
        // 비즈니스 로직 실행
        log.info("비즈니스 로직1 실행");
        // 비즈니스 로직 종료
        long endTime = System.currentTimeMillis();
        long resultTIme = endTime - startTime;
        log.info("resultTIme = {}", resultTIme);
    }
```

- 가운데에 있는 핵심 로직을 제외하고는 중복된다.

```java

@Slf4j
public abstract class AbstractTemplate {

    public void execute() {
        long startTime = System.currentTimeMillis();

        call();

        long endTime = System.currentTimeMillis();
        long resultTIme = endTime - startTime;
        log.info("resultTIme = {}", resultTIme);
    }

    protected abstract void call();
}

@Slf4j
public class SubClassLogic1 extends AbstractTemplate {
    @Override
    protected void call() {
        log.info("비즈니스 로직1 실행");
    }
}


@Slf4j
public class TemplateMethodTest {

    @Test
    void templateMethodV0() {
        logic1();
        logic2();
    }

    @Test
    void templateMethodV1() {
        AbstractTemplate template1 = new SubClassLogic1();
        template1.execute();

        AbstractTemplate template2 = new SubClassLogic2();
        template2.execute();
    }
}
```

- 템플릿 메서드 패턴을 이용하면 공통 로직을 분리할 수 있다.
- 수정 사항이 있으면 이제 각 부분만 고치면 된다.
    - SRP 단일 책임 원칙을 지켜 변경에 유리하다.

```java

@Slf4j
public class TemplateMethodTest {
    @Test
    void templateMethodV2() {
        AbstractTemplate template1 = new AbstractTemplate() {
            @Override
            protected void call() {
                log.info("비즈니스 로직1 실행");
            }
        };

        template1.execute();
    }
}
```

- 템플릿 메서드 패턴은 서브 클래스를 계속 만들어야 하는 단점이 있다.
- 익명 내부 클래스를 활용하면 편리하게 사용할 수 있다.

### 단점

- 템플릿 메서드 패턴은 상속을 사용하기 때문에 상속에서 오는 단점을 그대로 가진다.
- 자식 클래스가 부모 클래스와 컴파일 시점에 강결합된다.
    - 자식 클래스 입장에서는 부모 클래스 기능을 전혀 사용하지 않는데 상속을 다 받아야 한다.
    - 결국 이건 의존 관계에 대한 문제다.
    - 자식 클래스에는 부모 클래스가 extends 옆에 선언되어 있으므로 부모를 사용하지 않아도 강하게 의존한다.
        - 강하게 의존한다 = 자식 클래스 코드에 부모 클래스가 명확하게 적혀있다.
        - 부모 클래스에 뭔가 추가되거나 수정되면 자식에도 영향을 준다.

## 전략 패턴

- 템플릿 메서드 패턴과 비슷한 기능을 하면서 상속의 단점을 제거할 수 있는 패턴이다.

```java
public interface Strategy {
    void call();
}

@Slf4j
public class StrategyLogic1 implements Strategy {
    @Override
    public void call() {
        log.info("비즈니스 로직1 실행");
    }
}

// 필드에 전략을 보관하는 방식
@Slf4j
public class ContextV1 {

    private Strategy strategy;

    public ContextV1(Strategy strategy) {
        this.strategy = strategy;
    }

    public void execute() {
        long startTime = System.currentTimeMillis();

        strategy.call();

        long endTime = System.currentTimeMillis();
        long resultTIme = endTime - startTime;
        log.info("resultTIme = {}", resultTIme);
    }
}


```

- 변하지 않는 로직을 context에 두고 strategy 인터페이스에만 의존한다.
    - 구현체를 변경하거나 새로 만들어도 Context 코드에는 영향을 주지 않는다.
- 스프링 의존 관계 주입에서 사용하는 방식이 바로 이 전략 패턴이다.

```java

@Slf4j
public class ContextV1Test {

    @Test
    void strategyV1() {
        StrategyLogic1 strategyLogic1 = new StrategyLogic1();
        ContextV1 contextV1 = new ContextV1(strategyLogic1);
        contextV1.execute();
    }

    @Test
    void strategyV2() {
        Strategy strategy = new Strategy() {
            @Override
            public void call() {
                log.info("비즈니스 로직1 실행");
            }
        };

        ContextV1 contextV1 = new ContextV1(strategy);
        contextV1.execute();
    }

    @Test
    void strategyV3() {
        ContextV1 contextV1 = new ContextV1(new Strategy() {
            @Override
            public void call() {
                log.info("비즈니스 로직1 실행");
            }
        });
        contextV1.execute();
    }

    @Test
    void strategyV4() {
        ContextV1 contextV1 = new ContextV1(() -> log.info("비즈니스 로직1 실행"));
        contextV1.execute();
    }
}

```

- 구현체를 정의하거나 익명 내부 클래스, 람다를 사용할 수 있다.
- 선조립 후실행 방식에서 유용하다.
    - 조립한 이후에는 execute()만 실행하면 된다.
    - 그 이후엔 조립에 대해 더 이상 고민하지 않아도 된다.
    - 스프링에서 애플리케이션 로딩 시점에 의존 관계 주입을 모두 마쳐둔 다음 실제 요청을 처리하는 것과 같은 원리다.

### 단점

- 조립한 이후에는 전략을 변경하기가 번거롭다.
- Context를 싱글톤으로 사용한다면 동시성 이슈가 있어 고려할 게 많아진다.
- 전략을 실시간으로 변경해야 하면 차라리 다른 Context 하나를 생성하는 게 낫다.

```java
// 파라미터에 전략을 보관하는 방식
@Slf4j
public class ContextV2 {

    public void execute(Strategy strategy) {
        long startTime = System.currentTimeMillis();

        strategy.call();

        long endTime = System.currentTimeMillis();
        long resultTIme = endTime - startTime;
        log.info("resultTIme = {}", resultTIme);
    }
}

@Slf4j
public class ContextV2Test {

    @Test
    void strategyV1() {
        ContextV2 context = new ContextV2();
        context.execute(new StrategyLogic1());
    }

    @Test
    void strategyV2() {
        ContextV2 context = new ContextV2();
        context.execute(() -> log.info("비즈니스 로직1 실행"));
    }
}

```

- 그때그때 파라미터로 인수를 전달해서 유연하게 실행할 수도 있다.
- 하지만 역시 실행할 때마다 전략을 지정해줘야 한다.

## 템플릿 콜백 패턴

- 앞서 배운 내용에서 컨텍스트는 변하지 않는 템플릿 역할을, 변하는 부분은 파라미터로 넘어온 전략의 코드를 실행해서 처리한다.
- 이렇게 다른 코드의 인수로서 넘겨주는 실행 가능한 코드를 콜백이라고 한다.
- 스프링에서는 ContextV2 같은 방식의 전략 패턴을 템플릿 콜백 패턴이라고 한다.
    - GoF 패턴은 아니고, 스프링 내부에서 이런 방식을 자주 사용하기 때문에 스프링 안에서만 이렇게 부른다.
    - 스프링에 Template이 붙은 클래스가 있다면 템플릿 콜백 패턴으로 만들어져 있다고 생각하면 된다.