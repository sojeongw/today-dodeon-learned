# 검증 헤더와 조건부 요청

![](../../.gitbook/assets/kimyounghan-http-web-basic/08/screenshot%202021-04-04%20오후%207.25.26.png)

캐시 유효 시간이 만료되어 다시 요청했는데 데이터가 변경되지 않아 그대로 다운받아야 한다면 네트워크 낭비일 수 있다.

![](../../.gitbook/assets/kimyounghan-http-web-basic/08/screenshot%202021-04-04%20오후%207.25.31.png)

다시 무거운 데이터를 다운받는 대신에, 클라이언트 캐시 데이터와 서버 데이터가 같은지 확인하고 같으면 캐시를 다시 쓸 수 있다.

![](../../.gitbook/assets/kimyounghan-http-web-basic/08/screenshot%202021-04-04%20오후%207.25.36.png)

첫 요청에서 서버가 데이터 최종 수정일을 담아 응답한다.

![](../../.gitbook/assets/kimyounghan-http-web-basic/08/screenshot%202021-04-04%20오후%207.25.41.png)

웹 브라우저는 최종 수정일과 함꼐 응답 결과를 캐시에 저장한다.

![](../../.gitbook/assets/kimyounghan-http-web-basic/08/screenshot%202021-04-04%20오후%207.25.46.png)

다시 요청할 때 캐시 유효 시간이 초과됐다면

![](../../.gitbook/assets/kimyounghan-http-web-basic/08/screenshot%202021-04-04%20오후%207.25.51.png)

최종 수정일이 있는지 확인하고 `if-modified-since`에 담아 서버에 보낸다.

![](../../.gitbook/assets/kimyounghan-http-web-basic/08/screenshot%202021-04-04%20오후%207.25.56.png)

![](../../.gitbook/assets/kimyounghan-http-web-basic/08/screenshot%202021-04-04%20오후%207.26.01.png)

서버는 `if-modified-since`를 보고 수정일을 비교한다. 만약 수정되지 않았다면

![](../../.gitbook/assets/kimyounghan-http-web-basic/08/screenshot%202021-04-04%20오후%207.26.06.png)

응답에 `303 Not Modified`와 `Last-Modified`를 담아 보낸다. 이때, HTTP Body는 비워서 보낸다. 그럼 0.1MB만큼의 헤더만 전송된다. 네트워크
부하가 확 줄어든다.

![](../../.gitbook/assets/kimyounghan-http-web-basic/08/screenshot%202021-04-04%20오후%207.26.11.png)

![](../../.gitbook/assets/kimyounghan-http-web-basic/08/screenshot%202021-04-04%20오후%207.26.17.png)

클라이언트는 304 코드를 보고 캐시를 그대로 사용하면서 헤더 데이터를 갱신한다.

### 정리

- 검증 헤더 `Last-Modified`와 조건부 요청 `if-modified-since`를 조합해 캐시 갱신 여부를 확인한다.
- 캐시 유효 시간이 초과해도 서버 데이터가 갱신되지 않았다면 `304 Not Modified`와 바디 없이 헤더 메타 정보만 응답한다.
- 클라이언트는 캐시에 저장된 데이터를 재활용한다.
- 네트워크 다운로드가 매우 소량만 발생한다.

## 검증 헤더

- 캐시 데이터와 서버 데이터가 같은지 검증하는 데이터
- `last-modified`, `ETag`

## 조건부 요청 헤더

- 검증 헤더로 조건에 따라 분기한다.
- `if-modified-since`
    - `last-modified` 사용
- `if-none-match`
    - `ETag` 사용
- 조건을 만족하면 `200 OK`, 만족하지 않으면 `304 Not Modified`를 보낸다.

### last-modified, if-modified-since

해당 날짜 이후에 데이터가 수정되었는지 묻는다.

```text
2020년 11월 10일 10:00:00 vs 서버: 2020년 11월 10일 10:00:00
```

이 경우 데이터가 변경되지 않았으므로 `304 Not Modified`와 헤더 데이터만 전송한다. 바디는 포함되지 않는다.

```text
2020년 11월 10일 10:00:00 vs 서버: 2020년 11월 10일 11:00:00
```

이 경우엔 데이터가 변경되었으므로 `200 OK`와 함꼐 바디를 포함한 모든 데이터를 전송한다.

- 1초 미만(0.x 초) 단위로 캐시 조정이 불가하다.
    - 예시를 보면 알 수 있듯 초 단위이기 때문이다.
- 날짜 기반으로 로직을 사용한다.
  - a를 b로 수정했다가 다시 a가 됐어도 다시 다운받아야 한다.
  
### ETag(Entity Tag), if-none-match

```text
ETag: "v1.0"
ETag: "a2jiodwjekjl3"
```
캐시용 데이터에 임의의 고유한 버전 이름을 달아둔다.

```text
ETag: "aaaaa" -> ETag: "bbbbb"
```

데이터가 변경되면 Hash를 다시 생성한다. Hash는 파일 컨텐츠가 동일하면 동일한 값으로 나온다. 단순하게 ETag만 비교해서 같으면 유지하고 다르면 다시 받는 것이다.

![](../../.gitbook/assets/kimyounghan-http-web-basic/08/screenshot%202021-04-04%20오후%207.39.49.png)

첫 요청에 응답할 때 ETag를 담아 보낸다.

![](../../.gitbook/assets/kimyounghan-http-web-basic/08/screenshot%202021-04-04%20오후%207.39.54.png)

웹 브라우저는 ETag 값을 캐시에 저장한다.

![](../../.gitbook/assets/kimyounghan-http-web-basic/08/screenshot%202021-04-04%20오후%207.39.59.png)

![](../../.gitbook/assets/kimyounghan-http-web-basic/08/screenshot%202021-04-04%20오후%207.40.08.png)

시간이 초과됐다면 `if-none-match`에 ETag 값을 담아 요청한다.

![](../../.gitbook/assets/kimyounghan-http-web-basic/08/screenshot%202021-04-04%20오후%207.40.14.png)

![](../../.gitbook/assets/kimyounghan-http-web-basic/08/screenshot%202021-04-04%20오후%207.40.42.png)

데이터가 수정되지 않았으므로 즉, none-match 조건을 만족하지 않았으므로

![](../../.gitbook/assets/kimyounghan-http-web-basic/08/screenshot%202021-04-04%20오후%207.40.48.png)

body 없이 304로 응답한다.

![](../../.gitbook/assets/kimyounghan-http-web-basic/08/screenshot%202021-04-04%20오후%207.41.18.png)

클라이언트는 헤더 데이터를 갱신하고

![](../../.gitbook/assets/kimyounghan-http-web-basic/08/screenshot%202021-04-04%20오후%207.41.24.png)

캐시에서 재사용한다.

- 단순하게 ETag만 비교한다.
- 캐시 제어 로직을 서버에서 완전히 관리한다.
- 클라이언트는 단순히 ETag 값을 제공할 뿐 캐시 메커니즘은 모른다.
- 애플리케이션 배포 죽지에 맞춰 ETag를 갱신하는 등에 사용할 수 있다.