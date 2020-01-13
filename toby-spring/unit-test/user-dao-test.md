# 단위 테스트의 필요성

지금까지 우리는 `main()` 메소드를 이용해 `UserDao` 오브젝트의 `add()`, `get()`을 호출하고 테스트해왔다. 테스트를 통해 내가 만든 코드를 확신할 수 있으며, 결과가 다를 경우 결함을 발견하고 디버깅을 할 수 있다.

```java
public class UserDaoTest {
	public static void main(String[] args) throws ClassNotFoundException, SQLException {
		ApplicationContext context = new AnnotationConfigApplicationContext(DaoFactory.class);
		UserDao dao = context.getBean("userDao", UserDao.class);
		
		User user = new User();
		user.setId("whiteship");
		user.setName("백기선");
		user.setPassword("married");

		dao.add(user);
			
		System.out.println(user.getId() + " 등록 성공");
		
		User user2 = dao.get(user.getId());
		System.out.println(user2.getName());
		System.out.println(user2.getPassword());
			
		System.out.println(user2.getId() + " 조회 성공");
	}
}
```

1장에서 만들었던 테스트 코드의 내용은 다음과 같다.

- 자바에서 가장 쉽게 실행할 수 있는 `main()` 메소드 사용
- 테스트 대상인 `UserDao`의 오브젝트를 가져와 메소드 호출
- 테스트에 사용할 `User` 오브젝트 값을 직접 입력
- 테스트 결과와 성공 메시지를 콘솔에 출력

## 기존 방식의 문제점

보통 웹 애플리케이션을 테스트 하려면 `DAO` 뿐만 아니라 `서비스`, `프레젠테이션` 계층까지 대충이라도 다 만들고 서버에 배치해 웹을 띄우고, 폼에 입력을 넣어 확인한다. 이를 위해 폼에서 값을 받아 파싱해 오브젝트를 만들고 `UserDao`를 호출하는 기능이 필요하다.

이렇게 모든 기능을 다 만들고 나서야 테스트를 하게 되면 시간이 많이 걸리고 에러가 발생했을 때 무엇이 정확한 원인인지 확인하기 어렵다. 

## 단위 테스트(Unit Test)

테스트는 가능한 작은 단위로 쪼개서 수행해야 한다. 한꺼번에 몰아서 하면 테스트 과정이 복잡해지고 오류의 정확한 원인을 파악하기 힘들기 때문이다. `관심사의 분리`가 여기에도 적용된다.

`UserDaoTest`는 MVC 클래스나 서비스 오브젝트, 서버 등이 없어도 테스트 할 수 있다. 이렇게 작은 단위로 테스트 하는 것을 `단위 테스트`라고 한다.

`단위`는 딱 정해진 것이 아니라 크게는 사용자 관리 기능을 모두 통틀어 보기도 하고 메소드 하나만 의미할 때도 있다. 하나의 관심에 집중해서 효율적으로 테스트할 범위의 단위라고 생각하면 된다.

각 단위 기능은 잘 동작하는데 전체적으로 테스트 하면 동작하지 않는 경우도 있다. 그럼에도 불구하고 이미 각 단위로 충분히 검증을 했다면 전체 테스트는 훨씬 나을 수 있다.

단위 테스트는 만든 코드가 의도대로 동작하는지 스스로 빨리 확인하기 위한 것이므로 대상과 조건이 명확하고 코드가 짧을 수록 좋다.

### 자동 수행 테스트 코드

웹 화면에 폼을 띄워 값을 입력하고, 버튼을 누르고, 조회하는 등의 작업을 반복하면 지루하고 불편할 것이다. 그래서 테스트는 자동으로 수행되어야 한다.

또한, 테스트 코드를 별개의 클래스에 넣는 것이 좋다. 규모가 커질 수록 테스트 코드를 넣을 위치가 애매해지기 때문이다.

### 지속적인 개선과 점진적인 개발을 위한 테스트

