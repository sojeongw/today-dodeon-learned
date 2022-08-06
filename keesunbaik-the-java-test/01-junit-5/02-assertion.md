# JUnit 5 Assertion

## org.junit.jupiter.api.Assertions.*

### assertEquals(expected, actual)

실제 값이 기대한 값과 같은지 확인한다.

```java
@DisplayNameGeneration(DisplayNameGenerator.ReplaceUnderscores.class)
class StudyTest {    
    @Test
    @DisplayName("스터디 만들기")
    void create_new_study() {
        Study study = new Study();
        assertNotNull(study);

        // 메시지를 적어줘야 테스트 실패 시 확인이 용이하다.
        assertEquals(StudyStatus.DRAFT, study.getStatus(), "스터디를 처음 만들면 상태값이 DRAFT여야 한다.");

        // 람다로 표현하기
        // 람다로 표현하면 -> 뒤의 실행을 테스트가 실패했을 때만 하기 때문에, 문자열 연산을 최대한 하지 않는 이점이 있다.
        // 문자열 연산에 부담이 가는 테스트라면 람다를 사용해 성능을 높일 수 있다.
        assertEquals(StudyStatus.DRAFT, study.getStatus(), () -> "스터디를 처음 만들면 상태값이 DRAFT여야 한다.");

        // executable로 표현하기
        // 이 표현을 간추린 것이 위의 람다식이다.
        assertEquals(StudyStatus.DRAFT, study.getStatus(), new Supplier<String>() {
            @Override
            public String get() {
                return "스터디를 처음 만들면 상태값이 DRAFT여야 한다.";
            }
        });
        
    }
}
```

### assertNotNull(actual)

값이 null이 아닌지 확인한다.

### assertTrue(boolean)

다음 조건이 참(true)인지 확인한다.

### assertAll(executables...)

모든 확인 구문을 확인한다.

```java
@DisplayNameGeneration(DisplayNameGenerator.ReplaceUnderscores.class)
class StudyTest {

    @Test
    @DisplayName("스터디 만들기")
    void create_new_study() {
        Study study = new Study(-10);

        assertEquals(StudyStatus.DRAFT, study.getStatus(), "스터디를 처음 만들면 상태값이 DRAFT여야 한다.");
        assertTrue(study.getLimit() > 0, "스터디 최대 참석 가능 인원은 0보다 커야 한다.");
    }
}
```

여러 개의 assert문을 실행하면, 앞에서 실패했을 경우 나머지 구문들이 실행되지 않는다. 이 문제를 해결하기 위해 `assertAll()`을 사용할 수 있다.

```java
@DisplayNameGeneration(DisplayNameGenerator.ReplaceUnderscores.class)
class StudyTest {

    @Test
    @DisplayName("스터디 만들기")
    void create_new_study() {
        Study study = new Study(-10);

        assertAll(
                () -> assertNotNull(study),
                () -> assertEquals(StudyStatus.DRAFT, study.getStatus(), "스터디를 처음 만들면 상태값이 DRAFT여야 한다."),
                () -> assertTrue(study.getLimit() > 0, "스터디 최대 참석 가능 인원은 0보다 커야 한다.")
        );
    }
}
```

각각을 람다식으로 표현해 넣어주면 된다.

### assertThrows(expectedType, executable)

예외가 발생하는지 확인한다.
```java
@DisplayNameGeneration(DisplayNameGenerator.ReplaceUnderscores.class)
class StudyTest {

    @Test
    @DisplayName("스터디 만들기")
    void create_new_study() {
        // 발생하는 exception을 파라미터로 받을 수도 있다.
        IllegalArgumentException illegalArgumentException = assertThrows(IllegalArgumentException.class, () -> new Study(-10));

        // 받은 파라미터로 기대했던 메시지와 같은지 확인할 수 있다.
        String message = illegalArgumentException.getMessage();
        assertEquals("limit은 0보다 커야한다.", message);
    }
}
```

### assertTimeout(duration, executable)

특정 시간 안에 실행이 완료되는지 확인한다.

```java
@DisplayNameGeneration(DisplayNameGenerator.ReplaceUnderscores.class)
class StudyTest {

    @Test
    @DisplayName("스터디 만들기")
    void create_new_study() {
        // 특정 시간 안에 끝나야 할 때
        assertTimeout(Duration.ofSeconds(10), () -> new Study(10));
    }
}
```

마지막 매개변수로 Supplier<String> 타입의 인스턴스를 람다 형태로 제공할 수 있다.
복잡한 메시지 생성해야 하는 경우 사용하면 실패한 경우에만 해당 메시지를 만들게 할 수 있다.

(작성중)