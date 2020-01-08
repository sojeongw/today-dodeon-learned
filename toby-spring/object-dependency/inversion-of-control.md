# 제어의 역전

## 개념

제어의 역전\(Inversion of Control, IoC\)은 프로그램의 제어 흐름 구조가 거꾸로 되는 것이다.

원래 프로그램은 `main()` 에서 시작해 사용할 오브젝트를 결정하고 그 오브젝트를 만들어 안에 있는 메소드를 호출하는 일을 반복한다. `main()` 이 `능동적으로` 무엇을 사용할지 결정하고 만드는 구조다.

제어의 역전은 스스로 무슨 오브젝트를 사용할지 선택하지 않는다. 생성하지도 않고 자기 자신이 어떻게 만들어지고 사용되는지 모른다. 모든 권한을 제 3자에게 넘긴다. 그냥 기능만 구현해 놓으면 자신은 필요할 때 호출되어 사용되는 `수동적인` 구조다.

## 대표적인 예

### 프레임워크

| 라이브러리 | 프레임워크 |
| :---: | :---: |
| 애플리케이션 흐름을 직접 제어하고 필요할 때 능동적으로 라이브러리를 사용함 | 프레임워크가 흐름을 주도하는 도중에 애플리케이션 코드를 프레임워크가 사용함 |

최근 툴킷, 엔진, 라이브러리 등이 유행하면서 `프레임워크`라고 불리기도 하는데 이는 잘못된 것이다. `프레임워크`에는 `제어의 역전` 개념이 분명히 적용되어 있어야 한다. 애플리케이션 코드가 `프레임워크` 안에서 수동적으로 동작해야 한다.

### UserDao와 DaoFactory

원래 초기 코드에서 `ConnectionMaker`는 `UserDao`에 있었다. 그런데 지금은 `DaoFactory`에 있다. 어떤 구현 클래스를 사용할지 `DaoFactory` 에게 위임 했으니 이제 수동적으로 만들어지는 것이다.

## 스프링 IoC

제어의 역전은 컴포넌트를 생성하고, 관계를 설정하고, 어떻게 사용할지 등을 관리할 존재가 필요하다. IoC를 애플리케이션 전반에 걸쳐 사용하려면 스프링 같은 IoC 프레임워크를 사용하는 것이 훨씬 유리하다. `DaoFactory`가 하는 일을 좀 더 일반화 했다고 생각하면 된다.

### 빈

스프링이 직접 만들고 관계를 부여하는 오브젝트. 특히 스프링 빈은 스프링 컨테이너가 제어하는, 제어의 역전이 적용된 오브젝트를 말한다. 모든 오브젝트가 다 빈은 아니다. 스프링이 직접 생성과 제어를 담당해 IoC 방식으로 관리하는 오브젝트만 해당한다.

### 빈 팩토리

빈을 만들고 관계를 설정하는 IoC 오브젝트를 `빈 팩토리`라고 한다. IoC를 담당하는 핵심 컨테이너이며 `빈 팩토리`를 직접 사용하기 보다는 이를 확장(상속)한 `애플리케이션 컨텍스트`를 이용한다. `BeanFactory`라는 인터페이스에 `getBean()`이 들어있다.

### 애플리케이션 컨텍스트

보통 `빈 팩토리` 보다는 더 확장된 의미인 `애플리케이션 컨텍스트`를 사용한다. `애플리케이션 컨텍스트`는 빈의 생성, 관계 설정의 정보를 설정 파일에서 가져와 활용하는 범용적인 IoC 엔진이다.

이전에 `컴포넌트`와 `설계도`를 설명했었는데, 여기서의 `설계도`가 `애플리케이션 컨텍스트`와 `설정 정보`를 의미한다.

### 설정정보/설정 매타정보

`애플리케이션 컨텍스트` 또는 `빈 팩토리`가 IoC를 적용하기 위해 사용하는 메타정보다. IoC 컨테이너가 관리하는 오브젝트를 생성하고 구성할 때 사용한다.

### 컨테이너/IoC 컨테이너

`애플리케이션 컨텍스트`나 `빈 팩토리`를 일컫는다. `애플리케이션 컨텍스트`는 그 자체로 `ApplicationContext` 인터페이스를 구현한 오브젝트이지만 한 애플리케이션 안에 들어 있는 여러 개를 통틀어 `스프링 컨테이너`라고 부른다.

### 스프링 프레임워크

IoC 컨테이너, 애플리케이션 컨텍스트를 포함해 스프링이 제공하는 모든 기능을 통틀어 말할 때 사용한다.

## DaoFactory에 애플리케이션 컨텍스트 적용하기

### 설정 정보 만들기

