# 트랜잭션 서비스 추상화

레벨 업그레이드를 하는 도중에 장애가 생기면 어떻게 될까? 의도적으로 장애를 발생시켜 테스트 코드를 확인해보자.

```java
public class UserServiceTest {
    ...

    @Test
	public void upgradeAllOrNothing() {
        // 예외를 발생시킬 사용자의 id를 테스트용 오브젝트에 넣는다.
		UserService testUserService = new TestUserService(users.get(3).getId());  
        // 테스트용으로 한 번 쓰이는 것이니 userDao를 수동 DI 한다.
        // 이제 testUserService는 userService 빈과 동일하게 UserDao로 데이터 액세스 기능을 이용한다.
		testUserService.setUserDao(this.userDao);
		
		userDao.deleteAll();	
        // 다섯 명의 User 정보를 등록한다.		  
		for(User user : users) userDao.add(user);
		
		try {
            // 업그레이드 하는 도중에 예외가 발생해야 한다.
			testUserService.upgradeLevels();   
            // 혹시라도 예외가 발생하지 않으면 fail() 메서드로 테스트를 실패시킨다.
            // 테스트가 의도한 대로 동작하는지 확인하기 위해 집어넣은 코드다.
			fail("TestUserServiceException expected"); 
		}
        // TestUserService가 던지는 예외를 잡아서 계속 진행되도록 한다.
		catch(TestUserServiceException e) { 
		}
		
        // 예외가 발생하기 전에 레벨 변경이 있었던 사용자가 처음 상태로 다시 돌아갔는지 확인한다.
        // 예외가 발생한 건 4번째 사용자이므로 2번째 사용자는 원래 상태로 돌아가야 한다.
		checkLevelUpgraded(users.get(1), false);
	}

    // Userservice를 상속한 클래스를 하나 만든다.
    // 테스트에서만 사용할 클래스이므로 테스트 클래스 내부에 static으로 선언한다.
    static class TestUserService extends UserService {
        private String id;
    
        // 예외를 발생시킬 User 오브젝트의 id를 지정한다.
        private TestUserService(String id) {
            this.id = id;
        }
    
        // Userservice의 메서드를 오버라이딩 한다.
        // UserService 대부분의 메서드는 private으로 되어있는데
        // 이 경우 오버라이딩이 불가하므로 먼저 부모 클래스에서 protected로 변경 후 오버라이딩 한다.
        protected void upgradeLevel(User user) {
            // 지정된 id의 User 오브젝트가 발견되면 예외를 던져서 작업을 강제로 중단한다.
            if(user.getId().equals(this.id)) throw new TestUserServiceException();
            super.upgradeLevel(user);
        }
    }

    // 테스트 내부에서만 사용할 예외를 static 멤버 클래스로 만든다.
    static class TestUserServiceException extends RuntimeException {
    }
}
```

테스트 결과는 아래와 같다.

```text
java.lang.AssertionError: 
Expected: is <BASIC>
got: <SILVER>
```

왜 의도한대로 결과가 나오지 않을까? 트랜잭션 때문이다. 트랜잭션은 더 이상 나눌 수 없는 단위의 작업을 말한다. 모든 작업은 전체가 다 성공하든지 다 실패하든지 둘 중 하나여야 한다.

하지만 upgradeLevels() 메서드의 작업은 하나의 트랜잭션으로 적용되지 않았기 때문에 완전 성공 혹은 완전 실패가 나오지 않는 것이다.

## 트랜잭션 경계 설정

DB는 자체적으로 트랜잭션을 보장한다. 하지만 여러 SQL을 사용하는 작업을 하나의 트랜잭션으로 취급해야 한다면 아래의 두 가지를 고려해야 한다.

- 트랜잭션 롤백
    - 작업 중간에 문제가 발생할 경우 앞에서 처리한 SQL 작업도 취소시킨다.
- 트랜잭션 커밋
    - 모든 SQL 작업이 성공적으로 마무리되어 작업을 확정한다고 DB에 알린다.

트랜잭션이 시작되고 끝나는 위치를 `트랜잭션 경계`라고 부르며, 트랜잭션 시작을 선언하고 커밋이나 롤맥을 하는 작업을 `트랜잭션 경계 설정`이라고 부른다.

## UserService와 UserDao의 트랜잭션 경계 설정

우리는 코드 어디에도 트랜잭션을 시작하고 커밋/롤백 하는 코드를 설정하지 않았으므로 테스트에 실패한 것이다.

![](../../.gitbook/assets/toby/screenshot%202020-03-01%20오후%209.37.16.png)

위 그림은 `UserService`와 `UserDao`를 통해 트랜잭션이 일어나는 과정을 나타낸 것이다.

`upgradeLevels()`가 `UserDao`의 `update()`를 세번 호출했다면, 두 번째 호출에서 오류가 발생해 작업이 중단되어도 첫 번째 커밋한 트랜잭션 결과는 DB에 남는다.

이렇게 되면 DAO 메서드에서 DB 커넥션을 매번 만드므로 `UserService`의 여러 작업을 하나로 묶는 게 불가능해진다.

## 비즈니스 로직 내의 트랜잭션 경계설정

`upgradeLevels()`처럼 여러 번 DB에 업데이트 하는 과정을 하나의 트랜잭션으로 묶으려면 어떻게 해야 할까?

결국 트랜잭션 경계 설정 작업을 `UserService`로 가져가야 한다. `upgradeLevels()` 메서드의 시작과 끝에서 트랜잭션 작업이 이루어져야 하기 때문이다.

```java
public class UserService {
    public void upgradeLevels() throws Exception {
        // 1. DB Connection 생성
    
        // 2. 트랜잭션 시작
    
        try {
            // 3. DAO 메서드(update) 호출
    
            // 4. 트랜잭션 커밋
        }
        catch(Exception e) {
            // 5. 트랜잭션 롤백
            throw e;
        }
        finally{
            // 6. DB Connection 종료
        }
    }
}
```

순수하게 데이터 액세스 작업을 하는 코드는 `update()`에 있어야 하며, `Connection`은 반드시 `upgradeLevels()`에서 만든 것을 사용해야 같은 트랜잭션 안에서 동작한다.

![](../../.gitbook/assets/toby/screenshot%202020-04-07%20오전%2012.00.06.png)

이때 `upgrade()`를 직접 호출하는 것은 결국 `upgradeLevel()`이므로 `UserService` 내의 메서드끼리도 `Connection` 오브젝트를 사용하도록 전달해야 한다.

## UserService 트랜잭션 경계설정의 문제점

