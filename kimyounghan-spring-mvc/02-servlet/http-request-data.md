# HTTP 요청 데이터

HTTP 요청 메시지를 통해 클라이언트에서 서버로 데이터를 전달하는 방법을 알아보자.

## GET 쿼리 파라미터

```text
/url?username=hello&age=20
```

- 메시지 바디 없이 URL의 쿼리 파라미터에 데이터를 포함해서 전달한다. 
- 검색, 필터, 페이징 등에서 많이 사용하는 방식이다.

## POST HTML Form

```text
POST /save HTTP/1.1
Host: localhost:8080
Content-Type: application/x-www-form-urlencoded

username=kim&age=20
```

- 메시지 바디에 쿼리 파라미터 형식으로 전달한다.
  - username=kim&age=20
- 회원 가입, 상품 주문 등에 사용한다.

## HTTP message body

- 메시지 바디에 직접 담아서 요청한다.
- HTTP API에서 주로 사용한다.
- json, xml, text 
- 주로 json을 사용한다.
- POST, PUT, PATCH에서 사용한다.