# 코드 커버리지 측정법

```java
public class Moim {
    int maxNumberOfAttendees;
    int numberOfEnrollment;
    
    public boolean isEnrollmentFull() {
        if(maxNumberOfAttendees == 0) {
            return false;
        }
        if(numbefOfEnrollment < maxNumberOfAttendees) {
            return false;
        }
        return true;
    }
}
```

```java
public class MoimTest {
    @Test
    public void isFull() {
        Moim moim = new Moim();
        moim.maxNumberOfAttendees = 100;
        moim.numbefOfEnrollment = 10;
        Assert.assertFalse(moim.isEnrollmentFull());
    }   
}
```

코드 커버리지란 테스트 코드가 내 소스 코드의 얼마큼을 커버하는지 즉, 테스트하는지를 의미한다.

![](../../.gitbook/assets/the-java/02/스크린샷%202020-07-05%20오후%2011.42.38.png)

![](../../.gitbook/assets/the-java/02/스크린샷%202020-07-05%20오후%2011.44.22.png)

위의 코드를 `Jacoco`로 검사하면 내 코드가 얼마나 테스트되었는지 보여준다. 이 정보는 바이트코드를 이용한 것이다.

### 바이트코드 조작 라이브러리

[ASM](https://asm.ow2.io/)

[Javassist](https://www.javassist.org/)

[ByteBuddy](https://bytebuddy.net/)

여기서는 `ByteBuddy`를 이용해 바이트코드 조작을 실습한다.

## 아무것도 없는 모자에서 토끼를 꺼내는 마술

```java
public class Hat {

    public String pullOut() {
        return "";
    }
}

```

```java
public class Magician {

    public static void main(String[] args) {
        System.out.println(new Hat().pullOut());
    }
}

```

여기서 어떻게 모자에서 `pullOut()`을 하면 토끼가 나오도록 할 수 있을까?

```java
public class Magician {

    public static void main(String[] args) {
        try {
            // 바이트코드를 조작해서 저장한다.
            new ByteBuddy().redefine(Moja.class)
                .method(named("pullOut")).intercept(FixedValue.value("Rabbit!"))
                .make().saveIn(new File("프로젝트경로/target/classes/"));
        } catch (IOException e) {
            e.printStackTrace();
        }

        // 이때 아래 라인은 주석 처리해둔다. Hat이 먼저 로딩되기 때문이다.
        // 위의 바이트코드 조작을 먼저 실행한 뒤 아래를 실행하면 'Rabbit!'이 출력된다.
//        System.out.println(new Hat().pullOut());
    }
}
```

![](../../.gitbook/assets/the-java/02/스크린샷%202020-07-06%20오전%2012.00.06.png)
