# 클래스 로더

![](study/today-dodeon-learned/.gitbook/assets/keesunbaik-the-java/01/스크린샷%202020-07-05%20오후%2010.21.37.png)

클래스 로더는 로딩, 링크, 초기화 순으로 진행된다.

## 로딩

클래스 로더가 `.class` 파일을 읽고 그 내용에 따라 적절한 바이너리 데이터를 만들고 메서드 영역에 저장한다.

Bootstrap, Extension, Application 이라는 부모, 자식의 계층형 구조로 되어있다.

### 메서드 영역에 저장하는 데이터

- FQCN(Fully Qualified Class Name)
    - full 패키지 경로, 클래스 로더, 클래스 이름을 포함한 내용을 의미함
- 클래스
- 인터페이스
- ENUM
- 메서드
- 변수

로딩이 끝나면 해당 클래스 타입의 class 객체를 생성해 heap 영역에 저장한다.

```java
public class Dodeon {
    public void work() {
        // 클래스가 로딩되면 해당 클래스 타입의 인스턴스가 'class'에 저장되어 이렇게 static하게 접근할 수 있다.
        Dodeon.class;
        
        // 인스턴스를 만든다면 'getClass'로 그 객체에 접근할 수 있다.
        Dodeon dodeon = new Dodeon();
        dodeon.getClass();
    }
}
```

위의 코드와 같이 클래스를 만들면 그 데이터가 heap 영역에 저장되어 모든 스레드에서 공유할 수 있다.

## 링크

Verify, Prepare, Resolve(Optional) 세 단계로 나눠져 있다.

### Verification

`.class` 파일이 유효한지 체크한다. 만약 내가 `.class`을 마음대로 조작하면 실행할 수 없게 된다.

### Preparation

클래스 변수(static 변수)와 기본값에 필요한 메모리를 준비하는 과정이다.

### Resolve

심볼릭 메모리 레퍼런스를 메서드 영역에 있는 실제 레퍼런스로 교체한다. optional한 과정이라 이때 발생할 수도 있고 실제 사용할 때 발생할 수도 있다.

```java
public class Book {
    ...
}
public class App {
    // 아래의 book은 링크하는 과정에서 심볼릭 레퍼런스다. 
    // 즉, 이 코드를 읽었다고 해도 실제 레퍼런스를 가리키고 있지는 않다. 그저 논리적인 레퍼런스다.
    Book book = new Book();

    public static void main(String[] args) {
        ...
    }
}
```

## 초기화

Preparation 과정에서 준비한 static 변수의 값을 할당한다. static 블럭이 있다면 이때 실행된다.

```java
public class App {
    // 이 단계가 되어서야 static 변수에 'dodeon'이라는 값을 할당한다.
    static String name = "dodeon";
}
```

## 클래스 로더의 구조

클래스 로더는 계층 구조로 이루어져 있으며 기본적으로 세 가지 클래스 로더가 제공된다.

```java
public class App {
    public static void main(String[] args) {
        // 현재 사용하는 클래스 로더를 불러올 수 있다.
        ClassLoader classLoader = App.class.getClassLoader();

        // 계층형 구조이므로 부모를 불러올 수 있다.
        System.out.println(classLoader);
        System.out.println(classLoader.getParent());
        System.out.println(classLoader.getParent().getParent());
    }
}
```

![](study/today-dodeon-learned/.gitbook/assets/keesunbaik-the-java/01/스크린샷%202020-07-05%20오후%2011.05.21.png)

현재 클래스의 클래스 로더는 `AppClassLoader`이며 부모 클래스 로더는 `PlatformClassLoader`임을 알 수 있다. 그 위의 부모는 존재는 하지만 네이티브로 구현되어 있어 자바 코드로 참조할 수가 없으므로 `null`이 출력된다.

![](study/today-dodeon-learned/.gitbook/assets/keesunbaik-the-java/01/스크린샷%202020-07-05%20오후%2011.07.54.png)

![](study/today-dodeon-learned/.gitbook/assets/keesunbaik-the-java/01/스크린샷%202020-07-05%20오후%2011.07.29.png)

`PlatformClassLoader`에 들어가면 내용을 직접 볼 수 있다.

### 부트 스트랩 클래스 로더

`JAVA_HOME\lib`에 있는 코어 자바 API를 제공한다. 최상위 우선 순위를 가진 클래스 로더이다.

### 플랫폼 클래스 로더

`JAVA_HOME/lib/ext` 폴더, `java.ext.dirs` 시스템 변수에 해당하는 클래스를 읽는다.

### 애플리케이션 클래스 로더

애플리케이션 클래스 패스에서 클래스를 읽는다.

- 애플리케이션 클래스 패스 
    - 애플리케이션을 실행할 때 주는 `-classpath` 옵션 또는 `java.class.path` 환경 변수의 값에 해당하는 위치