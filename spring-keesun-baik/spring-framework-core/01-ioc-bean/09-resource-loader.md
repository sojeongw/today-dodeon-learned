# ResourceLoader

- 리소스를 읽어오는 기능을 제공하는 인터페이스.
- 역시 애플리케이션 컨텍스트가 상속한다.

## 리소스 읽어오기

- 파일 시스템에서 읽어오기
- classpath에서 읽어오기
- URL로 읽어오기
- 상대/절대 경로로 읽어오기

```java

@Component
public class AppRunner implements ApplicationRunner {
    @Autowired
    ResourceLoader resourceLoader;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        Resource resource = resourceLoader.getResource("classpath:test.txt");
        System.out.println(resource.exists());
        System.out.println(resource.getDescription());
        System.out.println(Files.readString(Path.of(resource.getURI())));
    }
}
```

```text
hello spring
```

결과는 아래와 같다.

```text
true
class path resource [test.txt]
hello spring
```