# 코드 개선

작성된 코드를 살펴볼 때는 다음과 같은 질문을 던져볼 필요가 있다.

- 코드에 중복된 부분은 없는가?
- 코드가 무엇을 하는 것인지 이해하기 불편하진 않은가?
- 코드가 자신이 있어야 할 자리에 있는가?
- 앞으로 변경이 일어난다면 어떤 것이 있을 수 있는가?
- 그 변화에 쉽게 대응할 수 있게 작성되어 있는가?

## upgradeLevels() 메서드의 문제점 개선

```java
public class UserService {
    UserDao userDao;

    public void setUserDao(UserDao userDao) {
        this.userDao = userDao;
    }

    public void upgradeLevels() {
        List<User> users = userDao.getAll();

        for(User user : users) {
            Boolean changed = null;

            // 레벨을 파악하는 로직 + 업그레이드 조건 로직이 함께 묶여있다.
            if(user.getLevel() == Level.BASIC && user.getLogin() >= 50) {
                // 다음 레벨로 업그레이드 하기 위한 작업이 함께 담겨있다.
                user.setLevel(Level.SILVER);
                // 맨 뒤에서 update가 필요한지 알려주기 위한 임시 플래그가 섞여있다.
                changed = true;
            }

            // 레벨 개수만큼 if 조건 블록이 반복된다.
            else if(user.getLevel() == Level.SILVER && user.getRecommend() >= 30) {
                user.setLevel(Level.GOLD);
                changed = true;
            }

            else if(user.getLevel() == Level.GOLD) { changed = false; }

            // 로그인 횟수가 충족되지 않을 경우와 새로운 레벨이 추가된 경우는
            // 성격이 다름에도 불구하고 같은 else 문에서 처리된다.
            else { changed = false; }

            if(changed) { userDao.update(user); }
        }
    }
}
```

문제점을 리팩토링 하면 다음과 같다.

{% tabs %}
{% tab title="After" %}
```java
public class UserService {
    // 숫자가 중복되므로 상수로 처리한다.
    public static final int MIN_LOGCOUNT_FOR_SILVER = 50;
    public static final int MIN_RECOMMEND_FOR_GOLD = 30;

    UserDao userDao;

    public void setUserDao(UserDao userDao) {
        this.userDao = userDao;
    }

    // upgradeLevels()에는 레벨을 업그레이드 하는 기본 흐름만 남겨둔다.
    public void upgradeLevels() {
        List<User> users = userDao.getAll();

        for(User user : users) {
            if(canUpgradeLevel(user)) {
                upgradeLevel(user);
            }
        }
    }

    // 업그레이드가 가능한지 알려준다.
    private boolean canUpgradeLevel(User user) {
        Level currentLevel = user.getLevel();

        // 레벨 별로 구분해서 조건을 판단한다.
        switch(currentLevel) {
            case BASIC: return (user.getLogin() >= MIN_LOGCOUNT_FOR_SILVER);
            case SILVER: return (user.getRecommend() >= MIN_RECOMMEND_FOR_GOLD);
            case GOLD: return false;
            // 다룰 수 없는 레벨이 주어지면 예외를 발생시킨다. 
            // 새로운 레벨을 추가하고 로직을 수정하지 않았을 때 참고할 수 있다.
            default: throw new IllegalArgumentException("Unknown Level: " + currentLevel);
        }
    }

    // 실제 업그레이드 작업을 진행한다.
    private void upgradeLevel(User user) {
        if(user.getLevel() == Level.BASIC) user.setLevel(Level.SILVER);
        else if(user.getLevel() == Level.SILVER) user.setLevel(Level.GOLD);

        userDao.update(user);
    }
}
```
{% endtab %}

{% tab title="Before" %}
```java
public class UserService {
    UserDao userDao;

    public void setUserDao(UserDao userDao) {
        this.userDao = userDao;
    }

    public void upgradeLevels() {
        List<User> users = userDao.getAll();

        for(User user : users) {
            Boolean changed = null;

            // 레벨을 파악하는 로직 + 업그레이드 조건 로직이 함께 묶여있다.
            if(user.getLevel() == Level.BASIC && user.getLogin() >= 50) {
                // 다음 레벨로 업그레이드 하기 위한 작업이 함께 담겨있다.
                user.setLevel(Level.SILVER);
                // 맨 뒤에서 update가 필요한지 알려주기 위한 임시 플래그가 섞여있다.
                changed = true;
            }

            // 레벨 개수만큼 if 조건 블록이 반복된다.
            else if(user.getLevel() == Level.SILVER && user.getRecommend() >= 30) {
                user.setLevel(Level.GOLD);
                changed = true;
            }

            else if(user.getLevel() == Level.GOLD) { changed = false; }

            // 로그인 횟수가 충족되지 않을 경우와 새로운 레벨이 추가된 경우는
            // 성격이 다름에도 불구하고 같은 else 문에서 처리된다.
            else { changed = false; }

            if(changed) { userDao.update(user); }
        }
    }
}
```
{% endtab %}
{% endtabs %}

