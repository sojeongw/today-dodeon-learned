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

### UserService와 UserDao의 트랜잭션 경계 설정

우리는 코드 어디에도 트랜잭션을 시작하고 커밋/롤백 하는 코드를 설정하지 않았으므로 테스트에 실패한 것이다.

![](../../.gitbook/assets/toby/screenshot%202020-03-01%20오후%209.37.16.png)
