# 프로젝트 환경 설정

## Querydsl 설정

```groovy
buildscript {
    ext {
        queryDslVersion = "5.0.0"
    }
}

plugins {
    id 'org.springframework.boot' version '2.7.0'
    id 'io.spring.dependency-management' version '1.0.11.RELEASE'
    // querydsl 추가
    id "com.ewerk.gradle.plugins.querydsl" version "1.0.10"
    id 'java'
}

group = 'study'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '11'

configurations {
    compileOnly {
        extendsFrom annotationProcessor
    }
}

repositories {
    mavenCentral()
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    implementation 'org.springframework.boot:spring-boot-starter-web'
    compileOnly 'org.projectlombok:lombok'
    runtimeOnly 'com.h2database:h2'
    annotationProcessor 'org.projectlombok:lombok'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
    // querydsl 추가
    implementation "com.querydsl:querydsl-jpa:${queryDslVersion}"
    annotationProcessor "com.querydsl:querydsl-apt:${queryDslVersion}"
}

tasks.named('test') {
    useJUnitPlatform()
}

// querydsl 추가 시작
// Q클래스를 생성하면 저장하는 위치
def querydslDir = "$buildDir/generated/querydsl"
querydsl {
    jpa = true
    querydslSourcesDir = querydslDir
}
sourceSets {
    main.java.srcDir querydslDir
}
configurations {
    querydsl.extendsFrom compileClasspath
}
compileQuerydsl {
    options.annotationProcessorPath = configurations.querydsl
}
// querydsl 추가 끝
```

## Q 클래스 생성

- gradle - other - compileQuerydsl 실행

```java
/**
 * QHello is a Querydsl query type for Hello
 */
@Generated("com.querydsl.codegen.DefaultEntitySerializer")
public class QHello extends EntityPathBase<Hello> {

    private static final long serialVersionUID = 1910216155L;

    public static final QHello hello = new QHello("hello");

    public final NumberPath<Long> id = createNumber("id", Long.class);

    public QHello(String variable) {
        super(Hello.class, forVariable(variable));
    }

    public QHello(Path<? extends Hello> path) {
        super(path.getType(), path.getMetadata());
    }

    public QHello(PathMetadata metadata) {
        super(Hello.class, metadata);
    }

}
```

- Gradle - Tasks - build - clean
- Gradle - Tasks - other - compileQuerydsl
- Q 타입 파일이 생성되어 있어야 한다.
    - Q 타입은 컴파일 시점에 자동 생성되므로 git에 포함하지 않는 것이 좋다.

## 동작 확인

```java

@SpringBootTest
@Transactional
class KimyounghanQuerydslApplicationTests {

    @Autowired
    EntityManager em;

    @Test
    void contextLoads() {
        Hello hello = new Hello();
        em.persist(hello);

        JPAQueryFactory query = new JPAQueryFactory(em);
        // Querydsl Q타입 동작 확인
        QHello qHello = QHello.hello;

        Hello result = query
                .selectFrom(qHello)
                .fetchOne();

        Assertions.assertThat(result).isEqualTo(hello);
        // lombok 동작 확인 (hello.getId())
        Assertions.assertThat(result.getId()).isEqualTo(hello.getId());
    }
}
```

## 라이브러리 살펴보기

```commandline
./gradlew dependencies --configuration compileClasspath
```

- querydsl-apt
    - Querydsl 관련 코드 생성
- querydsl-jpa
    - 실제 querydsl 라이브러리