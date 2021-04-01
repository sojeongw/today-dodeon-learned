# 빈 팩토리와 애플리케이션 컨텍스트

![](../../.gitbook/assets/kimyounghan-spring-core-principle/04/screenshot%202021-04-10%20오후%205.22.59.png)

## BeanFactory

- 스프링 컨테이너의 최상위 인터페이스
- 스프링 빈을 관리하고 조회하는 역할
- `getBean()`을 제공한다.
- 지금까지 사용했던 대부분의 기능이 빈 팩토리의 기능이다.

## ApplicationContext

- 빈 팩토리 기능을 모두 상속받아서 제공한다.
- 애플리케이션을 개발 시 빈 관리, 조회에 추가로 필요한 수 많은 부가 기능을 가지고 있다.

## ApplicationContext의 부가 기능

![](../../.gitbook/assets/kimyounghan-spring-core-principle/04/screenshot%202021-04-10%20오후%205.23.05.png)

- 메시지 소스를 활용한 국제화 기능
    - 한국에서 들어오면 한국어로, 영어권에서 들어오면 영어로 출력한다.
- 환경 변수
    - 로컬, 개발, 운영 등을 구분해서 처리한다.
- 애플리케이션 이벤트
    - 이벤트를 발행하고 구독하는 모델을 편리하게 지원한다.
- 편리한 리소스 조회
    - 파일, 클래스 패스, 외부 등에서 리소스를 편리하게 조회할 수 있다.
    
## 정리

- ApplicationContext는 BeanFactory의 기능을 상속받는다.
- ApplicationContext는 빈 관리 기능과 편리한 부가 기능을 제공한다.
- BeanFactory를 직접 사용할 일은 거의 없다. 부가 기능이 포함된 ApplicationContext를 사용한다.
- BeanFactory나 ApplicationContext를 스프링 컨테이너라고 한다.