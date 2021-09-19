# HTTP 메시지 컨버터

- 뷰 템플릿이나 HTML이 아니라 HTTP 메시지 바디에 직접 데이터를 읽고 쓰는 경우 HTTP 메시지 컨버를 쓰면 편하다.

## @ResponseBody 사용 원리

![](../../.gitbook/assets/kimyounghan-spring-mvc/06/screenshot%202022-03-01%20오후%204.46.22.png)

- 컨트롤러로 요청이 들어오면 viewResolver 대신 HttpMessageConverter가 동작한다.
    - 문자: StringHttpMessageConverter
    - 객체: MappingJackson2HttpMessageConverter
    - 기타 byte 처리 등 다양한 HttpMessageConverter 내장
- 클라이언트의 HTTP Accept 헤더와 서버의 컨트롤러 반환 타입을 조합해 HttpMessageConverter가 선택된다.

### HTTP 헤더

- content-type
    - 클라이언트가 보내는 데이터 타입
- accept
    - 클라이언트가 받을 수 있는 데이터 타입

## 메시지 컨버터 적용 대상

- 요청
    - @RequestBody
    - HttpEntity
    - RequestEntity
- 응답
    - @ResponseBody
    - HttpEntity
    - ResponseEntity

![](../../.gitbook/assets/kimyounghan-spring-mvc/06/screenshot%202022-03-01%20오후%205.00.59.png)

- canRead(), canWrite()
    - 메시지 컨버터가 해당 클래스와 미디어 타입을 지원하는지 확인한다.
- read(), write()
    - 메시지 컨버터를 이용해 메시지를 읽고 쓴다.

## 기본 메시지 컨버터

```text
0 = ByteArrayHttpMessageConverter
1 = StringHttpMessageConverter
2 = MappingJackson2HttpMessageConverter
```

### ByteArrayHttpMessageConverter

```text
# 요청
@RequestBody byte[] data

# 응답
@ResponseBody return byte[]
# 자동으로 들어가는 응답 데이터의 미디어 타입
application/octet-stream
```

- byte[] 데이터로 처리
- 클래스 타입
    - byte[]
- 미디어 타입
    - `*/*`
    - 모두 된다는 의미

### StringHttpMessageConverter

```text
# 요청
content-type: application/json
@RequestMapping
void hello(@RequestBody String data) {}

# 응답
@ResponseBody return "ok"
text/plain
```

- String 데이터로 처리
- 클래스 타입
    - String
- 미디어 타입
    - `*/*`

### MappingJackson2HttpMessageConverter

```text
# 요청
content-type: application/json
@RequestMapping
void hello(@RequestBody HelloData data) {}

# 응답
@ResponseBody return helloData
application/json
```

- JSON 데이터로 처리
- 클래스 타입
    - 객체
    - HashMap
- 미디어 타입
    - `text/plain`

## HTTP 요청 데이터 읽기

1. HTTP 요청
2. 컨트롤러의 @RequestBody, HttpEntity로 요청
3. 메시지 컨버터가 canRead()를 호출해 메시지를 읽을 수 있는지 확인
    - 대상 클래스 타입 확인
        - byte[], String, 객체
    - content-type 확인
        - text/plain, application/json, * / *
4. read()를 호출해 객체 생성 및 반환

## HTTP 응답 데이터 생성

1. 컨트롤러에서 @ResponseBody, HttpEntity로 값 반환
2. 메시지 컨버터가 canWrite()를 호출해 메시지를 읽을 수 있는지 확인
    - 대상 클래스 타입 확인
        - byte[], String, 객체
    - accept 미디어 타입
        - 정확히는 @RequestMapping의 produces 옵션을 확인한다.
        - text/plain, application/json, * / *
3. write()를 호출해 HTTP 응답 메시지 바디에 데이터 생성

```text
content-type: text/html
@RequestMapping
void hello(@RequestBody HelloData data) {}
```

만약 이렇게 되어있다면, 클래스 타입은 맞아도 미디어 타입이 application/json이 아니므로 탈락된다.
