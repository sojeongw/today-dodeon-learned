# JUnit 시작하기

## JUniit

자바 개발자가 가장 많이 사용하는 테스팅 프레임워크. 자바 8 이상을 필요로 한다.

- 대체제
    - TestNG, Spock...etc
    
![](study/today-dodeon-learned/.gitbook/assets/keesunbaik-the-java-test/01/스크린샷%202020-07-16%20오전%2010.29.53.png)

### Platform

테스트를 실행하는 런처와 TestEngine API를 제공한다.

#### 런처

JUnit 5으로 작성된 코드를 실행한다. 원래 자바는 main 메서드가 있어야 실행이 가능한데, 없어도 테스트 코드를 실행하도록 도와준다.

### Jupiter

TestEngine API 구현체로, JUnit 5를 제공한다.

### Vintage

JUnit 3, 4를 지원하는 TestEngine 구현체이다.

**Reference**

[JUnit User Guide](https://junit.org/junit5/docs/current/user-guide/)

## JUnit 시작하기

스프링부터 2.2 이상에서는 기본으로 JUnit 5로 의존성이 추가된다. 

### @Test

- `public`을 붙이지 않아도 테스트가 가능하다.
    - JUnit 5부터는 어차피 JUnit이 reflection을 사용해 접근지정자에 상관없이 접근할 수 있기 때문이다.

### @BeforeAll, @AfterAll

- 테스트 클래스 안에 있는 모든 테스트가 실행될 때 딱 한 번만 호출된다.
- `private`이 아니면서 `static`이고 리턴값이 없는 메서드만 사용해야 한다.
- 메서드 이름은 상관없다.

### @BeforeEach, @BeforeAfter

- 각각의 테스트를 실행하기 이전, 이후에 실행된다.

```java
class StudyTest {

    @Test
    void a() {
        System.out.println("a");
    }

    @Test
    void b() {
        System.out.println("b");
    }

    @BeforeAll
    static void beforeAll() {
        System.out.println("before all");
    }

    @AfterAll
    static void afterAll() {
        System.out.println("after all");
    }

    @BeforeEach
    void beforeEach() {
        System.out.println("before each");
    }

    @AfterEach
    void afterEach() {
        System.out.println("after each");
    }

}
```

![](study/today-dodeon-learned/.gitbook/assets/keesunbaik-the-java-test/01/스크린샷%202020-07-16%20오전%2011.06.13.png)

### @Disabled

실행하고 싶지 않은 테스트에 마킹하는 애너테이션이다.

```java
class StudyTest {

    @Test
    void a() {
        System.out.println("a");
    }

    @Test
    @Disabled
    void b() {
        System.out.println("b");
    }

    @BeforeAll
    static void beforeAll() {
        System.out.println("before all");
    }

    @AfterAll
    static void afterAll() {
        System.out.println("after all");
    }

    @BeforeEach
    void beforeEach() {
        System.out.println("before each");
    }

    @AfterEach
    void afterEach() {
        System.out.println("after each");
    }

}
```

![](study/today-dodeon-learned/.gitbook/assets/keesunbaik-the-java-test/01/스크린샷%202020-07-16%20오전%2011.13.15.png)

### @DisplayNameGeneration

- Method와 Class 레퍼런스를 사용해서 테스트 이름을 설정한다.
- 보통 테스트 메서드 이름을 언더바로 표기하는데, 자신이 원하는 문자로 표기하고 싶을 때 사용한다.
- 기본 구현체로 `ReplaceUnderscores`를 제공한다.
    - `@DisplayNameGeneration(DisplayNameGenerator.ReplaceUnderscores.class)`
    - 메서드 이름에 언더 스코어가 들어가면 공백으로 치환해준다.
    
```java
@DisplayNameGeneration(DisplayNameGenerator.ReplaceUnderscores.class)
class StudyTest {

}
```


### @DisplayName

- 어떤 테스트인지 테스트 이름을 보다 쉽게 표현할 수 있도록 해준다.
- 이모지도 넣을 수 있다.
- `@DisplayNameGeneration`보다 우선 순위가 높다.


```java
class StudyTest {
    @Test
    @DisplayName("스터디 만들기")
    void a() {
        Study study = new Study();
        assertNotNull(study);
        System.out.println("a");
    }
}
```

**Reference**

[Display Name](https://junit.org/junit5/docs/current/user-guide/#writing-tests-display-names)