# 싱글톤 패턴

애플리케이션이 시작될 때 어떤 클래스가 최초에 딱 한 번만 메모리를 할당하고, 해당 메모리에 인스턴스를 만들어 사용하는 패턴이다.

즉, 하나의 인스턴스만 생성해서 사용하는 디자인 패턴을 말한다. 인스턴스가 필요할 때 같은 인스턴스를 새로 만들지 않고 원래 있던 인스턴스를 활용하는 것이다.

따라서 생성자를 여러 번 호출해도 실제로 생성되는 객체는 하나다. 맨 처음에 한 번 만들면 그 이후에 호출된 생성자는 이미 만들어 둔 객체를 반환한다.

java에서는 생성자를 `private`으로 선언해서 다른 곳에서 만들지 못하도록 한다. 사용하려면 `getInstance()` 메소드를 통해야 한다.

## 사용하는 이유

객체를 생성할 때마다 메모리를 할당받아야 하는데, 이때 메모리 낭비를 방지하기 위해 사용한다. 싱글톤으로 구현한 인스턴스는 전역이기 때문에 다른 클래스들이 서로 데이터를 공유할 수 있다.

## 적용 사례

주로 공통된 객체를 공유해서 사용해야 할 때 적용한다. 또는 인스턴스가 절대적으로 한 개만 존재하는 걸 보증하고 싶을 때 사용한다.

> ex) DB 커넥션 풀, 스레드 풀, 캐시, 로그 기룩 객체 등

## 단점

[개방-폐쇄 원칙](../README.md)에 따르면 객체는 확장(상속)에는 열려 있고 수정에는 닫혀 있어야 한다. 하지만 싱글톤 인스턴스가 많은 데이터를 공유한다면 다른 클래스끼리 결합도가 높아진다. 결합도가 높아지면 수정, 즉 유지 보수가 어렵고 테스트 하기도 힘들다.

멀티 스레드 환경에서 동기화 처리를 하지 않으면 인스턴스가 2개 생성되는 문제가 발생할 수도 있다.

따라서 반드시 싱글톤이 필요한 게 아니면 지양하는 것이 좋다.

## 멀티 스레드 환경에서 안전하게 싱글톤 만들기
### Lazy Initialization

```java
public class ThreadSafe_Lazy_Initialization {
    // static으로 인스턴스 변수를 만든다.
    private static ThreadSafe_Lazy_Initialization instance;
    // 생성자를 private으로 지정해 외부에서 생성하는 것을 막는다.
    private ThreadSafe_Lazy_Initialization() {}
    // synchronized 동기화로 스레드를 안전하게 만든다.
    public static synchronized ThreadSafe_Lazy_Initialization getInstance() {
        if(instance == null) {
            instance = new ThreadSafe_Lazy_Initialization();
        }
        return instance;
    }
}
```

이 방법은 `synchronized`가 큰 성능 저하를 유발하기 때문에 권장하지 않는다.

### Lazy Initialization + Double-checked Locking

```java
public class ThreadSafe_Lazy_Initialization {
    private volatile static ThreadSafe_Lazy_Initialization instance;

    private ThreadSafe_Lazy_Initialization() {}

    public static ThreadSafe_Lazy_Initialization getInstance() {
        // 조건문으로 인스턴스 존재 여부를 확인한 뒤
        if(instance == null) {
            synchronized (ThreadSafe_Lazy_Initialization.class) {
                // 두 번째 조건문에서 synchronized로 동기화를 시켜 인스턴스를 생성한다.
                if(instance == null) {
                    instance = new ThreadSafe_Lazy_Initialization();
                }
            }
        }
    }   
}
```

이 방법은 스레드를 안전하게 만들면서 처음 생성하고 난 뒤에는 `synchronized`를 실행하지 않으므로 성능 저하를 완화시킬 수 있다. 하지만 여전히 완벽한 방법은 아니다.

### Initialization on demand holder idiom (holder에 의한 초기화)

```java
public class Something {
    private Something() {
    
    }

    // 클래스 안에 클래스(holder)를 만든다.
    // 클래스 로더가 실행될 때 static이기 때문에 클래스 로딩 시점에 딱 한 번만 호출한다.
    private static class LazyHolder {
        // final로 값이 다시 할당되지 않도록 한다.
        public static final Something INSTANCE = new Something();
    }

    public static Something getInstance() {
        return LazyHolder.INSTANCE;
    }
}
```

2번 방법처럼 개발자가 직접 동기화에 손을 대면 프로그램 구조가 복잡해지고 비용 문제가 발생한다. 코드 자체가 정확하지 못할 때도 많다.

따라서 위처럼 JVM에서 클래스를 초기화 하는 과정에서 보장되는 원자적 특성을 이용한다. 싱글톤의 초기화 문제에 대한 책임을 JVM에게 떠넘기는 것이다. 이는 실제로 싱글톤 패턴에 가장 많이 사용되는 방법이다.