그런데 `upgradeLevel()`의 경우 다음 단계를 확인하고 변경하는 로직이 함께 있는데다 노골적으로 드러나있다. 예외상황에 대한 처리도 없다.

우선, 레벨의 순서와 다음 레벨이 무엇인지 결정하는 일을 `Level` 클래스에 위임하자.


{% tabs %}
{% tab title="After" %}
```java
public enum Level {
    // DB에 저장할 값과 다음 단계의 레벨 정보를 함께 저장한다.
    BASIC(1, SILVER), SILVER(2, GOLD), GOLD(3, null);
    
    private final int value;
    // 다음 단계의 레벨 정보를 스스로 갖고 있도록 Level 타입의 next 변수를 추가한다.
    private final Level next;

    Level(int value, Level next) {
        this.value = value;
        this.next = next;
    }

    public int intValue() {
        return value;
    }

    // 다음 레벨이 무엇인지 알고 싶다면 호출한다.
    // 다음 레벨이 무엇인지 if문으로 일일이 찾을 필요가 없어졌다.
    public Level nextLevel() {
        return this.next;
    }

    public static Level valueOf(int value) {
        switch(value) {
            case 1: return BASIC;
            case 2: return SILVER;
            case 3: return GOLD;
            default: throw new AssertionError("Unknown value: " + value);
        }
    }
}
```
{% endtab %}

{% tab title="Before" %}
```java
public enum Level {
    BASIC(1), SILVER(2), GOLD(3);
    
    private final int value;

    Level(int value) {
        this.value = value;
    }

    public int intValue() {
        return value;
    }

    public static Level valueOf(int value) {
        switch(value) {
            case 1: return BASIC;
            case 2: return SILVER;
            case 3: return GOLD;
            default: throw new AssertionError("Unknown value: " + value);
        }
    }
}
```
{% endtab %}
{% endtabs %}

사용자 정보가 바뀌는 부분은 `UserService`에서 `User`로 옮긴다. 사용자의 내부 정보가 변경되는 것이므로 `User`가 다루는 것이 적절하다.

```java
public class User {
    ...
    Level level;
    int login;
    int recommend;

    public Level getLevel() {
        return level;
    }
	
    public void setLevel(Level level) {
        this.level = level;
    }

    public void upgradeLevel() {
        Level nextLevel = this.level.nextLevel();
        if(nextLevel == null) {
            throw new IllegalStateException(this.level + "은 업그레이드가 불가합니다.");
        }
        else {
            this.level = nextLevel;
        }
    }
}
```

{% tabs %}
{% tab title="After" %}
```java
public class UserService {
    ...
    private void upgradeLevel(User user) {
        // User 클래스로 이동한 upgradeLevel()을 사용한다.
        user.upgradeLevel();
        userDao.update(user);
    }
}
```
{% endtab %}

{% tab title="Before" %}
```java
public class UserService {
    ...
    private void upgradeLevel(User user) {
        if(user.getLevel() == Level.BASIC) user.setLevel(Level.SILVER);
        else if(user.getLevel() == Level.SILVER) user.setLevel(Level.GOLD);

        userDao.update(user);
    }
}
```
{% endtab %}
{% endtabs %}

이제 훨씬 간결하게 정리되었다.

## User 테스트 코드 작성

`User` 클래스에 로직을 추가했으므로 테스트를 만들어두자.

```java
// User 오브젝트는 스프링이 IoC로 관리하는 오브젝트가 아니므로 
// 스프링 테스트 컨텍스트를 사용하지 않아도 된다.
public class UserTest {
    User user;
    
    @Before
    public void setUp() {
        // 따라서 @Autowired로 오브젝트를 가져오는 대신
        // 생성자를 호출해서 테스트할 user 오브젝트를 만든다.
        user = new User();
    }

    @Test()
    public void upgradeLevel() {
        Level[] levels = Level.values();
        for(Level level : levels) {
            // 다음 레벨이 없다면 넘긴다.
            if(level.nextLevel() == null) continue;

            user.setLevel(level);
            // 레벨을 업그레이드 했을 때
            user.upgradeLevel();
            // 다음 레벨로 바뀌는지 확인한다.
            assertThat(user.getLevel(), is(level.nextLevel()));
        }
    }

    // 더 이상 업그레이드할 레벨이 없을 경우 예외가 발생해야 성공한다.
    @Test(expected = IllegalStateException.class)
    public void cannotUpgradeLevel() {
        Level[] levels = Level.values();
        for(Level level : levels) {
            if(level.nextLevel() != null) continue;
            
            user.setLevel(level);
            user.upgradeLevel();
        }
    }

}
```

## UserServiceTest 개선

