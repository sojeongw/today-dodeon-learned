# 학습 테스트

자신이 만들지 않은 프레임워크나 다른 개발팀에서 만들어 제공한 라이브러리에 대해 테스트 하는 것을 `학습 테스트`라고 한다. 학습 테스트는 자신이 사용할 API나 프레임워크의 기능을 테스트하면서 사용법을 익히려는 것이다. 기능에 대한 검증이 목적이 아니다.

테스트 코드를 만드는 과정을 통해 API의 사용 방법을 익히고 내가 가진 기술에 대한 지식도 검증할 수 있다. 어설프게 알고 있거나 오해하고 있던 지식을 테스트를 통해 바로 잡기도 한다. 테스트로 만들어서 돌려보면 잘못 이해했던 기술이 금세 확인되기 때문이다.

학습 테스트는 테스트 대상보다는 `테스트 코드 자체`에 관심을 갖고 만들어야 한다.

## 장점

### 다양한 조건에 따라 기능을 확인할 수 있다

예제를 만들면서 학습하는 것은 수동 테스트와 비슷하다. 조건에 따라 어떻게 동작하는지 보려면 수동으로 값을 입력하거나 코드를 수정해가며 예제를 계속 실행해야 한다. 결과도 콘솔에 메시지를 출력하거나 UI 화면에 나타내주는 방법밖에 없다.

반면 학습 테스트는 자동화된 테스트 코드로 만들어지기 때문에 조건에 따른 다양한 결과를 빠르게 확인할 수 있다.

### 학습 테스트 코드를 개발 중에 참고할 수 있다

수동으로 예제를 만들면 결국 최종 수정한 예제 코드만 남는다. 반면 학습 테스트는 다양한 기능과 조건에 대해 개별적으로 테스트 코드를 만들 수 있다.

이렇게 테스트 코드를 만들어두면 실제 개발에서 샘플 코드로 참고할 수 있다. 설정 파일, 초기화 방법, API 호출 방식, 결과 출력 방법 등이 모두 테스트 안에 만들어지기 때문이다.

토비 또한 새로운 기술을 사용할 때면 학습 테스트로 사용법을 익히고 이를 애플리케이션 테스트 패키지의 일부로 추가해둔다. 그리고 필요할 때마다 테스트 코드를 참고한다.

### 프레임워크나 제품을 업그레이드할 때 호환성 검증을 도와준다

요즘은 제품이 인터넷을 통해 매우 빠르게 업그레이드 된다. 이때 기존에는 잘 동작하던 기능이 문제를 일으킬 수 있다.

하지만 학습 테스트를 미리 만들어놓았다면 새로운 버전을 테스트에 적용해보고 미리 확인할 수 있다. 버그가 있다면 업그레이드 일정을 늦추거나 애플리케이션 코드를 수정할 수 있을 것이다.

이는 주요 기능에 대한 학습 테스트를 충분히 만들어놨을 경우에 가능하다.

### 테스트 작성에 대한 좋은 훈련이 된다

테스트 작성에 훈련이 되어있지 않거나 부담이 된다면 학습 테스트로 연습할 수 있다. 학습 테스트는 실제 사용하는 애플리케이션 코드의 테스트와 비슷하게 만들어지기 때문이다.

또한 한 두가지 간단한 기능에만 초점을 맞춰도 되기 때문에 단순하고 부담이 적다. 새로운 테스트 방법을 연구하는 데에도 도움이 된다.

### 새로운 기술을 공부하는 과정이 즐거워진다.

책이나 레퍼런스 문서는 쉽게 지루해지고 능률을 떨어뜨린다. 반면 테스트 코드를 만들면서 학습하면 흥미롭고 재미있다. 동작하는 것을 직접 볼수 있기 때문이다.

## 학습 테스트에 좋은 소스

스프링 자신에 대한 테스트 코드를 추천한다. 스프링은 꼼꼼하게 테스트하며 개발해온 프레임워크이기 때문에 거의 모든 기능에 대해 테스트가 만들어져 있다.

스프링 배포판 소스를 보면 소스 코드와 함께 테스트 코드도 담겨있다. 테스트를 보면 레퍼런스에 없는 중요한 정보도 많이 얻을 수 있으며 테스트 작성 방법에 대한 좋은 팁을 얻을 수 있다.

## 학습 테스트 예제

