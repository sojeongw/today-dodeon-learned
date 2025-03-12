# 스레드 로컬

- 해당 스레드만 접근할 수 있는 특별한 저장소
- 같은 인스턴스의 스레드 로컬 필드에 접근해도 동시성 문제가 없다.

```java

@Slf4j
public class ThreadLocalService {

    private ThreadLocal<String> nameStore = new ThreadLocal<>();

    public String logic(String name) {
        log.info("저장 name = {} -> nameStore = {}", name, nameStore.get());
        nameStore.set(name);
        sleep(1000);

        log.info("조회 nameStore = {}", nameStore.get());
        return nameStore.get();
    }

    private void sleep(int millis) {
        try {
            Thread.sleep(millis);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

- set(), get()으로 값을 저장하고 조회할 수 있다.
- 모두 사용하고 나면 반드시 remove()로 값을 제거해준다.

## 주의사항

- 만약 스레드 로컬 값을 사용 후 제거하지 않고 그냥 두면 WAS처럼 스레드 풀을 사용하는 경우 문제가 발생할 수 있다.
    - 데이터가 살아있는 채로 스레드가 재사용 될 수 있다.