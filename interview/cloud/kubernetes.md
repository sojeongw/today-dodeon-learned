# Kubernetes

- 다수의 컨테이너 오케스트레이션을 제공하는 서비스

컨테이너는 프로세스/애플리케이션 끼리, 혹은 기반 시스템으로부터 격리하기 위해 만들어졌다. 그래서 개별 컨테이너를 만들고 배치하는 것은 쉽다.

하지만 DB와 웹 프론트엔드, 백엔드 등 여러 컨테이너를 모아서 한 단위로 관리하는 큰 애플리케이션을 만드는 것은 어렵다. 만약 여기에 컨테이너 각각을 독립적으로 연결, 관리, 확장해야 한다면 이 모든 걸 관장하는 방법이 필요하다.

쿠버네티스는 여러 컨테이너 애플리케이션을 여러 호스트에 배치하고 관리하는 작업을 자동화해준다. 각각의 컨테이너를 직접 관리할 필요가 없다.

소수의 사용자만 사용하는 단순한 컨테이너 앱은 보통 오케스트레이션이 필요 없으므로 쿠버네티스를 사용할 필요가 없다. 

## 사용 사례
### 복잡한 서비스

두 개 이상의 컨테이너를 사용하는 앱이라면 오케스트레이션이 필요하다. 하지만 크기가 크지 않다면 쿠버네티스 보다는 도커 스웜 같은 좀 더 최소화된 솔루션이 낫다.

### 확장성과 복구가 중요한 서비스

쿠버네티스는 조건이 바뀔 때마다 코딩 대신 원하는 시스템 상태를 서술하는 방식으로 [워크로드](https://zetawiki.com/wiki/워크로드)와 컨테이너 간의 균형을 맞출 수 있다.

### 현대적인 CI/CD를 적용할 때

오케스트레이션은 블루/그린 배치나 롤링 업그레이드(클러스터의 노드를 하나씩 업그레이드)를 위한 배치 패턴을 지원한다.