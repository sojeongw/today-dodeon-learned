# HTTP 헤더 - 일반 헤더

![](../../.gitbook/assets/kimyounghan-http-web-basic/07/screenshot%202021-04-04%20오후%203.39.42.png)

헤더 필드는 `field-name` + `:` + `OWS` + `field-value` + `OWS`로 이루어져 있다.(OWS = 띄어쓰기 허용) `field-name`에는 대소문자 구분이 없다.

- HTTP 전송에 필요한 모든 부가 정보
    - 메시지 바디 내용, 크기, 압축, 인증, 요청 클라이언트, 서버 정보, 캐시 관리 정보 등
- 표준 헤더가 너무 많다.
- 필요 시 임의의 헤더를 추가할 수 있다.
    - `helloworld: hihi`
    
## RFC2616(과거)
### HTTP 헤더

![](../../.gitbook/assets/kimyounghan-http-web-basic/07/screenshot%202021-04-04%20오후%203.39.55.png)

- Request Header
    - 요청 정보
    - ex) User-Agent: Mozilla/5.0 (Macintosh; ..)
- Response Header
    - 응답 정보
    - ex) Server: Apache
- General Header
    - 메시지 전체에 적용되는 정보
    - ex) Connection: close
- Entity Header
    - Entity 바디 정보
    - ex) Content-Type: text/html, Content-Length: 3423
    
### HTTP 바디

![](../../.gitbook/assets/kimyounghan-http-web-basic/07/screenshot%202021-04-04%20오후%203.40.12.png)

메시지 본문은 Entity 본문을 전달하는데에 사용한다.

- Entity 본문
    - 요청이나 응답에서 전달할 실제 데이터
- Entity 헤더
    - Entity 본문의 데이터를 해석할 수 있는 정보
    - 데이터 유형(html, json), 데이터 길이, 압축 정보 등

---

1999년에 나온 RFC2616가 폐기되고 2014년에 RFC7230~7235가 등장하면서 Entity 바디 개념이 사라지게 되었다.

- Entity 대신 표현(Representation)이 추가되었다.
- 표현은 표현 메타데이터와 표현 데이터를 합친 개념이다.

## RFC7230(최신)
### HTTP 바디

![](../../.gitbook/assets/kimyounghan-http-web-basic/07/screenshot%202021-04-04%20오후%203.40.31.png)

메시지 본문(message body = payload)을 통해 표현 데이터를 전달한다.

- 표현
    - 요청이나 응답에서 전달할 실제 데이터
- 표현 헤더
    - 표현 데이터를 해석할 수 있는 정보
    - 데이터 유형(html, json), 데이터 길이, 압축 정보 등
    - 표현 메타데이터와 페이로드 메시지를 구분하지만 여기서는 생략한다.