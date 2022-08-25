# Spring Boot Dev Tools

- 스프링이 제공하는 옵셔널한 툴
- 캐시 설정을 개발 환경에 맞게 끄거나 켠다.
- 코드를 수정하면 리스타트 해준다.
    - 죽, 클래스 패스에 있는 파일이 변경 될 때마다 프로젝트 빌드를 하면 자동으로 재시작한다.
    - 직접 껐다 켜는것(cold starts)보다 빠르다.
    - 릴로딩 보다는 느리다.
    - JRebel 같은건 아니다.
- spring.devtools.restart.exclude
    - 리스타트 하고 싶지 않은 리소스
- spring.devtools.restart.enabled = false
    - 리스타트 끄기

### 라이브 릴로드

- 리스타트 했을 때 브라우저를 자동 리프레시한다.
- 브라우저 플러그인을 설치해야 한다.
- spring.devtools.liveload.enabled = false
    - 라이브 릴로드 서버 끄기

## 글로벌 설정

- ~/.spring-boot-devtools.properties
    - dev tools 플러그인이 있으면 1순위로 적용해준다.

## 리모트 애플리케이션

- 위험하기 때문에 운영 대신 개발용으로 사용한다.