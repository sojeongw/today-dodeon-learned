# JPA Auditing으로 생성시간/수정시간 자동화하기

데이터의 생성 시간, 수정 시간을 등록하는 코드가 반복적으로 들어가면 코드가 지저분해진다. 이 문제를 해결하기 위해 JPA Auditing을 사용할 수 있다.

## LocalDate

Java 8부터 `LocalDate`와 `LocalDateTime`이 등장했다. Java의 기본 날짜 타입인 `Date`의 문제를 제대로 고친 타입이므로 Java 8이면 무조건 사용해야 한다.

### Date와 Calendar의 문제점

#### 불변 객체가 아니다.

불변 객체란 `변경이 불가능한 객체`를 의미한다. 멀티 스레드 환경이라면 언제든 문제가 발생할 수 있다.

#### Calendar는 월(Month) 값 설계가 잘못되었다.

`Calendar`에서 10월을 나타내는 `Calendar.OCTOBER`의 값은 9이다. 당연히 10이라고 생각하는 개발자들에게 혼란을 불러일으킨다.

**Reference**

[Java의 날짜와 시간 API](https://d2.naver.com/helloworld/645609)

## JPA Auditing 적용

{% tabs %}
{% tab title="BaseTimeEntity.java" %}
```java
@Getter
@MappedSuperclass
@EntityListeners(AuditingEntityListener.class)
public class BaseTimeEntity {
    
    @CreatedDate
    private LocalDateTime createdDate;
    
    @LastModifiedDate
    private LocalDateTime modifiedDate;
}
```
{% endtab %}
{% tab title="Posts.java" %}
```java
@Getter
@NoArgsConstructor
@Entity
public class Posts extends BaseTimeEntity {
    ...   
}
```
{% endtab %}
{% tab title="Application.java" %}
```java
@EnableJpaAuditing
@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

}
```
{% endtab %}
{% endtabs %}

### @MappedSuperclass

JPA Entity 클래스들이 `BaseTimeEntity`를 상속할 경우 `createdDate`, `modifiedDate`과 같은 필드들도 칼럼으로 인식하도록 한다.

### @EntityListeners(AuditingEntityListener.class)

`BaseTimeEntity` 클래스에 Auditing 기능을 포함시킨다.

### @CreatedDate

Entity가 생성되어 저장될 때 시간이 자동 저장된다.

### @LastModifiedDate

조회한 Entity의 값을 변경할 때 시간이 자동 저장된다.

### @EnableJpaAuditing

JPA Auditing을 활성화한다.

{% tabs %}
{% tab title="PostsRepositoryTest.java" %}
```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class PostsRepositoryTest {

   ...

    @Test
    public void BaseTimeEntity_등록() {
        // given
        LocalDateTime now = LocalDateTime.of(2019, 6, 4, 0, 0, 0);
        postsRepository.save(Posts.builder().title("title").content("content").author("author").build());

        // when
        List<Posts> postsList = postsRepository.findAll();

        // then
        Posts posts = postsList.get(0);

        System.out.println(">>>>> createdDate= " + posts.getCreatedDate() + ", modifiedDate= " + posts.getModifiedDate());

        assertThat(posts.getCreatedDate()).isAfter(now);
        assertThat(posts.getModifiedDate()).isAfter(now);
    }
}
```
{% endtab %}
{% endtabs %}

테스트를 해보면 실제 날짜 데이터가 잘 들어가있음을 확인할 수 있다.