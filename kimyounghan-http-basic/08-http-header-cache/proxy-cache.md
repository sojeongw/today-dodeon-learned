# 프록시 캐시

![](../../.gitbook/assets/kimyounghan-http-web-basic/08/screenshot%202021-04-04%20오후%208.11.32.png)

클라이언트가 아주 멀리 있는 서버에 접근한다면 시간이 오래 걸린다.

![](../../.gitbook/assets/kimyounghan-http-web-basic/08/screenshot%202021-04-04%20오후%208.11.37.png)

그래서 중간에 프록시 캐시 서버를 둔다. CDN 서비스라고 해서 대표적인 것이 AWS Cloud Front다. 

요청이 오면 원 서버가 아니라 프록시 캐시 서버를 거친다. 좀 더 가까이 있으므로 응답 시간이 빨라진다. 처음 요청한 유저는 느리지만 그 다음 요청한 유저는 빠르게 사용할 수 있다. 아니면 아예 캐시에 원 서버 데이터를 밀어 넣는 경우도 있다.

![](../../.gitbook/assets/kimyounghan-http-web-basic/08/screenshot%202021-04-04%20오후%208.11.41.png)

프록시 캐시 서버처럼 공용으로 사용하는 곳은 public 캐시, 로컬에서 사용하는 웹 브라우저 캐시는 private 캐시라고 한다.

## Cache-Control

- 캐시 지시어(directives)
- Cache-Control: public
    - 응답을 public 캐시에 저장할 수 있다.
- Cache-Control: private
    - 기본 값
    - 로그인 정보 등 해당 사용자만을 위한 응답인 private 캐시에 저장해야 한다.
- Cache-Control: s-maxage
    - 프록세 캐시에만 적용되는 max-age 값
- Age: 숫자
    - HTTP 헤더 값
    - 초 단위
    - 오리진 서버에서 응답 후 프록시 캐시에서 머문 시간