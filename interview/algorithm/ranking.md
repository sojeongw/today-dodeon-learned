# 랭킹 알고리즘

매칭 알고리즘으로 찾은 페이지에 순서를 정하는 작업이다. 선택된 페이지를 대상으로 어떤 페이지가 고객의 요구에 더 부합하는 것인지 결정한다.

## 하이퍼링크 알고리즘

특정 페이지에 링크가 많이 걸려 있다면 고객의 요구에 부합할 가능성이 높은 것으로 판단해 앞에 위치시키는 방법이다.

![](../../.gitbook/assets/interview/algorithm/screenshot%202020-02-20%20오전%208.42.35.png)

민호와 주연의 레시피가 각각 있을 때 다른 사람의 웹 페에지에서 각 페이지에 대한 언급을 위와 같이 했다고 가정하자.

하이퍼링크 알고리즘은 주연의 레시피를 검색 결과 상위에 위치시킨다. 민호는 링크를 1개만 갖고 있지만 주연은 3개의 링크를 갖고 있기 때문이다. 즉, 링크를 더 많이 가진 문서가 더 유용하다고 판단한다.

## 단점

링크가 나쁜 페이지를 지칭하는 데에 사용될 수도 있다. 만약 '민호의 레시피는 맛이 없다'는 링크가 있다면 부정적 의미이기 때문에 추천할 때 고려되지 않아야 한다. 하지만 하이퍼링크 알고리즘은 이런 점을 고려하지 않는다.

## 권위 링크 알고리즘

![](../../.gitbook/assets/interview/algorithm/screenshot%202020-02-13%20오후%202.32.59.png)

민호의 레시피를 링크한 조현희의 홈페이지를 다른 2개의 페이지가 참고하고 있다. 이 경우 민호의 레시피는 2점이다.

반면 주연의 레시피 링크가 있는 주현익의 홈페이지는 다른 8개의 페이지가 조현익의 페이지를 참고하고 있다. 따라서 주연의 레시피는 8점을 가진다.

이처럼 민호와 주연은 각각 1개의 링크를 가지지만 링크를 제공하는 조현희, 조현익 페이지가 가지는 가중치에 따라 점수를 부여한다. 이런 방식은 부정적인 연결을 고려할 수 있어 하이퍼링크 알고리즘의 문제를 해결할 수 있다.

## 장점

링크에 가중치를 부여해 실제 적용 시 개선된 결과를 볼 수 있다.

## 단점

사이클이 형성되는 경우에는 특정 페이지의 점수가 계속 올라간다.

예를 들어, 조현익의 홈페이지가 어떤 페이지를 참고할 때 그 페이지가 조현익의 페이지를 참고한다면 서로의 점수가 계속 올라가게 된다.

## 무작위 서퍼 알고리즘

현재 구성된 페이지 사이 링크를 따라 서핑을 수행하고 이를 기반으로 방문 빈도를 따지는 것이다. 구글 검색의 근간을 이루는 원리다.

![](../../.gitbook/assets/interview/algorithm/screenshot%202020-02-20%20오전%209.05.56.png)

16개의 페이지가 위와 같이 연결되어있고 연결된 페이지 중 하나를 선택하는 작업을 1000번 반복한다고 가정하자.

연결된 페이지가 아닌 임의의 페이지로 이동하는 동작도 포함한다. B에서 C로 가는 화살표가 그렇다.

각 페이지별 방문한 횟수를 기반으로 확률을 계산하면 다음과 같다.

- B는 여러 문서와 연결되어 있고(하이퍼링크 트릭) 참고하는 문서의 중요성이 높아서(권위 트릭) 10%의 높은 확률이 나온다.
- A는 참고하는 문서가 하나이지만 14%라는 높은 확률을 가지므로 방문할 확률이 높게 나온다(권위 링크 트릭).
- C는 A처럼 참고하는 문서가 하나 밖에 없고 낮은 확률을 가지므로 방문할 확률이 낮다(권위 링크 트릭).
- D는 참고하는 문서가 많으므로 방문할 확률이 높다(하이퍼링크 트릭).

