# 프로젝트 생성

- 프로젝트 선택
    - Project: Gradle Project
    - Language: Java
    - Spring Boot: 2.4.x
- Project Metadata
    - Group: hello
    - Artifact: springmvc
    - Name: springmvc
    - Package name: hello.springmvc
    - Packaging: **Jar**
        - WAR
            - 톰캣같은 WAS 서버를 별도로 설치하고 거기에 빌드된 파일을 넣을 때 사용한다.
            - JSP를 쓸 때도 사용한다.
        - JAR
            - 그냥 내장 톰캣을 쓴다면 선택한다.
            - 내장 서버에 최적화되어 webapp 경로도 사용하지 않는다.
            - 최근에 주로 사용하는 방식이다.
    - Java: 11

