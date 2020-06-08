# 스프링부트에서 JPA로 데이터베이스 다루기

## JPA의 등장

이전에는 MyBatis와 같은 SQL 매퍼로 쿼리를 작성했다. 그러다보니 개발하는 시간보다 SQL을 다루는 시간이 많아졌다. 이후 JPA라는 자바 표준 ORM이 등장하면서 문제를 해결하게 된다.

> SQL 매퍼는 쿼리를 매핑하는 것이고 ORM은 객체를 매핑하는 것이다.

## JPA 소개

관계형 데이터베이스는 애플리케이션 개발에 꼭 필요한 요소다. 그러다보니 객체를 관계형 데이터 베이스에서 관리하는 것이 중요해졌다.

관계형 데이터베이스가 계속 웹 서비스의 중심이 되면서 프로젝트에는 코드보다 SQL이 가득 차게 되었다. 아무리 개발자가 코드를 잘 짜더라도 RDB는 SQL만 인식할 수 있으므로 매번 기본적인 CRUD SQL을 생성해야 했던것이다.

RDB는 어떻게 데이터를 저장할지에 집중한다면 객체지향 프로그래밍 언어는 메시지를 기반으로 기능과 속성을 관리하는 기술이다.

이렇게 다른 목적과 시작점에서 출발해 발행한 문제를 `패러다임 불일치`라고 부른다.

```java
public class Test {
    User user = findUser();
    Group group = user.getGroup();
}
```

위의 코드는 명확하게 User와 Group이 부모 자식 관계임을 알 수 있다. User에서 Group을 가지고 오고 있기 때문이다.

하지만 여기에 데이터베이스가 개입된다면,

```java
public class Test {
    User user = userDao.findUser();
    Group group = groupDao.findGroup(user.getGroupId());
}
```

User와 Group을 따로 조회하게 된다. 둘이 어떤 관계인지 알아내거나 상속, 1:N과 같은 다양한 객체 모델링을 데이터베이스로 표현하기 어렵다.

이렇게 서로 지향하는 바가 다른, 객체지향 프로그래밍 언어와 관계형 데이터베이스라는 두 영역을 중간에서 연결해주는 기술이 JPA다.

이제 개발자가 객체지향적으로 코드를 짜면 JPA가 RDB에 맞게 SQL을 대신 생성하고 실행해준다. 개발자는 더이상 SQL에 종속적인 개발을 하지 않아도 된다.


## Spring Data JPA

JPA는 인터페이스인 자바 표준 명세서다. 인터페이스를 사용하려면 구현체가 필요한데, 대표적으로 Hibernate, Eclipse Kink 등이 있다. 하지만 Spring에서는 이 구현체를 직접 다루지 않는다.

Spring Data JPA는 구현체를 좀 더 쉽게 사용하기 위해 추상화시킨 모듈이다.

> JPA < Hibernate < Spring Data JPA

Hibernate를 쓰는 것과 Spring Data JPA를 쓰는 것 사이에 큰 차이가 없음에도 한 단계 감싸놓은 이유는 두 가지가 있다.

### 구현체 교체의 용이성

언젠가 Hibernate가 망해서 다른 구현체를 사용해야 할 때, Spring Data JPA를 쓰고 있었다면 아주 쉽게 교체할 수 있다.

### 저장소 교체의 용이성

트래픽 등의 이유로 RDB로는 감당이 안 될 때 MongoDB가 필요하다면, Spring Data JPA에서 Spring Data MongoDB로 의존성만 교체하면 된다.

이는 SPring Data의 하위 프로젝트들이 CRUD 인터페이스를 `save()`, `findAll` 등 같은 방식으로 구현하고 있기 때문이다.