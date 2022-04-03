# 스트레스 테스트 툴로 성능 측정하기

- API 성능을 측정하려면 목표 latency를 잡아야 한다.
- 목표 latency를 충족하기 위해 여러 값을 돌려보면서 스케일 업을 하거나 nginx를 두는 등 해결책을 생각해본다.

```yaml
config:
  target: "http://00.00.00.00"
  phases:
    # 부하 테스트를 진행할 시간
    - duration: 60
      # 부하 테스트에 생성할 virtual user 수
      arrivalRate: 1
      name: Warm up
scenarios:
  - name: "just get hash"
    flow:
      - get:
          url: "/hash/123"
```

```commandline
artillery run --output report.json cpu-test.yaml
```

부하 테스트를 실행한다.

```commandline
artillery report report.json
```

테스트 결과를 html로 만든다.

![](../../.gitbook/assets/backend-system-practice/01/screenshot%202022-04-03%20오후%206.36.01.png)

- arrivalRate: 1

![](../../.gitbook/assets/backend-system-practice/01/screenshot%202022-04-03%20오후%206.38.01.png)

- arrivalRate: 8

## 염두에 둬야 할 것

- 예상 TPS보다 여유롭게 성능 목표치를 잡는다.
    - 예상이 1000 TPS라면 트래픽이 몰릴 걸 예상해 최소 3~4천 이상으로 생각하고 인스턴스를 구성한다.
- 기대 latency를 만족할 때까지 성능을 테스트한다.
    - 먼저 단일 요청에 대한 latency를 확인한다.
    - 기대치보다 높다면 스케일 아웃으로 해결되지 않는다.
    - 코드가 비효율적이거나 해당 API에서 I/O가 병목인 경우가 많다.
    - 네트워크에서 latency가 발생하는 경우도 있다.
- 스케일 아웃으로 해결되지 않으면 여러 방면으로 병목을 의심해보자.