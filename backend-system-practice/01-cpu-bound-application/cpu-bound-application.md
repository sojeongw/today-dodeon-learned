# CPU를 극단적으로 사용하는 애플리케이션 만들기

## 프로그램 실행 과정

- 하드디스크
    - 애플리케이션이 위치한다.
- 메모리
    - 하드디스크에 있는 애플리케이션을 실행하면 메모리에 올라간다.
    - 메모리에 올라간 애플리케이션을 프로세스라고 한다.
- CPU
    - 메모리에 있는 프로세스를 실행한다.
    - 어떤 프로세스를 실행할지 고르는 방법을 스케줄링이라고 한다.

하드 디스크는 느리고 CPU는 매우 빠르다. 그런데 CPU가 프로세스 실행 때문에 하드 디스크에 직접 요청하면 아무리 빨라도 느려지게 된다.

그래서 중간에 속도 차이를 줄이기 위에 메모리가 존재한다. 메모리와 CPU 사이의 속도 차이를 줄이기 위해 메모리 보단 빠르고 CPU 보단 느린 캐시 메모리를 사용하기도 한다.

그럼에도 CPU와 하드 디스크가 상호 작용 즉, I/O 해야할 때가 있는데 그동안 다른 프로세스를 실행하면서 최대한 CPU를 효율적으로 사용한다.

## I/O & CPU

I/O는 하드 디스크 뿐만 아니라 DB, 네트워크도 있다.

- I/O Burst
    - 한 프로세스 실행 도중 I/O 하는 시간
- CPU Burst
    - 한 프로세스 실행 도중 CPU에서 실행되는 시간
- I/O Bound 애플리케이션
    - 프로세스가 I/O를 많이 하는 애플리케이션
- CPU Bound 애플리케이션
    - 프로세스가 CPU를 많이 사용하는 애플리케이션

![](../../.gitbook/assets/backend-system-practice/01/screenshot%202022-04-03%20오후%205.29.43.png)

a는 CPU 바운드, b는 I/O 바운드 애플리케이션이다.

## CPU를 많이 사용하는 애플리케이션

- I/O를 적게 사용하고 CPU 연산을 많이 사용하는 애플리케이션
- 대표적인 예로 Hash 연산을 많이 반복하는 애플리케이션을 만들어본다.

```java

@RestController
public class HashController {

    @RequestMapping("/hash/{input}")
    public String getDigest(@PathVariable("input") String input) throws NoSuchAlgorithmException {
        for (int i = 0; i < 100_000; i++) {
            input = getMD5Digest(input);
        }
        return input;
    }

    @RequestMapping("/hello")
    public String hello() {
        return "hello";
    }

    private String getMD5Digest(String input) throws NoSuchAlgorithmException {
        MessageDigest md = MessageDigest.getInstance("MD5");
        md.update(input.getBytes());
        byte[] digest = md.digest();
        String myHash = DatatypeConverter
                .printHexBinary(digest).toUpperCase();

        return myHash;
    }
}
```

```commandline
sudo java -jar cpu-0.0.1-SNAPSHOT.jar 
```

VM 인스턴스에서 서버를 실행한다.

- hello
    - 평균 20ms
- hash
    - 평균 100ms
    - 100 - 20 = 80ms 동안 MD5 해시 연산을 10만 번 수행했다고 볼 수 있다.