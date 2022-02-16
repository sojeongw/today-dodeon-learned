# 스프링 부트 오류 페이지

스프링 부트는 복잡한 에러 페이지 설정 과정을 기본으로 제공한다.

- ErrorPage를 자동으로 등록한다.
    - ErrorMvcAutoConfiguration이 오류 페이지를 자동 등록한다.
    - /error 라는 경로로 기본 오류 페이지를 설정한다.
        - 별도의 ErrorPage나 상태코드, 예외를 설정하지 않으면 기본을 사용한다.
        - 서블릿 밖으로 예외가 발생하거나 sendError()가 호출되면 모든 오류는 /error를 호출한다.
- 스프링 컨트롤러 BasicErrorController를 호출한다.
    - ErrorPage에서 등록한 /error를 매핑해서 처리한다.
    - 기본적인 로직이 모두 있기 때문에 개발자는 룰과 우선순위에 따라 등록만 하면 된다.

## 뷰 선택 우선 순위

1. 뷰 템플릿
    1. resources/templates/error/500.html
    2. resources/templates/error/5xx.html
2. 정적 리소스
    - static, public 폴더
    1. resources/static/error/400.html
    2. resources/static/error/404.html
    3. resources/static/error/4xx.html
3. 적용 대상이 없을 때
    1. resources/templates/error.html

해당 경로에 HTTP 상태 코드 이름으로 뷰 파일을 넣어두면 된다. 404 등 구체적인 것이 4xx 등 포괄적인 것보다 우선 순위가 높다.