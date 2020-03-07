# 스프링 AOP

아래와 같은 코드가 있다고 생각해보자.

 ```java
class A {
    method a() {
        pre();
        aaa();
        post();
    }
}
class B {
    method b() {
        pre();
        bbb();
        post();
    }
}
class C {
    method c() {
        pre();
        bbb();
        post();
    }
}
```

모든 클래스는 `pre()`와 `post()`를 공통으로 가지고 있다. 이때 이 두 메서드에 수정이 필요하다면 세 클래스 모두 건드려야 한다. 

```java
class A {
    method a() {
        aaa();
    }
}
class B {
    method b() {
        bbb();
    }
}
class C {
    method c() {
        bbb();
    }
}
class PrePro {
    method prePro(JoinPoint point) {
        pre();
        point.execute();
        post();
    }
}
```

그래서 공통된 메서드는 별도의 클래스로 빼낸다. 이게 바로 AOP다.

## AOP 구현 방법

### 컴파일

> A.java ----> (AOP) ----> A.class

A.java 파일을 컴파일 하면 A.class 파일이 생긴다. 컴파일을 하는 과정에서 AOP를 끼워넣는 것이다. `AspectJ`가 이 방법을 지원한다.

### 바이트코드 조작

> A.java ----> A.class ----> (AOP) ----> 메모

클래스 로더가 A.class 파일을 읽어오면서 메모리에 올릴 때 조작하는 방법이다. 

클래스 파일의 코드 상엔 아무것도 없는데 클래스를 로딩하는 시점에 메모리 상에서 추가하고 싶은 코드가 들어가는 것이다. 역시 `AspectJ`가 제공한다.

### 프록시 패턴

스프링 AOP가 사용하는 방법이다.

**Reference**

[프록시 패턴](https://refactoring.guru/design-patterns/proxy)