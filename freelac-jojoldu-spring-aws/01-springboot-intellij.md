# IntelliJ로 스프링부트 시작하기

## IntelliJ 소개

- 강력한 Smart Completion 기능
- 다양한 리팩토링과 디버깅 기능
- 이클립스 Git에 비해 훨씬 높은 자유도
- 프로젝트 시작 시 인덱싱으로 파일에 대한 빠른 검색 가능
- HTML, CSS, JS, XML 등에 대한 강력한 기능 지원
- 자바, 스프링부트 버전에 맞춘 빠른 업데이트

![](../.gitbook/assets/freelac-jojoldu-spring-aws/01/스크린샷%202020-07-19%20오후%208.27.52.png)

Maximum Heap Size가 750MB로 되어있는데, 이 설정은 IntelliJ를 실행할 때 얼마나 메모리를 할당할지 결정하는 값이다.

이 기본값은 PC의 메모리가 4G 이하일 때를 가정하고 설정된 값이므로 8G라면 1024~2048, 16G라면 2048~4096을 선택하면 된다. 

## IntelliJ 프로젝트 생성하기

IntelliJ에는 Eclipse의 Workspace 개념이 없다. Project와 Module 개념만 존재한다. 따라서 모든 프로젝트를 한 번에 불러올 수 없다. 한 화면에 한 프로젝트만 열 수 있다.

*Reference*

[IntelliJ의 Project](http://bit.ly/2orXeGl)

## Gradle 프로젝트를 스프링부트 프로젝트로 변경하기

{% tabs %}
{% tab title="build.gradle" %}
```groovy
plugins{
    id 'java'
}

group 'com.jojoldu.book'
version '1.0-SNAOSHOT'

sourceCompatibility = 1.8

repositories {
    mavenCentral()
}

dependencies {
    testCompile group: 'junit', name: 'junit', version: '4.12'
}
```
{% endtab %}
{% endtabs %}

처음에 gradle 프로젝트를 생성하면 이렇게 간단한 코드만 들어있다.여기에 스프링부트에 필요한 요소를 하나씩 추가할 것이다.

스프링 이니셜라이저를 통해 진행하면 간편하지만 `build.gradle`의 코드가 무슨 역할을 하는지, 추가로 의존성을 설정해야 할 때 어떻게 해야할지 모르게 된다. 따라서 완전 처음이라면 하나씩 코드를 작성하길 추천한다.

{% tabs %}
{% tab title="build.gradle" %}
```groovy
buildscript {
    // 전역 변수를 사용하겠다는 의미
    ext {
        springBootVersion = '2.1.7.RELEASE'
    }
    repositories {
        mavenCentral()
        jcenter()
    }
    dependencies {
        // springBootVersion라는 전역변수의 값을 플러그인의 의존성으로 받는다.
        classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")
    }
}

// 앞에 선언한 플로그인 의존성을 적용할 것인지 결정하는 코드
apply plugin: 'java'
apply plugin: 'eclipse'
apply plugin: 'org.springframework.boot'
apply plugin: 'io.spring.dependency-management' // 스프링부트 의존성을 관리해주는 플러그인이므로 꼭 추가해야 한다.

group = 'com.jojoldu'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '1.8'

configurations {
    compileOnly {
        extendsFrom annotationProcessor
    }
}

// 각종 의존성(라이브러리)를 어떤 원격 저장소에서 받을지 결정한다.
repositories {
    mavenCentral()
    jcenter()
}

// 프로젝트 개발에 필요한 의존성을 선언한다.
dependencies {
    compile('org.projectlombok:lombok')
    compile('org.springframework.boot:spring-boot-starter-web')
    testCompile('org.springframework.boot:spring-boot-starter-test')
}

test {
    useJUnitPlatform()
}

```
{% endtab %}
{% endtabs %}

#### repositories 

각종 의존성(라이브러리)를 어떤 원격 저장소에서 받을지 결정한다.

의존성 저장소인 mavenCentral은 이전부터 많이 사용되어 왔지만, 본인이 만든 라이브러리를 업로드하기 위해 많은 과정과 설정이 필요하다. 그러다보니 점점 공유가 안 되는 상황이 발생했고 이런 문제점을 개선한 jcenter가 각광받게 되었다. 게다가 jcenter에 업로드 하면 mavenCentral에도 자동으로 업로드되기 때문에 점점 jcenter로 이동하는 추세다. 여기서는 둘 다 쓰도록 설정했다.

#### dependencies

프로젝트 개발에 필요한 의존성을 선언한다.

IntelliJ는 메이븐 저장소의 데이터를 인덱싱해서 관리하기 때문에 `ctrl` + `space`로 의존성 자동데 완성이 가능하다.

의존성 코드는 직접 작성해도 되고 자동 완성으로 만들어도 되지만 특정 버전을 명시해서는 안된다. 그래야만 맨 위에서 `${springBootVersion}`의 버전을 따라가게 된다.