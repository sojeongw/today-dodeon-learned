# 캐시와 조건부 요청 헤더
## Cache-Control

- 캐시를 제어하는 지시어(directives)
- Cache-Control: max-age
    - 캐시 유효 시간
    - 초 단위로 설정
- Cache-Control: no-cache
    - 데이터를 캐시해도 되지만 항상 origin 서버에 변경됐는지 검증받고 사용한다.
- Cache-Control: no-store
    - 데이터에 민감한 정보가 있으므로 저장하지 않는다.
    - 메모리에서 사용하고 최대한 빨리 삭제한다.

## Pragma

```text
Pragma: no-cache
```

- `no-cache`처럼 동작한다.
- HTTP 1.0 하위 호환이 필요하면 사용한다.

## Expires

```text
expires: Mon, 01 Jan 1990 00:00:00 GMT
```

- 캐시 만료일을 초 단위가 아닌 정확한 날짜로 지정한다.
- HTTP 1.0부터 사용할 수 있다.
- 지금은 초 단위를 사용해 더 유연한 `Cache-Control: max-age`를 권장한다.
- `Cache-Control: max-age`와 `Expires`를 함께 사용하면 `Expires`는 무시된다.

## 검증 헤더(validator)

- ETag
- Last-Modified

## 조건부 요청 헤더

- If-Match, If-None-Match
    - ETag 사용
- If-Modified-Since, If-Unmodified-Since
    - Last-Modified 사용