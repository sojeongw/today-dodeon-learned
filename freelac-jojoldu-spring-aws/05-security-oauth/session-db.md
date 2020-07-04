# 세션 저장소로 데이터베이스 사용하기

지금까지 만든 서비스는 애플리케이션을 재실행하면 로그인이 풀린다. 세션이 내장 톰캣 메모리에 저장되기 때문이다. 따라서 애플리케이션을 실행할 때마다 항상 초기화된다. 

게다가 만약 2대 이상의 서버에서 서비스를 하고 있다면, 톰캣마다 세션 동기화 설정이 필요하다. 이런 문제 때문에 현업에서는 다음의 3가지 중 하나를 선택한다.

1. 톰캣 세션을 사용한다.
    - 별 다른 설정을 하지 않을 때 기본으로 선택하는 방식이다.
    - 톰캣에 세션이 저장되므로 2대 이상의 WAS가 구동되는 환경에서는 톰캣끼리 세션을 공유하기 위한 추가 설정이 필요하다.
2. 데이터베이스를 세션 저장소로 사용한다.
    - 여러 WAS 간에 공용으로 세션을 사용할 수 있는 가장 쉬운 방법이다.
    - 설정을 많이 하지 않아도 되지만, 로그인을 요청할 때마다 DB I/O가 발생해 성능 이슈가 있다.
    - 그래서 로그인 요청이 많이 없는 백오피스, 사내 시스템 용도로 많이 사용한다.
3. Redis, Memcached와 같은 메모리 DB를 세션 저장소로 사용한다.
    - B2C 서비스에서 가장 많이 사용하는 방식이다.
    - 실제 서비스로 사용하려면 Embedded Redis와 같은 방식이 아닌 외부 메모리 서버가 필요하다.
    
이 책에서는 두 번째 방식으로 진행한다. 설정이 간단하고 사용자가 많지 않으며 비용을 절감할 수 있기 때문이다. 레디스나 엘라스틱 캐시같은 서비스는 별도로 사용료를 지불해야 한다.

## spring-session-jdbc 등록

{% tabs %}
{% tab title="build.gradle" %}
```groovy
dependencies {
    ...
    compile('org.springframework.session:spring-session-jdbc')
}
```
{% endtab %}
{% tab title="application.properties" %}
```properties
spring.session.store-type=jdbc
```
{% endtab %}
{% endtabs %}

먼저 `spring-session-jdbc` 의존성을 추가한다. 그리고 `application.properties`에 세션 저장소를 jdbc로 선택하도록 코드를 추가한다.

![](../../.gitbook/assets/freelac-jojoldu-spring-aws/05/스크린샷%202020-09-13%20오후%206.25.04.png)

`h2-console`에 접속하면 세션을 위한 `SPRING_SESSION`, `SPRING_SESSION_ATTRIBUTES` 테이블을 볼 수 있다.

세션 저장소를 DB로 교체했지만 H2 또한 스프링을 재실행하면 재시작되기 때문에 세션도 풀려버린다. 나중에 AWS에 배포하게 되면 AWS의 데이터베이스 서비스인 RDS를 이용해 세션이 풀리지 않게 할 수 있다.