### JUnit 테스트 오브젝트 테스트

JUnit은 테스트 메소드를 수행할 때마다 새로운 오브젝트를 만든다고 했다. 그런데 정말 그럴까? 한번 테스트해보자.

```java
public class JUnitTest {
    static JUnitTest testObject;

    @Test
    public void test1() {
        // 지금 쓰는 클래스와 static 객체를 비교해본다.
        assertThat(this, is(notsameInstance(testObject)));  // 같지 않아야 성공
        testObject = this;
    }

    // 테스트를 수행할 때마다 새롭게 생성되는지 확인하기 위해 메소드를 여러 개 만든다.
    @Test
    public void test2() {
        assertThat(this, is(notsameInstance(testObject)));
        testObject = this;
    }

    @Test
    public void test3() {
        assertThat(this, is(notsameInstance(testObject)));
        testObject = this;
    }
}
```

`sameInstance`는 실제로 같은 오브젝트인지 비교하는 매처이다. `equals()`를 써서 같은 주소 값을 갖는지 체크해도 되지만 테스트의 의도를 명확하게 드러내기 위해 `sameInstance`를 사용했다.

이 방식은 직전 테스트에서 만들어진 테스트 오브젝트와만 비교하므로 첫 번째 오브젝트와 세 번째 오브젝트가 같을 가능성도 있다. 검증 방법을 바꿔보자.

```java
public class JUnitTest {
    // HashSet으로 오브젝트를 저장한다.
    static Set<JUnitTest> testObjects = new HashSet<JUnitTest>();

    @Test
    public void test1() {
        // static에 담긴 오브젝트와 지금 만들어진 오브젝트 중 겹치는 게 없어야 한다.
        assertThat(testObjects, not(hasItem(this)));  // 같지 않아야 성공
        testObject = this;
    }

    @Test
    public void test2() {
        assertThat(testObjects, is(notsameInstance(testObject)));
        testObject = this;
    }

    @Test
    public void test3() {
        assertThat(testObjects, is(notsameInstance(testObject)));
        testObject = this;
    }
}
```

### 스프링 테스트 컨텍스트 테스트

이번에는 스프링 테스트 컨텍스트 프레임워크에 대한 학습 테스트를 만들어보자. 애플리케이션 컨텍스트는 JUnit과 반대로 딱 하나만 만들어져 공유된다.

테스트를 위한 설정 파일을 만든다. 이때 DI 기능이 아니라 컨텍스트가 생성되는 방식을 보는 것이므로 빈을 등록할 필요는 없다.

```markup
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans 
       http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">
</beans>
```

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("junit.xml")  // 위에서 만든 설정 파일을 넣는다.
public class JUnitTest {
    // 매번 주입해주는 애플리케이션 컨텍스트
    @Autowired 
    ApplicationContext context;

    static Set<JUnitTest> testObjects = new HashSet<JUnitTest>();
    // context를 저장해둘 static 변수
    static ApplicationContext contextObject = null;

    @Test public void test1() {
        assertThat(testObjects, not(hasItem(this)));
        testObjects.add(this);

        // 스태틱 변수가 null이라면 첫 번째 테스트라는 의미이다. 
        // 값이 있다면 현재 컨텍스트와 일치하는지 확인한다.
        // is() 매처는 타입만 일치하면 값을 검증할 수 있다.
        assertThat(contextObject == null || contextObject == this.context, is(true));
        // 스태틱 변수에 현재 컨텍스트를 저장한다. 이제 스태티 변수는 null이 아니다.
        contextObject = this.context;
    }

    @Test public void test2() {
        assertThat(testObjects, not(hasItem(this)));
        testObjects.add(this);

        // assertTrue는 조건문의 결과가 true인지 확인한다. assertThat보다 간결하다.
        assertTrue(contextObject == null || contextObject == this.context);
        contextObject = this.context;
    }

    @Test public void test3() {
        assertThat(testObjects, not(hasItem(this)));
        testObjects.add(this);

        // 이번에는 매처의 조합을 활용해 or 조건으로 비교한다.
        // nullValue는 오브젝트가 null인지 확인한다.
        assertThat(contextObject, either(is(nullValue())).or(is(this.contextObject)));
        contextObject = this.context;
    }
}
```

assert 하는 다양한 방식을 알아보았다. 이중에서 편한 방법을 선택하면 된다.

