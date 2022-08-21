# 독립적으로 실행 가능한 JAR

```shell
mvn package -DskipTests
```

- 배포하거나 이미지를 만들려면 JAR 패키지로 만들어야 한다.
- target 패키지를 보면 JAR 파일이 생성된다.
- JAR 파일 하나로 애플리케이션이 잘 돌아간다.
- 그 수많은 의존성은 다 어떻게 되는걸까?
    - JAR 안에 모든 의존성이 다 들어가있다.

그런데 자바에는 jar를 읽을 수 있는 표준적인 방법이 없다. 옛날에는 모든 걸 다 합쳐 압축해서 뭐가 뭔지 알 수가 없었다.

## 스프링 부트의 전략

- 내장 JAR
    - JAR 안에 JAR를 묶어놓고 읽을 수 있게 만들어 놓는다.
- 애플리케이션 클래스와 라이브러리의 위치를 구분한다.
- org.springframework.boot.loader.jar.JarFile
    - 내장 JAR를 읽는다.
- org.springframework.boot.loader.Launcher
    - JAR 파일을 실행한다.