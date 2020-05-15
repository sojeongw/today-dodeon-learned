# JVM 구조

![](../../.gitbook/assets/the-java/01/스크린샷%202020-07-05%20오후%208.28.42.png)

## 클래스 로더 시스템

`.class`에서 바이트 코드를 읽고 메모리에 저장한다.

- 로딩
    - 클래스를 읽어오는 과정
- 링크
    - 레퍼런스를 연결하는 과정
- 초기화
    - static 값 초기화 및 변수 할당
    
```java
public class App {
    static String myName;
    
    static {
        myName = "dodeon";
    }
}

public class Dodeon {
    // static으로 선언하면 이렇게 바로 접근할 수 있다.
    public void work() {
        App.myName;
    }
}
```
    
## 메모리

클래스 수준의 정보를 저장하는 공유 자원이다. 힙과 메서드는 여러 곳에서 공유하지만 스택, PC, 네이티브 메서드 스택은 해당 스레드에서만 공유한다.

- 클래스 이름
- 부모 클래스 이름
- 메서드
- 변수

### 힙 영역

객체 즉, 모든 인스턴스 변수를 저장하는 공유 자원이다. new 키워드로 인스턴스가 생성되면 해당 인스턴스의 정보를 메모리의 낮은 주소에서 높은 주소로 저장한다.


### 메서드 영역

클래스에 대한 정보와 클래스 변수(static variable)이 저장되는 영역이다. 자바 애플리케이션에서 특정 클래스가 실행되면 해당 클래스의 클래스 파일인 `.class`를 읽어 해당 정보를 메서드 영역에 저장한다.

### 스택 영역

스레드마다 런타임 스택을 생성하고 그 안에 메서드 호출을 스택 프레임이라는 블럭으로 쌓는다. 스레드를 종료하면 런타임 스택도 사라진다.

#### 스택 프레임

method call을 의미한다.

![](../../.gitbook/assets/the-java/01/스크린샷%202020-07-05%20오후%209.38.29.png)

에러가 나면 에러가 쭉 쌓이는 걸 볼 수 있다. 이것이 바로 스택 안에 호출된 메서드가 쌓인 것이다.

### PC(Program Counter) 레지스터

각 스레드 안에서 현재 실행할 스택 프레임을 가리키는 포인터가 생성되는 곳이다.

스레드마다 생성한 스택에 메서드가 쭉 쌓이면 현재 어디를 가리키는지 저장하는 것이므로 스택과 같이 해당 스레드에 국한된다.

### 네이티브 메서드 스택

네이티브 메서드를 호출할 때 쌓이는 별도의 스택으로, 역시 스레드마다 생성된다.

네이티브 메서드 라이브러리를 쓰려면 네이티브 메서드 인터페이스를 통해야 하고, 이것을 쓰려면 네이티브 메서드 스택에 쌓아야 한다.

[런타임 시의 JVM] (https://javapapers.com/core-jav/java-jvm-run-time-data-areas)

## 실행 엔진

- 인터프리터
    - 바이트 코드를 한 줄씩 실행
- JIT(Just In Time) 컴파일러
    - 인터프리터의 효율을 높이기 위해 사용
    - 인터프리터가 JIT 컴파일러를 이용해 반복되는 바이트 코드를 네이티브 코드로 변환함
    - 인터프리터는 그 네이티브 코드로 컴파일된 코드를 바로 사용함
- GC(Garbage Collector)
    - 더 이상 참조되지 않는 객체를 모아서 정리함

## JNI(Java Native Interface)

자바 애플리케이션에서 C, C++, 어셈블리로 작성된 함수를 사용할 수 있는 방법을 제공하며 Native 키워드를 사용해 메서드를 호출한다.

[JNI Example] (https://medium.com/@bschlining/a-simple-java-native-interface-jni-example-in-java-and-scala-68fdafe76f5f)

## 네이티브 메소드 라이브러리

메서드에 Native라는 키워드가 붙어있고 그 구현을 자바가 아닌 C나 C++로 한 것을 말한다.

```java
public class App {
    public static void main(String[] args) {
        Thread.currentThread();
    }
}
```

예를 들어 `Thread`에는 `currentThread()`메서드가 있는데,

![](../../.gitbook/assets/the-java/01/스크린샷%202020-07-05%20오후%209.51.47.png)

들어가보면 `native`가 붙어있어 네이티브 메서드인 것을 알 수 있다. currentThread()는 사용할 수 있도록 연결해주는 인터페이스 즉, JNI이고 실제 구현은 C로 되어있는 네이티브 메서드 라이브러리이다.


**Reference**

[JVM 작동 방식] (https://www.geeksforgeeks.org/jvm-works-jvm-architecture/)

[JVM 아키텍처] (https://dzone.com/articles/jvm-architecture-explained)

[JVM Internals] (http://blog.jamesdbloom.com/JVMInternals.html)