# 콘텐츠 협상

클라이언트가 선호하는 표현을 서버에 요청하는 것이다. 서버는 원하는 우선 순위에 최대한 맞춰서 표현 데이터를 만들어주게 된다. 물론 안될 가능성도 있다. 이 협상 헤더는 요청할 때만 사용한다.

## Accept




## Accept-Charset

## Accept-Encoding

## Accept-Language

![](../../.gitbook/assets/kimyounghan-http-web-basic/07/screenshot%202021-04-04%20오후%205.37.58.png)

Accept-Language를 적용하기 전에는 기본값인 영어로 응답을 한다.

![](../../.gitbook/assets/kimyounghan-http-web-basic/07/screenshot%202021-04-04%20오후%205.38.02.png)

적용하면 선호하는 언어를 확인하고 있으면 그 언어로 보내준다.

### 우선 순위

![](../../.gitbook/assets/kimyounghan-http-web-basic/07/screenshot%202021-04-04%20오후%205.38.07.png)

하지만 만약 기본이 독일어고 영어를 지원하는 곳이면 한국어가 안되더라도 영어로 보기를 원하지만 독일어로 나온다.

![](../../.gitbook/assets/kimyounghan-http-web-basic/07/screenshot%202021-04-04%20오후%205.38.12.png)

그래서 우선 순위를 정할 수 있다. q값에 0~1 사이의 우선 순위를 부여한다. 클 수록 우선순위가 높으며 생략하면 1이다.

예시는 

1. ko-KR;q=1 (q 생략)
2. ko;q=0.9
3. en-US;q=0.8 
4. en:q=0.7

와 같은 우선 순위를 가진다.

![](../../.gitbook/assets/kimyounghan-http-web-basic/07/screenshot%202021-04-04%20오후%205.38.18.png)

우선 순위를 주면 한국어 다음으로 영어를 원한다는 것을 서버가 파악하고 영어로 보내준다.

![](../../.gitbook/assets/kimyounghan-http-web-basic/07/screenshot%202021-04-04%20오후%205.38.23.png)

내용은 구체적인 것이 우선한다.

1. text/plain;format=flowed
2. text/plain
3. text/*
4. */*

이 순서로 매칭이 된다.