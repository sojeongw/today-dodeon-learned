# 빈의 스코프

## 싱글턴

애플리케이션 전반에 걸쳐 빈의 인스턴스가 오직 하나 뿐인 것을 말한다.

```java

@SpringBootApplication
public class DemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }

}
```

```java

@Component
public class AppRunner implements ApplicationRunner {
    @Autowired
    Single single;

    @Autowired
    Proto proto;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        // AppRunner가 불러오는 proto
        System.out.println(proto);
        // Single이 불러오는 proto
        System.out.println(single.getProto());
    }
}
```

```java

@Component
public class Single {
    @Autowired
    Proto proto;

    public Proto getProto() {
        return proto;
    }
}
```

```java

@Component
public class Proto {
}
```

```text
me.whiteship.beanscope.Proto@3c49fab6
me.whiteship.beanscope.Proto@3c49fab6
```

주입 받은 `proto`와 `single` 객체에서 불러온 `proto`의 주소가 같은 것을 확인할 수 있다.

## 프로토타입

매번 새로운 인스턴스를 생성하는 것이다.

- Request
- Session
- WebSocket

```java

@Component
@Scope("prototype")
public class Proto {
}
```

`Proto`에 `Scope`을 prototype으로 정의해놓으면

```text
me.whiteship.beanscope.Proto@28f8e165
me.whiteship.beanscope.Proto@545f80bf
```

이렇게 서로 다른 주소값이 나오게 된다.

```java

@Component
public class AppRunner implements ApplicationRunner {
    @Autowired
    ApplicationContext ctx;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        System.out.println("proto");
        System.out.println(ctx.getBean(Proto.class));
        System.out.println(ctx.getBean(Proto.class));
        System.out.println(ctx.getBean(Proto.class));

        System.out.println("\nsingle");
        System.out.println(ctx.getBean(Single.class));
        System.out.println(ctx.getBean(Single.class));
        System.out.println(ctx.getBean(Single.class));
    }
}
```

```text
proto
me.whiteship.beanscope.Proto@632aa1a3
me.whiteship.beanscope.Proto@20765ed5
me.whiteship.beanscope.Proto@3b582111

single
me.whiteship.beanscope.Single@2899a8db
me.whiteship.beanscope.Single@2899a8db
me.whiteship.beanscope.Single@2899a8db
```

여러 번을 시도해도 역시 같은 결과를 가져온다.

## 프로토타입 빈이 싱글턴 빈을 참조하는 경우

```java

@Component
@Scope("prototype")
public class Proto {
    @Autowired
    Single single;
}
```

- 프로토타입 빈은 항상 새롭지만 안에서 참조하는 싱글턴 빈은 항상 동일하므로 아무 문제가 없다.

## 싱글턴 빈이 프로토타입 빈을 참조하는 경우

- 싱글턴 빈은 한 번만 만들어지므로 이미 프로토타입의 빈이 이미 세팅된 상태다.
- 따라서 프로토타입 빈이 변경되지 않는다.

```java

@Component
public class AppRunner implements ApplicationRunner {
    @Autowired
    ApplicationContext ctx;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        System.out.println("\nproto by single");
        System.out.println(ctx.getBean(Single.class).getProto());
        System.out.println(ctx.getBean(Single.class).getProto());
        System.out.println(ctx.getBean(Single.class).getProto());
    }
}
```

```text
proto by single
me.whiteship.beanscope.Proto@1162410a
me.whiteship.beanscope.Proto@1162410a
me.whiteship.beanscope.Proto@1162410a
```

코드로 테스트 해보니 역시 프로토타입임에도 같은 주소값을 출력한다.

### 해결법

프록시 모드를 설정하면 된다. 기본은 DEFAULT(프록시 모드 사용 안 함)으로 되어있다.

```java

@Component
@Scope(value = "prototype", proxyMode = ScopedProxyMode.TARGET_CLASS)
public class Proto {
    @Autowired
    Single single;
}
```

```text
proto by single
me.whiteship.beanscope.Proto@4a951911
me.whiteship.beanscope.Proto@55b62629
me.whiteship.beanscope.Proto@a53bb6f
```

매번 새로운 인스턴스가 만들어짐을 확인할 수 있다.

- `TARGET_CLASS`
    - 클래스 기반의 프록시로 빈(`Proto`)를 감싸라고 설정하는 것
    - 프록시를 쓰지 않으면 직접 참조하므로 프로토타입을 생성할 때마다 새로 바꿔 줄 여지가 없다.
    - 프록시를 거쳐서 참조하도록 하면 매번 바꿔서 참조할 수 있다.
- 원래 자바 안에 있는 다이내믹 프록시는 인터페이스의 프록시만 만들 수 있다.
    - `CG LIB(Code Generator Library`라는 서드 파티 라이브러가 `TARGET_CLASS` 설정을 보고 클래스도 만들 수 있도록 한다.
    - 만약 인터페이스였다면 `INTERFACES`로 설정해서 인터페이스 기반의 프록시를 썼을 것이다.
- 프록시 빈도 프로토타입의 빈을 상속해서 만들었기 때문에 해당 빈과 타입(여기서는 `Proto` 타입)은 같다.
    - 따라서 의존성 주입이 가능한 것이다.

**Reference**

[프록시 패턴](https://ko.wikipedia.org/wiki/프록시_패턴)

## 싱글턴 객체 사용 시 주의할 점

- 프로퍼티에 담긴 값이 안정적일 것이라고, thread-safe 할 것이라고 생각해서는 안 된다.

```java

@Component
public class Single {
    @Autowired
    Proto proto;

    // 이 값이 그대로일 거라고 보장하지 못한다.
    int value = 0;

    public Proto getProto() {
        return proto;
    }
}
```

- 싱글턴 객체는 ApplicationContext를 만들 때 인스턴스를 생성하게 되어 있다.
    - 따라서 애플리케이션 구동에 시간이 걸릴 수 있다.