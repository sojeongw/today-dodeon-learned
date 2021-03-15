# 전송 방식
## 단순 전송(Content-Length)

![](../../.gitbook/assets/kimyounghan-http-web-basic/07/screenshot%202021-04-04%20오후%205.38.36.png)

요청을 하면 미리 컨텐츠의 길이를 알고 지정해서 보내는 방식이다.

## 압축 전송(Content-Encoding)

![](../../.gitbook/assets/kimyounghan-http-web-basic/07/screenshot%202021-04-04%20오후%205.38.40.png)

서버에서 gzip 등으로 압축을 해서 용량을 확 줄인 후 Content-Encoding에 뭘로 압축했는지 정보를 같이 보낸다. 그래야 클라이언트에서 알고 풀 수 있다.

## 분할 전송(Transfer-Encoding)

![](../../.gitbook/assets/kimyounghan-http-web-basic/07/screenshot%202021-04-04%20오후%205.38.45.png)

덩어리로 쪼개서 보낼 것이라는 의미로 `chunked` 값을 주는 방식이다.

![](../../.gitbook/assets/kimyounghan-http-web-basic/07/screenshot%202021-04-04%20오후%205.38.50.png)

`5Byte를 먼저 보낼 건데 그 값은 Hello야`라고 쪼개서 클라이언트에 보낸다. 끝났으면 `0\r\n`을 보낸다.

한 번에 쭉 보내려면 대기 시간이 필요한데 이렇게 쪼개면 만들어지는 대로 바로바로 응답을 받을 수 있는 장점이 있다.

이 방식은 Content-Length를 사용할 수 없다. 쪼개서 보내므로 길이를 예상할 수 없기 때문이다. 

## 범위 전송(Range, Content-Range)

![](../../.gitbook/assets/kimyounghan-http-web-basic/07/screenshot%202021-04-04%20오후%205.38.54.png)

절반 정도 받고 끊겼는데 처음부터 다시 요청하기엔 아까우니 범위를 지정해서 요청하는 방식이다.