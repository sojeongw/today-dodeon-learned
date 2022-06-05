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
    - 애플리케이션 자체가 종료된다.
    - hang 상태에 빠진다고 얘기한다.
    - nginx는 인스턴스가 더 이상 요청을 받을 수 없다고 판단하고 연결을 끊어버린다.
- arrivalRate = 8일때 안정적이란 결론을 내릴 수 있다.

## 인스턴스를 하나씩 내렸을 때

```commandline
docker ps | grep {image name}
docker ps | grep spring-boot-cpu-bound
```

```commandline
docker container kill -s 15 {container id}
```

- 이제 인스턴스 2, 3을 내리고 1만 있을 때 얼마나 버티는지 확인한다.
- `-s 15`
    - 컨테이너가 스스로 graceful 하게 종료할 수 있다.
- graceful
    - 애플리케이션이 스스로 실행 중인 로직을 전부 다 처리한 다음에 종료한다.
- 테스트 하면 3개를 띄웠을 때보다 에러가 많이 발생한다.

### 인스턴스 복원

- 젠킨스 - build now
    - 인스턴스 2, 3에 다시 도커를 띄우고 원복한다.

## 배포 중일 떄

- 배포를 하는 상황처럼 애플리케이션을 하나씩 내렸다가 올린다.
    - 스트레스 테스트를 돌려놓고 인스턴스를 한 개만 남기고 하나씩 내려본다.
- 정상적으로 200을 받는 걸 확인한다.