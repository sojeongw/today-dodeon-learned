## Spring AutoWiring
`@autowired` 애노테이션으로 dependency injection을 할 수 있다. class나 interface 등 type으로 알맞은 property를 찾아 자동으로 주입한다.

## AutoWiring Example
1. 스프링이 `@Components`를 스캔한다.
2. `FortuneService` 인터페이스를 구현한 클래스를 찾는다.
3. `HappyFortuneService`가 구현하고 있으므로 이를 주입한다.

## AutoWiring Injection Types
- Constructor Injection
- Setter Injection
- Field Injection