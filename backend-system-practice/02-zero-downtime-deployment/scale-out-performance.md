# 서버를 늘려서 성능 측정하기

- 로드밸런싱 된 CPU bound 애플리케이션을 스트레스 테스트 한다.
- 스트레스 테스트 도중 배포해 무중단 배포가 되는지 확인한다.

## 인스턴스가 3개일 때

```yaml
config:
  target: "http://{로드밸런서 ip}"
  phases:
    - duration: 360
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
artillery report report.json
```

- arrivalRate를 1 -> 8 -> 16으로 바꿔가며 테스트한다.
- 16정도 되면 에러 레이트가 급격하게 늘어난다.
- 500 error
    - CPU 바운드 애플리케이션 성능이 요청에 대한 응답을 할 수 없는 상태
- 502 error
    - CPU 바운드 애플리케이션이 500 error를 계속 던지다가 던질 힘조차 없을 때
    - hang 상태에 빠진다고 얘기한다.
    - nginx는 인스턴스가 더 이상 요청을 받을 수 없다고 판단하고 연결을 끊어버린다.

## 인스턴스를 하나씩 내렸을 때

- arrivalRate = 8일때 안정적이란 것을 알았으니 이제 