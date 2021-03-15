# 표현

![](../../.gitbook/assets/kimyounghan-http-web-basic/07/screenshot%202021-04-04%20오후%203.40.40.png)

표현 헤더는 전송, 응답 둘 다에 사용된다.

- Content-Type
    - 표현 데이터의 형식
    - html인지 json인지 등을 알려준다.
- Content-Encoding
    - 표현 데이터의 압축 방식
- Content-Language
    - 표현 데이터의 자연 언어
- Content-Length
    - 표현 데이터의 길이
    - 엄밀히 말하면 payload 헤더에 해당한다.
    
## Content-Type

![](../../.gitbook/assets/kimyounghan-http-web-basic/07/screenshot%202021-04-04%20오후%203.40.47.png)

미디어 타입, 문자 인코딩 등 표현 데이터의 형식 즉, 바디에 어떤 데이터가 들어가는지 설명한다.

- text/html; charset=utf-8
- application/json
  - 기본이 utf-8
- image/png

## Content-Encoding

![](../../.gitbook/assets/kimyounghan-http-web-basic/07/screenshot%202021-04-04%20오후%203.40.52.png)

표현 데이터를 압축할 때 사용한다. 데이터를 전달하는 곳에서 메시지 바디를 압축하고 그 압축 정보를 인코딩 헤더에 추가해 보낸다. 그럼 데이터를 읽는 쪽에서 이 정보로 압축을 해제하게 된다.

- gzip
- deflate
- identity
  - 압축 안한다는 의미

## Content-Language

![](../../.gitbook/assets/kimyounghan-http-web-basic/07/screenshot%202021-04-04%20오후%203.41.01.png)

표현 데이터의 자연 언어를 표현한다. 글로벌 서비스에서 언어를 선택할 때 쓰인다.

- ko
- en
- en-US

## Content-Length

![](../../.gitbook/assets/kimyounghan-http-web-basic/07/screenshot%202021-04-04%20오후%203.41.07.png)

표현 데이터의 길이를 바이트 단위로 나타낸다. Transfer-Encoding(전송 코딩)을 쓸 때는 전송 코딩 안에 정보가 다 들어있기 때문에 사용하면 안된다.