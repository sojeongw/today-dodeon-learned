# Backend

## Project Setup

## A word on @types

타입을 가지고 있는 라이브러리. 모든 라이브러리가 타입을 가지고 있지는 않아서 따로 설치해야 한다.

타입이 있으면 맞고 틀리고를 바로 알 수 있는 장점이 있다.

## GraphQL Yoga and Express

### GraphQL Yoga
create-react-app처럼 gql 환경을 만들어준다.

### helmet, morgan, cors

이들은 미들웨어다.

미들웨어는 앱의 연결이나 요청을 다루는 방식을 수정하는 것이다. API로 무언가를 요청할 때마다 미들웨어가 요청을 가로채서 기록하고 다음 단계로 진행한다.

### Express

graphql-yoga 내부의 서버 부분. node.js의 서버 프레임워크다.

### makeExecutableSchemas

allTypes가 하는 것처럼 schema를 하나로 합쳐준다.