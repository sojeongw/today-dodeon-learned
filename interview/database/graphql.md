# GraphQL

![](../../.gitbook/assets/interview/database/graphql-pipeline.png)

GraphQL은 페이스북에서 만든 쿼리 언어이다.

## SQL과의 차이점

|SQL|GQL|
|:---:|:---:|
|DB 시스템에 저장된 데이터를 효율적으로 가져오기 위함|웹 클라이언트가 데이터를 서버로부터 효율적으로 가져오기 위함|
|백엔드 시스템에서 쿼리문을 작성하고 호출함|클라이언트 시스템에서 작성하고 호출함|

### SQL 쿼리 예시

```mysql
SELECT plot_id, species_id, sex, weight, ROUND(weight / 1000.0, 2) FROM surveys;
```
### GraphQL 쿼리 예시

```postgresql
{
  hero {
    name
    friends {
      name
    }
  }
}
```

## 특징

- 애플리케이션이 GQL 쿼리를 입력받고, 처리한 결과를 다시 클라이언트에 돌려준다. 
- 특정 DB나 플랫폼, 언어, 네트워크 방법에 종속적이지 않다. 마치 HTTP API처럼.
    - 따라서 일반적으로는 네트워크 레이어이 L7의 HTTP POST 메서드와 웹소켓 프로토콜을 활용하지만 L4의 TCP/UDP나 L2의 이더넷을 사용할 수도 있다.

## REST API와의 차이점

|REST API|GraphQL|
|:---:|:---:|
|URL과 Method를 조합해 다양한 Endpoint를 만들 수 있음|단 하나의 Endpoint만 존재함|
|Endpoint마다 SQL 쿼리가 달라짐|GQL의 스키마 타입마다 쿼리가 달라짐|

REST API는 어떤 기능이 필요할 때마다 새로운 API를 추가해야 한다. 예를 들어 `Account` 모델이 있고 `/accounts`라는 endpoint가 있다면 하나의 정보를 가져오려면 `/account/{id}`라는 새로운 API를 만들어야 한다.

이러다보면 애플리케이션 규모가 커졌을 때 수 많은 endpoint가 생성되어 유지보수가 힘들다. 반면 GraphQL은 클라이언트에서 쿼리를 만들어 보내면 원하는대로 서버에서 결과를 반환해준다.

{% tabs %}
{% tab title="Request" %}
```roomsql
query {
    account(id: "1") {
        username
        email
        firstName
        lastName
        friends {
            firstName
            username
        }
    }
}
```
{% endtab %}

{% tab title="Response" %}
```roomsql
{
  "data": {
    "account": {
      "username": "velopert",
      "email": "public.velopert@gmail.com",
      "firstName": "Minjun",
      "lastName": "Kim",
      "friends": [
        {
          "firstName": "Jayna",
          "username": "jn4kim"
        },
        {
          "firstName": "Abet",
          "username": "abet"
        }
      ]
    }
  }
}
```
{% endtab %}
{% endtabs %}

필요한 정보만 쿼리로 만들어서 서버에 전달하면 서버가 알아서 주어진 틀대로 데이터를 보내준다.

![](../../.gitbook/assets/interview/database/graphql-stack.png)

![](../../.gitbook/assets/interview/database/graphql-mobile-api.png)

위의 그림처럼, GQL API를 사용하면 네트워크를 여러 번 호출하지 않고 한 번에 처리할 수 있다.

## Mutation

서버에 저장된 데이터를 수정하는 작업을 GraphQL에서는 `Mutation`이라고 한다. 뮤테이션에는 3가지 종류가 있다.

- Create: 새로운 데이터 생성
- Update: 기존 데이터 수정
- Delete: 기존 데이터 삭제

뮤테이션은 쿼리문과 문법 구조는 같지만 `mutation` 키워드로 시작해야 한다. 아래는 `Person`을 생성하는 예시다.


{% tabs %}
{% tab title="Request" %}
```roomsql
mutation {
  createPerson(name: "Bob", age: 36) {
    name
    age
  }
}
```
```
{% endtab %}

{% tab title="Response" %}
```json
"createPerson" {
  "name": "Bob",
  "age": 36,
}
```
{% endtab %}
{% endtabs %}




[GraphQL 개념잡기](https://tech.kakao.com/2019/08/01/graphql-basic/)
[How to GraphQL](https://velog.io/@cadenzah/graphql-03-core)