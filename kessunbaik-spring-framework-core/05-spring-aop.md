# 스프링 AOP

## 개념

- OOP를 보완하는 수단으로 흩어진 Aspect를 모듈화 할 수 있는 프로그래밍 기법
- 흩어져 있으면 유지보수가 어렵다.
- 관심사별로 모은 뒤 적용할 곳을 입력해준다.

![](../.gitbook/assets/keesunbaik-spring-framework-core/04/스크린샷%202022-08-14%20오후%208.37.43.png)

## AOP 주요 개념

- Aspect
    - 각 모듈
- Target
    - Aspect가 적용되는 대상
- Advice
    - 해야할 일
- Join point
    - 끼워넣을 지점
    - ex. 메서드 실행 시
- Pointcut
    - 어디에 적용되어야 하는지 지정하는 곳

## AOP 구현체

- Java
    - AspectJ
    - 스프링 AOP

## 적용 방법

- 컴파일
- 로드 타임
    - 클래스 파일을 로딩하는 시점
- 런타임