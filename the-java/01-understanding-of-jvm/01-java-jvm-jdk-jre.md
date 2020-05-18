# 자바, JVM, JDK, JRE

![](../../.gitbook/assets/the-java/01/스크린샷%202020-07-05%20오후%207.38.04.png)

## JVM(Java Virtual Machine)

자바 바이트 코드를 어떻게 실행할지에 대한 표준이다.

- 자바 가상 머신으로 자바 바이트 코드인 `.class` 파일을 OS에 특화된 코드로 변환해 실행
    - 변환 시에는 인터프리터와 JIT 컴파일러가 동작
- 바이트 코드를 실행하는 표준이자 특정 벤더가 구현한 구현체
- 특정 플랫폼에 종속적임 

### 자바 바이트 코드

![](../../.gitbook/assets/the-java/01/스크린샷%202020-07-05%20오후%208.06.16.png)

**Reference**

[JIT 컴파일러](https://aboullaite.me/understanding-jit-compiler-just-in-time-compiler)

### JVM 벤더

오라클, 아마존, Azul 등

**Reference**

[JVM 스펙](https://docs.oracle.com/javase/specs/jvms/se11/html)

## JRE(Java Runtime Environment)

JVM + 라이브러리

- 자바 애플리케이션을 실행할 수 있도록 구성된 배포판

### 구성

- JVM
- 핵심 라이브러리
- 자바 런타임 환경에서 사용하는 프로퍼티 세팅
- 리소스 파일

컴파일러 등 개발과 관련된 요소들은 JDK에서 제공하므로 포함하지 않는다.

## JDK(Java Development Kit)

JRE + 개발 툴

- 소스 코드를 작성할 때부터 자바는 플랫폼에 독립적임
- 오라클은 자바 11부터 JDK만 제공하고 JRE를 따로 제공하지 않음
- Write Once Run Anywhere

## 자바

프로그래밍 언어

- JDK에 들어있는 자바 컴파일러인 `javac`를 이용해 바이트코드인 `.class`로 컴파일 함
- 오라클 JDK 11 버전부터 상용으로 사용 시 유료
    - 오라클 Open JDK 혹은 오라클 외의 회사의 JDK 라면 무료
    
**Reference**

[자바 유료화](https://medium.com/@javachampions/java-is-still-free-c02aef8c9e04)

## JVM 언어

JVM 기반으로 동작하는 프로그래밍 언어

- 클로저
- 그루비
- JRuby
- Jython
- Kotiln
- Scala

**Reference**

[JDK, JRE, JVM](https://howtodoinjava.com/java/basics/jdk-jre-jvm/)

[JVM 언어](https://en.wikipedia.org/wiki/List_of_JVM_languages)