만약 처음부터 모든 코드를 만들고 검증하려 했다면 쉴새없이 쏟아지는 에러 메시지에 막막해졌을 것이다. 하지만 우리는 매우 작은 코드도 테스트를 해가며 만들어왔기 때문에 오히려 작업 속도가 붙고 더 쉬워졌을 수 있다.

기능을 추가할 때에도 미리 만들어둔 테스트 코드 덕분에 코드에 확신을 가지고 조금씩 기능을 추가하면서 점진적으로 발전시킬 수 있다.

## 테스트 실패의 유형

### 테스트 에러

테스트가 진행되는 동안 에러가 발생해서 실패하는 경우

### 테스트 실패

에러가 발생하진 않았지만 결과가 기대한 것과 다르게 나오는 경우

## UserDaoTest의 문제점

### 수동 확인 작업의 번거로움

`UserDao`는 테스트를 수행하고 데이터를 입력하는 과정을 모두 자동으로 진행했다. 하지만 여전히 `add()`로 등록한 정보가 `get()`에서 출력한 값과 일치하는지는 우리 눈으로 확인해야 한다.

### 실행 작업의 번거로움

아무리 간단한 `main()` 메소드라도 매번 실행하는 것은 번거롭다. 만약 `DAO`가 늘어나면 `main()` 메소드도 그만큼 만들어져서 수십 번 실행 버튼을 눌러야 할 것이다.

## 테스트 검증의 자동화

`테스트 에러`는 콘솔에 정보가 출력되므로 확인이 쉽다. 하지만 `테스트 실패`는 별도로 확인을 해야 한다. 이번에는 기대한 결과와 다를 때 `테스트 실패` 메시지를 출력해보자.

{% tabs %}
{% tab title="After" %}
```java
public class UserDaoTest {
	public static void main(String[] args) throws ClassNotFoundException, SQLException {
		ApplicationContext context = new AnnotationConfigApplicationContext(DaoFactory.class);
		UserDao dao = context.getBean("userDao", UserDao.class);
		
		User user = new User();
		user.setId("whiteship");
		user.setName("백기선");
		user.setPassword("married");

		dao.add(user);
			
		System.out.println(user.getId() + " 등록 성공");
		
		User user2 = dao.get(user.getId());

        if (!user.getName().equals(user2.getName())) {
            System.out .println("테스트 실패 (name)");
        }
        else if (!user.getPassword().equals(user2.getPassword())) {
            System . out.println("테스트 실패 (password)") ;
        }
        else {
            System.out.println("조회 테스트 성공");
        }
	}
}
```
{% endtab %}

{% tab title="Before" %}
```java
public class UserDaoTest {
	public static void main(String[] args) throws ClassNotFoundException, SQLException {
		ApplicationContext context = new AnnotationConfigApplicationContext(DaoFactory.class);
		UserDao dao = context.getBean("userDao", UserDao.class);
		
		User user = new User();
		user.setId("whiteship");
		user.setName("백기선");
		user.setPassword("married");

		dao.add(user);
			
		System.out.println(user.getId() + " 등록 성공");
		
		User user2 = dao.get(user.getId());
		System.out.println(user2.getName());
		System.out.println(user2.getPassword());
			
		System.out.println(user2.getId() + " 조회 성공");
	}
}
```
{% endtab %}
{% endtabs %}

처음 `add()`에 전달한 값과 `get()`으로 가져오는 값을 비교해서 일치하는지 확인하고 메시지를 출력하도록 했다. 이 테스트는 등록과 조회가 잘 동작하는지 손쉽게 확인시켜준다.

켄트 벡은 "테스트란 개발자가 마음 편하게 잠자리에 들 수 있게 해주는 것" 이라고 했다. 개발자는 코드를 추가하면 기존 기능에 문제가 있을까봐 불안해한다. 하지만 만들어진 코드의 기능을 모두 점검할 수 있는 `포괄적인 테스트`를 만들면 빠르게 조치를 취할 수 있다.

수정할 때 코드에 자신감을 갖게 되며 새 기술에 문제가 없는지 빠르게 확인하는 방법은 스스로 테스트 수행과 결과를 확인해주는 `자동화된 테스트` 작성이다.