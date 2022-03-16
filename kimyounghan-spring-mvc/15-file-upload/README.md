# 파일 업로드

- HTML Form으로 파일 업로드를 하려면 두 가지 방식을 이해해야 한다.
    - application/x-www-form-urlencoded
    - multipart/form-data

## application/x-www-form-urlencoded

- HTML Form 데이터를 서버로 전송하는 가장 기본적인 방법

![](../../.gitbook/assets/kimyounghan-spring-mvc/15/screenshot%202022-03-27%20오전%208.36.09.png)

- Form 태그에 별도의 enctype 옵션이 없다면 웹 브라우저는 요청 HTTP 메시지 헤더에 내용을 추가한다.
    - content-type
        - application/x-www-form-urlencoded
    - Form 데이터
        - username=kim&age=20
        - HTTP Body에 데이터를 &로 구분해 문자로 전송한다.

### 문제점

- 파일을 업로드 하려면 문자 대신 바이너리 데이터를 전송해야 한다.
- 보통은 Form으로 데이터를 보낼 때 파일만 전송하지 않는다.
    - 이름, 나이와 첨부파일을 동시에 전송할 경우 문자와 바이너리를 한 번에 처리해야 한다.

## multipart/form-data

- application/x-www-form-urlencoded 방식의 문제를 해결하기 위한 전송 방식
- 파일을 다른 종류의 데이터와 함께 전송할 수 있다.

![](../../.gitbook/assets/kimyounghan-spring-mvc/15/screenshot%202022-03-27%20오전%208.36.29.png)

- 각 항목을 구분해 한 번에 전송한다.
- enctype="multipart/form-data"
    - multipart/form-data를 사용하기 위해 Form 태그에 지정해야 한다.
- Content-Disposition 헤더
    - 각 항목마다 부가 정보를 추가한다.
    - 일반 데이터는 각 데이터 별로 문자를 전송한다.
    - 파일은 파일 이름과 content-type을 추가하고 바이너리 데이터를 전송한다.

### Part

- application/x-www-form-urlencoded에 비해 복잡하고 각각의 part로 나눠져 있다.