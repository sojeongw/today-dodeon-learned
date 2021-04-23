# 애플리케이션 아키텍처
## 계층형 구조
![](../../.gitbook/assets/kimyounghan-spring-boot-and-jpa/03/screenshot%202021-05-19%20오후%205.17.10.png)

- controller, web
    - 웹 계층
- service
    - 비즈니스 로직, 트랜잭션 처리
- repository
    - JPA를 직접 사용하는 계층
    - 엔티티 매니저 사용
- domain
    - 엔티티가 모여있는 계층
    - 모든 계층에서 사용
    
controller도 repository에 직접 접근할 수 있되 방향은 단방향으로 흐르도록 작업할 예정이다.
    
## 패키지 구조

- jpabook.jpashop
    - domain
    - exception
    - repository
    - service
    - web

## 개발 순서

1. service, repository 계층을 개발
2. 테스트 케이스 작성 및 검증
3. 웹 계층 개발