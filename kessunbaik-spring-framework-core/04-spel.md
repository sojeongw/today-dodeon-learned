# SpEL (Spring Expression Language)

- 객체 그래프를 조회하고 조작하는 기능을 제공한다.
- 메서드 호출과 문자열 템플릿 기능도 지원한다.
- 스프링 3.0부터 지원한다.

## 예시

```text
ExpressionParser parser = new SpelExpressionParser();
```

```text
StandardEvaluationContext context = new StandardEvaluationContext(bean);
```

```text
Expression expression = parser.parseExpression(“SpEL 표현식”);
```

```text
String value = expression.getvalue(context, String.class);
```

## 문법

- #{“표현식"}
- ${“프로퍼티"}
- 표현식은 프로퍼티를 가질 수 있지만, 그 반대는 안된다.
  - ex. #{${my.data} + 1}

```java

@Component
public class AppRunner implements ApplicationRunner {

    // 일반적인 표현식 사용 예
    @Value("#{1 + 1}")
    int value;

    @Value("#{'hello' + ' world'}")
    String greeting;

    @Value("#{1 eq 1}")
    boolean trueOrFalse;

    // 프로퍼티 사용 예
    @Value("${my.value}")
    String myValue;

    @Value("#{${my.value} eq 100}")
    boolean isMyValue100;

    // 빈 속성 사용 예 (여기서는 Sample 이라는 빈을 사전에 정의 했음)
    @Value("#{sample.value}")
    int sampleValue;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        System.out.println(value);
        System.out.println(greeting);
        System.out.println(trueOrFalse);
        System.out.println(myValue);
        System.out.println(isMyValue100);
        System.out.println(sampleValue);
    }
}
```

```text
2
hello world
true
100
true
100
```

## 용례

- @Value
- @ConditionalOnExpression
- 스프링 시큐리티
    - @PreAuthorize
    - @PostAuthorize
    - @PreFilter
    - PostFilter
- 스프링 데이터
    - @Query
- Thymeleaf