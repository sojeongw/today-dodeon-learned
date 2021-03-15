# 캐시 무효화

```text
Cache-Control: no-cache, no-store, must-revalidate
```

```text
Pragma: no-cache
```

설정과 관계 없이 웹 브라우저가 임의로 캐시를 하는 경우가 있는데 이를 막기 위한 설정이다.

## Cache-Control: no-cache

- 데이터는 캐시해도 되지만 항상 origin 서버에 검증하고 사용한다.
- 이름에 속지말자.

## Cache-Control: no-store

- 데이터에 민감한 정보가 있으므로 저장하면 안된다.
- 메모리에서 사용하고 최대한 빨리 삭제한다.

## Cache-Control: must-revalidate

- 캐시 만료 후 최초로 조회할 때 origin 서버에 검증해야 한다.
- 원 서버 접근에 실패하면 반드시 `504 Gateway Timeout`이 발생해야 한다.
- 캐시 요휴 시간 내에 있다면 캐시를 사용한다.

## Pragma: no-cache

- HTTP 1.0 하위 호환용이다.

![](../../.gitbook/assets/kimyounghan-http-web-basic/08/screenshot%202021-04-04%20오후%208.11.48.png)

클라이언트가 `no-cache`로 서버에 요청하면 프록시 서버는 `no-cache`이므로 원 서버에 요청을 전달한다.

![](../../.gitbook/assets/kimyounghan-http-web-basic/08/screenshot%202021-04-04%20오후%208.11.52.png)

원 서버는 적절한 응답을 내려준다.

![](../../.gitbook/assets/kimyounghan-http-web-basic/08/screenshot%202021-04-04%20오후%208.11.57.png)

만약 프록시 캐시 서버가 원 서버에 요청을 보내야하는데 순간적으로 네트워크가 단절돼서 접근이 불가하다면 설정에 따라 프록시 서버에서 캐시 데이터를 반환할 수 있다.

![](../../.gitbook/assets/kimyounghan-http-web-basic/08/screenshot%202021-04-04%20오후%208.12.02.png)

하지만 `must-revalidate`라면 원 서버에 접근이 불가할 때 `504 Gateway Timeout` 오류를 보낸다. 통장 잔고 등 중요한 정보가 원 서버를 못 탔다고 해서 예전 데이터로 뜬다면 큰 문제가 생긴다. 이런 경우 `must-revalidate`를 써야 한다. 
