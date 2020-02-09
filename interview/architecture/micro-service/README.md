# MSA

![](../../../.gitbook/assets/pic1.png)

MSA는 Micro Service Architecture의 준말이다. Micro 라는 말이 붙었듯, 하나의 커다란 애플리케이션을 여러 개의 작은 애플리케이션으로 쪼개서 조합하도록 만든 아키텍처다. 레고처럼 블럭 하나 하나를 붙여 큰 결과물을 만드는 것이라고 생각하면 된다.

## 등장 배경

### Monolith 아키텍처

모놀리스는 모든 내용이 한 프로젝트에 통합되어있는 형태다. 아키텍처가 간단하고 유지 보수가 쉽기 때문에 소규모의 프로젝트에는 적합하다.

하지만 규모가 커지고 수 십, 수 백명의 개발자가 달라붙어야 하는 상황이라면 여러가지 문제가 발생한다.

* 서비스가 커질 수록 전체 시스템의 구조와 영향도를 파악하기가 힘들다.
* 빌드, 테스트, 배포 하는 시간이 기하급수적으로 늘어난다.
* 서비스를 부분적으로 스케일 아웃\(수평적 확장\) 하기 힘들다.
* 한 부분에서 난 장애가 전체 서비스에 영향을 미칠 수 있다.

## 정의

```text
the microservice architectural style is an approach to developing a single application as a suite of small services, each running in its own process and communicating with lightweight mechanisms, often an HTTP resource API. These services are built around business capabilities and independently deployable by fully automated deployment machinery.

- Martin Fowler
```

`각자의 프로세스에서 돌아가는 작은 서비스들의 모음집`이며 `독립적인 배포가 가능한` 구조다. 서비스가 많아서 배포가 잦기 때문에 `자동화`된 배포가 필요하다.

* 각각으로 나뉜 서비스는 크기가 작아졌을 뿐 그 자체는 하나의 모놀리틱 아키텍처와 유사하다.
* 각 서비스는 독립적으로 배포할 수 있어야 한다.
* 다른 서비스에 대한 의존도는 최대한 작아야 한다.
* 개별 프로세스로 구동되어야 한다.
* 서로 REST와 같은 가벼운 방식으로 통신해야 한다.

그럼 하나의 마이크로 서비스는 각자의 비즈니스와 시스템 설정에 적합한 범위를 설정하는 것이 중요하다.

## 장점

### 빠른 배포

* 서비스 별로 각자 배포가 가능해서 전체 서비스를 중단하지 않고도 배포할 수 있다.
* 요구 사항이 있을 때 빠르게 반영해서 배포할 수 있다.

### 확장의 용이성

* 서비스 확장을 좀 더 쉽게 할 수 있다.
* 클라우드 사용에 적합한 구조다.

### 장애 컨트롤

* 부분적인 장애가 전체 서비스로 퍼져나갈 가능성이 적다.
* 따라서 부분적으로 발생한 장애를 따로 격리하기가 수월하다.

## 단점

### 성능 이슈

* 모노리스보다 복잡한 아키텍처이므로 전체 서비스가 커질 수록 복잡도도 기하급수적으로 커진다.
* 서비스끼리 API로 호출하기 때문에 통신 비용과 Latency가 늘어난다.
* 서로 분리되어 있기 때문에 테스트와 트랜잭션을 수행하는 것이 복잡하다.
* 데이터가 여러 서비스에 걸쳐서 분산되어 있기 때문에 한 번에 조회하기 어렵다.
* 데이터 정합성\(데이터 값이 서로 일치하는 \) 관리가 어렵다.

[Microservices](https://martinfowler.com/articles/microservices.html)

