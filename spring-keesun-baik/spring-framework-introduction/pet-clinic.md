# 스프링 예제 프로젝트 PetClinic

## 프로젝트 세팅

```shell script
./mvnw package   # 프로젝트 빌드
java -jar target/*.jar    # 프로젝트 실행
```

Maven의 `package`라는 명령어를 실행하면 빌드해서 패키지 파일을 만든다. 설정 파일에 아무것도 하지 않았으면 기본적으로 `jar`로 생성된다.

## 또 다른 실행 방법

src.main.java에 있는 `PetClinicApplication`을 실행하면 된다. 이때 반드시 패키징을 먼저 해놔야 한다. 패키징 하는 과정에서 프론트 관련 라이브러리를 생성하는 플러그인이 동작하기 때문이다. 그래서 패키징을 하지 않으면 화면이 깨져 나온다.