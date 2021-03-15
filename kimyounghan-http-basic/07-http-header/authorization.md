# 인증

## Authorization

```text
Authorization: Basic {value}
```

클라이언트 인증 정보를 서버에 전달한다. 들어가는 값은 인증 방식마다 다르다.

## WWW-Authenticate

```text
WWW-Authenticate: Newauth realm="apps", type=1, title="Login to \"apps\"", Basic realm="simple"
```

- 리소스에 접근할 때 필요한 인증 방법을 정의한다.
- 401 Unauthorized 응답에 함께 사용한다.