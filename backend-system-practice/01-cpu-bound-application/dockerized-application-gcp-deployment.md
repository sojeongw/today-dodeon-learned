# Dockerized 애플리케이션 GCP에 배포하기

1. 로컬 작업
    1. 도커 파일을 작성한다.
    2. 도커 이미지를 build한다.
    3. docker hub에 push 한다.
2. GCP instance 작업
    1. docker hub에 push한 이미지를 pull 한다
    2. run 명령어를 실행한다.
    3. 도커 이미지가 컨테이너가 되어 애플리케이션을 실행한다.

## 도커 파일 작성

[Spring Boot with Docker](https://spring.io/guides/gs/spring-boot-docker/)

```dockerfile
FROM openjdk:8-jdk-alpine
ARG JAR_FILE=target/*.jar
COPY ${JAR_FILE} app.jar
ENTRYPOINT ["java","-jar","/app.jar"]
```

- 튜토리얼에 있는 DockerFile을 생성한다.

## 도커 파일 빌드

[Docker Hub](https://hub.docker.com/)

```commandline
docker build -t {사용자 이름}/{저장소 이름} .
docker build -t user-name/spring-boot-cpu-bound .
```

- 도커 허브에 가입 후 도커 데스크탑이 실행된 상태에서 빌드 명령을 입력한다.

```commandline
sudo docker run -p 80:80 user-name/spring-boot-cpu-bound
```

```text
http://localhost/hash/123
```

- 로컬에서 시험 삼아 띄워본다.
    - 이전에는 jar 파일을 직접 실행했다.
    - 이번에는 도커 이미지에 있는 jar 파일을 실행한 것이다.
- 우리는 프로젝트에 80 포트를 사용하도록 설정했으므로 80 포트로 run 한다.

## 도커 허브에 업로드

```commandline
docker push user-name/spring-boot-cpu-bound 
```

```commandline
docker login
```

- push 명령어를 실행한다.
    - denied가 뜨면 도커 로그인을 해준다.

![](../../.gitbook/assets/backend-system-practice/01/screenshot%202022-04-03%20오후%207.12.26.png)

- 도커 허브에 다시 들어가보면 push 했던 이미지가 보인다.

## GCP 인스턴스에 도커 설치

```commandline
sudo yum install docker
```

- VM 인스턴스에 도커를 설치한다.

```commandline
sudo systemctl start docker
```

- 도커가 실행된다.

## docker hub에 있는 이미지 pull

```commandline
sudo docker pull user-name/spring-boot-cpu-bound
```

- pull 받는다.
- 꼭 sudo를 붙여야 받을 수 있다.

```commandline
sudo docker run -p 80:80 user-name/spring-boot-cpu-bound
```

- 외부의 80번 포트와 내부의 80번 포트를 연결한 뒤 run 한다.

```text
http://00.00.000.00/hash/123
```

- 정상 작동 하는지 테스트 한다.

## 성능 테스트

- 이전에 만든 artillery 스크립트에 target ip를 넣고 실행한다.