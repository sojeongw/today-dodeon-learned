# HTTP 헤더 - 캐시와 조건부 요청
## 캐시 기본 동작

![](../../.gitbook/assets/kimyounghan-http-web-basic/08/screenshot%202021-04-04%20오후%207.12.33.png)

![](../../.gitbook/assets/kimyounghan-http-web-basic/08/screenshot%202021-04-04%20오후%207.12.38.png)

클라이언트에서 요청한 내용을 보고 서버가 실제 이미지와 관련된 바이트 코드를 포함한 응답을 내린다. HTTP 헤더가 0.1MB, 바디가 1.0MB로 총 1.1MB라고 가정한다.

![](../../.gitbook/assets/kimyounghan-http-web-basic/08/screenshot%202021-04-04%20오후%207.12.43.png)

![](../../.gitbook/assets/kimyounghan-http-web-basic/08/screenshot%202021-04-04%20오후%207.12.48.png)

또 다시 요청하면 첫 번째처럼 똑같이 1.1MB의 응답을 보낸다.

캐시가 없으면 데이터가 변경되지 않아도 계속 데이터를 다운받아야 한다. 인터넷 네트워크는 일반 메모리들(HDD 등)보다 느리고 비싸다. 그럼 브라우저 로딩 속도가 느려지니 사용자 경험이 좋지 않게 된다.

![](../../.gitbook/assets/kimyounghan-http-web-basic/08/screenshot%202021-04-04%20오후%207.12.54.png)

만약 캐시를 적용한다면 처음 요청했을 때 캐시가 유효한 시간을 `cache-control`에 설정해서 응답을 보낸다.

![](../../.gitbook/assets/kimyounghan-http-web-basic/08/screenshot%202021-04-04%20오후%207.12.59.png)

웹 브라우저에는 캐시를 저장하는 공간이 있어서 유효한 시간만큼 응답 결과를 캐시에 저장해둔다.

![](../../.gitbook/assets/kimyounghan-http-web-basic/08/screenshot%202021-04-04%20오후%207.13.04.png)

다시 요청할 때는 캐시를 조회한 뒤 시간이 유효한지 확인한다.

![](../../.gitbook/assets/kimyounghan-http-web-basic/08/screenshot%202021-04-04%20오후%207.13.09.png)

유효하면 네트워크를 타지 않고 캐시에서 가져온다.

캐시 덕분에 캐시 가능 시간동안 네트워크를 사용하지 않아도 된다. 비싼 네트워크 사용량을 줄이고 브라우저 로딩 속도가 빨라진다.

![](../../.gitbook/assets/kimyounghan-http-web-basic/08/screenshot%202021-04-04%20오후%207.13.15.png)

만약 유효 시간이 초과되었다면

![](../../.gitbook/assets/kimyounghan-http-web-basic/08/screenshot%202021-04-04%20오후%207.13.19.png)

다시 서버에 요청을 한다. 서버는 다시 캐시 유효 시간을 담아 보낸다. 이떄 다시 네트워크 다운로드가 발생한다.

![](../../.gitbook/assets/kimyounghan-http-web-basic/08/screenshot%202021-04-04%20오후%207.13.24.png)

브라우저는 기존 캐시를 지우고 새 캐시를 덮어 쓴다.