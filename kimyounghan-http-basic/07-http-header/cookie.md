# 쿠키

![](../../.gitbook/assets/kimyounghan-http-web-basic/07/screenshot%202021-04-04%20오후%206.25.03.png)

웹 브라우저에서 어떤 서버에 처음 접근을 했다고 해보자.

![](../../.gitbook/assets/kimyounghan-http-web-basic/07/screenshot%202021-04-04%20오후%206.25.08.png)

로그인을 하면 서버는 로그인을 했다고 응답을 준다.

![](../../.gitbook/assets/kimyounghan-http-web-basic/07/screenshot%202021-04-04%20오후%206.25.12.png)

로그인 후 다시 접근을 하면 서버는 처음 접근한 것과 같은 응답을 준다. `/welcome`에 접근한다는 것 말고는 어떤 정보도 없기 때문에 서버는 이 사람이 로그인을 했는지 알 수 없다. 

HTTP는 무상태 프로토콜이다. 클라이언트와 서버가 요청과 응답을 주고 받으면 연결이 끊긴다. 서로 상태를 유지하지 않기 떄문에 클라이언트가 다시 요청해도 서버는 이전 요청을 기억하지 못한다. 

![](../../.gitbook/assets/kimyounghan-http-web-basic/07/screenshot%202021-04-04%20오후%206.25.18.png)

대안으로 모든 요청에 사용자 정보를 포함해서 보낼 수 있다.

![](../../.gitbook/assets/kimyounghan-http-web-basic/07/screenshot%202021-04-04%20오후%206.25.23.png)

하지만 이 대안은 모든 요청과 링크에 사용자 정보를 포함하기 때문에 보안 문제부터 시작해서 개발이 복잡해진다. 브라우저를 완전히 종료하고 다시 열 경우, 로컬에 web storage를 사용하면 되지만 개발이 복잡해진다.

![](../../.gitbook/assets/kimyounghan-http-web-basic/07/screenshot%202021-04-04%20오후%206.25.30.png)

그래서 쿠키가 도입되었다. 브라우저가 로그인을 하면 서버가 `Set-Cookie` 헤더에 정보를 넣어 반환한다. 그럼 웹 브라우저 내부 쿠키 저장소에 `Set-Cookie` 값을 저장한다.

![](../../.gitbook/assets/kimyounghan-http-web-basic/07/screenshot%202021-04-04%20오후%206.25.34.png)

로그인 이후에 다시 페이지에 접근하면 자동으로 쿠키 저장소에서 조회해서 `Cookie`에 유저 값을 넣어 보낸다. 지저분하게 URL에 넣을 필요가 없어진다.

![](../../.gitbook/assets/kimyounghan-http-web-basic/07/screenshot%202021-04-04%20오후%206.25.39.png)

쿠키는 모든 요청에 쿠키 정보를 자동으로 포함시킨다.

```text
set-cookie: sessionId=abcde1234; expires=Sat, 26-Dec-2020 00:00:00 GMT; path=/; domain=.google.com; Secure
```

유저 정보를 그대로 보내면 위험하기 때문에 세션 키를 서버에서 만들어서 DB에 저장해두고 `sessionId` 값으로 보낸다. 클라이언트는 그 `sessionId`를 저장해두고 요청할 때마다 보낸다.

## 사용처

- 사용자 로그인 세션 관리
- 광고 정보 트래킹
    - 이 웹브라우저를 쓰는 사람이 이런 광고를 본다는 정보를 수집한다.
    
## 단점

쿠키 정보는 항상 서버에 전송되므로 네트워크 트래픽을 유발한다. 그래서 최소한의 정보만 사용해야 한다. 세션 ID와 인증 토큰 정도. 주민 번호와 신용카드 번호 등 보안에 민감한 데이터는 저장하면 안된다.

서버에 전송하지 않고 클라이언트 즉, 웹 브라우저 내부에 데이터를 저장하고 싶다면 웹 스토리지(localStorage, sessionStorage)를 사용한다. 

## 생명 주기(expires, max-age)

```text
Set-Cookie: expires=Sat, 26-Dec-2020 04:39:21 GMT
```

만료일이 되면 쿠키를 삭제한다.

```text
Set-Cookie: max-age=3600 (3600초)
```

0이나 음수를 지정하면 쿠키를 삭제한다.

### 세션 쿠키

만료 날짜를 생략하면 브라우저를 종료할 때까지만 유지한다.

### 영속 쿠키

만료 날짜를 입력하면 해당 날짜까지 유지한다.

## domain

쿠키가 사용되는 도메인을 명시할 수 있다.

### 명시

```text
domain=example.org
```

- 명시한 문서 기준의 도메인과 서브 도메인에 제공한다.
    - `example.org`는 물론이고 `dev.example.org`의 형태에도 쿠키로 접근한다.
    
### 생략

- 현재 문서 기준의 도메인만 적용한다.
    - `example.org`에서만 쿠키로 접근한다.
    - `dev.example.org`는 쿠키로 접근할 수 없다.

## path

```text
path=/home
```

- 해당 경로를 포함한 하위 경로 페이지만 쿠키로 접근한다.
- 일반적으로는 `path=/루트`로 지정한다. 
- `/home`, `/home/level1`, `/home/level1/level2`: 가능 
- `/hello`: 불가능

## 보안
### Secure

원래 쿠키는 http, https를 구분하지 않고 전송한다. 하지만 Secure를 적용하면 https인 경우에만 전송한다.

### HttpOnly

XSS 공격을 방지한다. 자바스크립트에서 원래 쿠키를 접근할 수 있는데(document.cookie) 이 옵션이 들어가면 불가능하다. HTTP를 통해서만 쿠키를 사용하게 된다.

### SameSite

XSRF 공격을 방지한다. 요청한 도메인과 쿠키에 설정된 도메인이 같을 때만 쿠키를 전송하는 것이다. 다른 사이트에서 세팅한 쿠키를 전송되지 않게 한다.