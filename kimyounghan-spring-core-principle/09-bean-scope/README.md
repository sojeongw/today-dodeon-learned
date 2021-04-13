# 빈 스코프

스코프는 빈이 존재할 수 있는 범위를 말한다.

지금까지는 스프링 빈이 스프링 컨테이너의 시작에서 종료까지 유지된다고 배웠다. 스프링 빈이 기본적으로 싱글턴 스코프이기 때문이다.

## 종류

### singleton

- 기본 스코프
- 스프링 컨테이너의 시작과 종료까지 유지되는 가장 넒은 범위의 스코프

### prototype

- 스프링 컨테이너가 프로토타입 빈의 생성과 의존 관계 주입까지만 관여한다.
- 매우 짧은 범위의 스코프

### request

- 웹 요청이 들어오고 나갈 때까지 유지되는 스코프

### session

- 웹 세션이 생성되고 종료될 때까지 유지되는 스코프

### application

- 웹 서블릿 컨텍스트와 같은 범위로 유지되는 스코프

## 빈 스코프 지정 방법

### 컴포넌트 스캔 자동 등록

```java

@Scope("prototype")
@Component
public class HelloBean() {

}
```

### 수동 등록

```java
public class PrototypeBean {
  @Scope("prototype")
  @Bean
  PrototypeBean HelloBean() {
    return new HelloBean();
  }
}
```