`DaoFactory`를 스프링에 적용해보자.

{% tabs %}
{% tab title="After" %}
```java
@Configuration  // 애플리케이션 컨텍스트가 사용할 설정 정보라는 표시
public class DaoFactory {

    @Bean   // 오브젝트 생성을 담당하는 IoC 메소드라는 표시
    public UserDao userDao() {
        UserDao userDao = new UserDao(connectionMaker());

        return userDao;
    }

    @Bean
    public ConnectionMaker connectionMaker() {
        return new DConnectionMaker();
    }
}
```
{% endtab %}

{% tab title="Before" %}
```java
public class DaoFactory {
    public UserDao userDao() {
        UserDao userDao = new UserDao(connectionMaker());

        return userDao;
    }

    public ConnectionMaker connectionMaker() {
        return new DConnectionMaker();
    }
}
```
{% endtab %}
{% endtabs %}

설정 정보를 담당하는 클래스라고 알려주기 위해 `@Configuraton` 애노테이션을 추가한다. 그리고 오브젝트를 실제 만드는 메소드에 `@Bean` 애노테이션을 붙인다. 이제 `애플리케이션 컨텍스트`가 IoC를 적용할 때 사용할 설정 정보가 세팅되었다.

### 애플리케이션 컨텍스트 만들기

이제 위의 설정 정보를 사용하는 애플리케이션 컨텍스트를 만들어보자.

{% tabs %}
{% tab title="After" %}
```java
public class UserDaoTest {
    public static void main(String[] args) {
        // 애플리케이션 컨텍스트 생성
        // 생성자 파라미터로 설정 정보인 DaoFactory를 보낸다.
        AnnotationConfigurationContext context = new AnnotationConfigurationContext(DaoFactory.class);

        // getBean()으로 애플리케이션 컨텍스트가 관리하는 오브젝트를 요청한다.
        UserDao dao = context.getBean("userDao", UserDao.class);
    }
}
```
{% endtab %}

{% tab title="Before" %}
```java
public class UserDaoTest {
    public static void main(String[] args) {
        UserDao dao = new DaoFactory.userDao();
    }
}
```
{% endtab %}
{% endtabs %}

`AnnotationConfigurationContext`를 이용하면 `@Configuration`이 붙은 자바 코드를 설정 정보로 가져올 수 있다.

`애플리케이션 컨텍스트`가 관리하는 `UserDao`는 `getBean()`으로 불러온다. `@Bean`이 붙은 메소드의 이름이 빈의 이름이 되어 getBean\(\)의 첫 번째 파라미터로 넘겨진다. 만약 생성하는 방법이나 구성을 다르게 설정한 메소드를 따로 추가했다면 그 메소드 이름을 적으면 된다.

## 애플리케이션 컨텍스트의 동작 방식

기존에 `오브젝트 팩토리`를 사용했던 코드와 비교해보자.

`오브젝트 팩토리`에 대응되는 것이 `애플리케이션 컨텍스트`다. `IoC 컨테이너`, `스프링 컨테이너`, `빈 팩토리`라고도 부른다.


| 오브젝트 팩토리 | 애플리케이션 컨텍스트 |
| :---: | :---: |
| DAO 오브젝트 생성 및 DB 생성 오브젝트의 관계 설정 등 제한적 역할 | IoC를 적용해 모든 오브젝트에 대한 생성과 관계 설정 담당 |
| 직접 오브젝트를 생성하고 관계 맺는 코드가 있음 | 직접적인 코드 없이 설정 정보나 외부 오브텍트 팩토리를 통해 얻음 |
| `DaoFactory` 처럼 어떤 팩토리 클래스를 가져올지 알아뒀다가 필요할 때마다 생성해야 함 | 클라이언트가 구체적인 팩토리 클래스를 알 필요가 없음 |
| 오브젝트 생성과 다른 오브젝트 간의 관계 설정이 주요 기능임 | 오브젝트 생성 뿐만 아니라 후처리, 조합, 설정 다변화, 인터셉팅 등 다양한 기능을 제공함 |

`애플리케이션 컨텍스트`는 `@Configuration`이 붙은 설정 정보를 불러오고 `@Bean`이 붙은 메소드의 이름을 가져와 bean 목록을 만든다.

클라이언트가 `애플리케이션 컨텍스트`의 `getBean()`을 요청하면 bean 목록에서 요청한 이름이 있는지 찾는다. 그리고 bean을 생성하는 오브젝트를 호출해서 생성하고 클라이언트에 돌려준다.

`getBean()`은 타입만으로 빈을 검색하거나 특별한 애노테이션 설정이 되어 있는 빈을 찾을 수도 있다.