{% tabs %}
{% tab title="After" %}
```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations="/test-applicationContext.xml")
public class UserServiceTest {
    @Autowired 	UserService userService;	
    @Autowired UserDao userDao;
    
    List<User> users;	
    
    @Before
    public void setUp() {
        users = Arrays.asList(
            new User("bumjin", "박범진", "p1", Level.BASIC, MIN_LOGCOUNT_FOR_SILVER-1, 0),
            new User("joytouch", "강명성", "p2", Level.BASIC, MIN_LOGCOUNT_FOR_SILVER, 0),
            new User("erwins", "신승한", "p3", Level.SILVER, 60, MIN_RECCOMEND_FOR_GOLD-1),
            new User("madnite1", "이상호", "p4", Level.SILVER, 60, MIN_RECCOMEND_FOR_GOLD),
            new User("green", "오민규", "p5", Level.GOLD, 100, Integer.MAX_VALUE)
            );
    }

    @Test
    public void upgradeLevels() {
        userDao.deleteAll();
        for(User user : users) userDao.add(user);
        
        userService.upgradeLevels();
        
        checkLevelUpgraded(users.get(0), false);
        checkLevelUpgraded(users.get(1), true);
        checkLevelUpgraded(users.get(2), false);
        checkLevelUpgraded(users.get(3), true);
        checkLevelUpgraded(users.get(4), false);
    }

    // 어떤 레벨로 바뀔 것인가가 아니라, 다음 레벨로 업그레이드 가능한지 여부를 지정한다.
    private void checkLevelUpgraded(User user, boolean upgraded) {
        User userUpdate = userDao.get(user.getId());
        if(upgraded){
            // 업그레이드가 일어났는지 확인한다.
            assertThat(userUpdate.getLevel(), is(user.getLevel().nextLevel()));
        }
        else {
            // 업그레이드가 일어나지 않았음을 확인한다.
            assertThat(userUpdate.getLevel(), is(user.getLevel()));
        }
    }

    @Test
    public void add() {
        userDao.deleteAll();

        User userWithLevel = users.get(4);
        User userWithoutLevel = users.get(0);
        userWithoutLevel.setLevel(null);

        userService.add(userWithLevel);
        userService.add(userWithoutLevel);

        User userWithLevelRead = userDao.get(userWithLevel.getId());
        User userWithoutLevelRead = userDao.get(userWithoutLevel.getId());

        assertThat(userWithLevelRead.getLevel(), is(userWithLevel.getLevel()));
        assertThat(userWithoutLevelRead.getLevel(), is(Level.BASIC));
    }
}
```
{% endtab %}

{% tab title="Before" %}
```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations="/test-applicationContext.xml")
public class UserServiceTest {
    @Autowired 	UserService userService;	
    @Autowired UserDao userDao;
    
    List<User> users;	
    
    @Before
    public void setUp() {
        users = Arrays.asList(
            new User("bumjin", "박범진", "p1", Level.BASIC, 49, 0),
            new User("joytouch", "강명성", "p2", Level.BASIC, 50, 0),
            new User("erwins", "신승한", "p3", Level.SILVER, 60, 29),
            new User("madnite1", "이상호", "p4", Level.SILVER, 60, 30),
            new User("green", "오민규", "p5", Level.GOLD, 100, 100)
            );
    }

    @Test
    public void upgradeLevels() {
        userDao.deleteAll();
        for(User user : users) userDao.add(user);
        
        userService.upgradeLevels();
        
        checkLevelUpgraded(users.get(0), false);
        checkLevelUpgraded(users.get(1), true);
        checkLevelUpgraded(users.get(2), false);
        checkLevelUpgraded(users.get(3), true);
        checkLevelUpgraded(users.get(4), false);
    }

    private void checkLevelUpgraded(User user, Level expectedLevel) {
        User userUpdate = userDao.get(user.getId());
        assertThat(userUpdate.getLevel(), is(expectedLevel));
    }

    @Test
    public void add() {
        userDao.deleteAll();

        User userWithLevel = users.get(4);
        User userWithoutLevel = users.get(0);
        userWithoutLevel.setLevel(null);

        userService.add(userWithLevel);
        userService.add(userWithoutLevel);

        User userWithLevelRead = userDao.get(userWithLevel.getId());
        User userWithoutLevelRead = userDao.get(userWithoutLevel.getId());

        assertThat(userWithLevelRead.getLevel(), is(userWithLevel.getLevel()));
        assertThat(userWithoutLevelRead.getLevel(), is(Level.BASIC));
    }
}
```
{% endtab %}
{% endtabs %}

## UserLevelUpgradePolicy로 분리하기

레벨을 업그레이드 하는 정책을 유연하게 변경할 수 있도록 개선할 수도 있다. 업그레이드 정책을 `UserService`에서 분리한 뒤 분리한 오브젝트를 DI로 `UserService`에 주입하는 것이다. 정책이 바뀔 때마다 정책 클래스만 바꿔주면 되므로 편리하다.

```java
public interface UserLevelUpgradePolicy {
    boolean canUpgradeLevel(User user);
    void upgradeLevel(User user);
}
```