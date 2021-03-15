# 일반 정보와 특별한 정보

## 일반 정보

### From

- 유저 에이전트의 이메일 정보
- 요청에서 사용하는 정보
- 잘 사용하지 않는다.
- 검색 엔진 등에서 주로 사용한다.
    - 내 정보를 크롤링 해가는 곳에 원하지 않는다고 연락할 용도 등

### Referer

- 현재 요청된 페이지의 이전 웹 페이지 주소
- 요청에서 사용하는 정보
- 굉장히 자주 사용한다.
- A에서 B로 이동하는 경우, B 요청 시 `Referer: A`를 포함해서 요청한다.
- Referer 정보로 유입 경로를 분석할 수 있다.

### User-Agent

```text
user-agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/ 537.36 (KHTML, like Gecko) Chrome/86.0.4240.183 Safari/537.36
```

- 클라이언트의 애플리케이션 정보
    - 웹 브라우저 정보 등
- 요청에서 사용하는 정보
- 어떤 종류의 브라우저에서 장애가 발생하는지 파악할 수 있다.
- 사용자 통계 정보를 뽑기에 좋다.

### Server

```text
Server: Apache/2.2.22 (Debian)
```

- 요청을 처리하는 origin 서버의 소프트웨어 정보
    - HTTP를 사용하면 중간에 여러 프록시 서버를 거친다. 실제 응답을 해주는 진짜 서버를 origin이라고 한다.
- 응답에서 사용하는 정보

### Date

```text
Date: Tue, 15 Nov 1994 08:12:31 GMT
```

- 메시지가 발생한 날짜와 시간
- 응답에서 사용하는 정보

## 특별한 정보

### Host(도메인)

![](../../.gitbook/assets/kimyounghan-http-web-basic/07/screenshot%202021-04-04%20오후%206.12.42.png)

- 요청에서 사용하는 정보
- 필수 값

![](../../.gitbook/assets/kimyounghan-http-web-basic/07/screenshot%202021-04-04%20오후%206.12.47.png)

하나의 서버가 여러 도메인을 처리해야 할 때 즉, 한 IP에 여러 도메인이 적용된 경우 사용한다. 예를 들어, 가상 호스트로 여러 도메인을 한 번에 처리하는 서버가 있어서 애플리케이션이 여러 개 구동되고 있을 수 있다.

![](../../.gitbook/assets/kimyounghan-http-web-basic/07/screenshot%202021-04-04%20오후%206.12.53.png)

host가 없으면 서버 입장에서는 `/hello`가 어떤 애플리케이션에 해당되는지 알 수가 없다. 통신은 IP로만 하기 때문이다.

![](../../.gitbook/assets/kimyounghan-http-web-basic/07/screenshot%202021-04-04%20오후%206.12.58.png)

host를 담아 전달하면 서버는 적절한 곳으로 보낸다.

### Location

- 페이지 리다이렉션에 쓰인다.
- 웹 브라우저가 3xx 응답 결과에 Location이 있으면 그 위치로 자동 이동한다.
- 201 Created일 땐 요청에 의해 생성된 리소스 URI 값이 들어온다.
- 3xx Redirection일 땐 요청을 자동으로 리다이렉션 하기 위한 대상 리소스를 가리킨다.

### Allow

```text
Allow: GET, HEAD, PUT
```

- 허용 가능한 HTTP 메서드를 나타낸다.
- 405 Method Not Allowed일 경우 응답에 포함해야 한다.

### Retry-After

```text
Retry-After: Fri, 31 Dec 1999 23:59:59 GMT (요청 가능한 날짜 표기) 
Retry-After: 120 (기다려야 할 초 표기)
```

- 유저 에이전트가 다음 요청을 하기까지 기다려야 하는 시간
- 503 Service Unavailable일 때 서비스가 언제까지 불능인지 알려줄 수 있다.