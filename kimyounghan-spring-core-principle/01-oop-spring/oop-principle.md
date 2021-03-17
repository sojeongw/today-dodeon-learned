# 좋은 객체 지향 설계의 5가지 원칙(SOLID)

## SRP: 단일 책임 원칙 (Single Responsibility Principle)

- 하나의 클래스는 하나의 책임만 가진다.

하나의 책임이란 의미는 모호하다. 클 수도 있고, 작을 수도 있다. 문맥과 상황에 따라 다르다. 중요한 판단의 기준은 **변경**이다. 변경이 있을 때 파급 효과가 적으면 단일 책임 원칙을 잘 따른 것이다.

## OCP: 개방-폐쇄 원칙 (Open/Closed Principle)

- 소프트웨어 요소는 확장에는 열려있지만 변경에는 닫혀있어야 한다.

기능을 확장하려면 당연히 변경이 필요한데 어떻게 가능할까? 다형성을 활용하면 된다. 인터페이스를 구현한 새로운 클래스를 만들어서 기능을 구현한다. 지금까지 배운 역할과 구현의 분리를 생각해보자.

### 문제점

```java
public class MemberService {
//  private MemberRepository memberRepository = new MemoryMemberRepository();
  private MemberRepository memberRepository = new JdbcMemberRepository();
}
```

하지만 구현 객체를 변경하려면 클라이언트 코드를 변경해야 한다. 분명 다형성을 사용했지만, OCP 원칙을 지키지 않고 있다.

이 문제는 객체를 생성하고 연관 관계를 맺어주는 별도의 조립이나 설정자가 필요하다. 이걸 바로 스프링이 해준다.

## LSP: 리스코프 치환 원칙 (Liskov Substitution Principle) 

- 객체는 프로그램의 정확성을 깨지 않으면서 하위 타입의 인스턴스로 바꿀 수 있어야 한다.

다형성에서 하위 클래스는 인터페이스 규약을 다 지켜야 한다는 의미다. 인터페이스를 구현한 구현체를 믿고 사용하기 위함이다.

자동차 인터페이스의 엑셀은 앞으로 가라는 기능인데 뒤로 가게 구현하면 컴파일은 성공한다. 하지만 느리더라도 앞으로 간다는 규약을 꼭 치켜야 한다.

## ISP: 인터페이스 분리 원칙 (Interface Segregation Principle) 

- 특정 클라이언트를 위한 인터페이스 여러 개가 범용 인터페이스 하나보다 낫다.

자동차 인터페이스는 운전과 정비에 관련된 인터페이스로 분리할 수 있다. 그럼 사용자 클라이언트를 운전자 클라이언트와 정비사 클라이언트로 분리할 수 있어서, 정비 인터페이스 자체가 변해도 운전자 클라이언트에 영향을 주지 않는다.

이렇게 하면 인터페이스가 좀 더 명확해지고 쉽게 대체할 수 있게 된다.

## DIP: 의존관계 역전 원칙 (Dependency Inversion Principle)

- 구체적인 것이 아니라 추상적인 것에 의존해야 한다.

의존성 주입은 이 원칙을 따르는 방법 중 하나다. 구현 클래스에 의존하지 않고 인터페이스에 의존하라는 의미다. 앞서 이야기 한, 구현이 아닌 역할에 의존하라는 말과 같다.

### 문제점

```java
public class MemberService {
  private MemberRepository memberRepository = new MemoryMemberRepository();
}
```

그런데 `MemberService`는 인터페이스 뿐만 아니라 구현 클래스에도 동시에 의존한다. `MemberService` 클라이언트가 `MemoryMemberRepository`라는 구현 클래스를 직접 선택하기 때문이다. 즉, DIP를 위반한다.

## 정리

- 객체 지향의 핵심은 다형성이다.
- 하지만 다형성만으로는 쉽게 부품을 바꾸듯 개발할 수 없다.
    - 구현 객체를 변경할 때 클라이언트 코드도 함께 변경되기 때문이다.
- 다형성 만으로는 OCP, DIP를 지킬 수 없다.
- 무언가가 더 필요하다!