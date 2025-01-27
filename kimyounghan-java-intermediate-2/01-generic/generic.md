# 제네릭

## 제네릭이 필요한 이유

```java
public class IntegerBox {
    private Integer value;

    public void set(Integer value) {
        this.value = value;
    }

    public Integer get() {
        return value;
    }
}

public class StringBox {
    private String value;

    public void set(String object) {
        this.value = object;
    }

    public String get() {
        return value;
    }
}
```

- 단순히 set, get 하는 로직인데 타입이 달라서 중복으로 만들어야 한다.

```java
public class ObjectBox {
    private Object value;

    public void set(Object object) {
        this.value = object;
    }

    public Object get() {
        return value;
    }
}
```

```java
public class BoxMain2 {
    public static void main(String[] args) {
        ObjectBox integerBox = new ObjectBox();
        integerBox.set(10);
        Integer integer = (Integer) integerBox.get(); // Object -> Integer 캐스팅
        System.out.println("integer = " + integer);

        ObjectBox stringBox = new ObjectBox();
        stringBox.set("hello");
        String str = (String) stringBox.get(); // Object -> String 캐스팅
        System.out.println("str = " + str);

        // 잘못된 타입의 인수 전달시
        integerBox.set("문자100");
        Integer result = (Integer) integerBox.get(); // String -> Integer 캐스팅 예외
        System.out.println("result = " + result);
    }
}
```

- 중복을 해결하기 위해 Object로 선언한다.
- 하지만 get() 할 때마다 직접 다운 캐스팅을 해줘야 하고 잘못 캐스팅 했을 때 예외가 발생한다.
- 원하는 타입을 정확하게 지정해서 받을 수 없다.
- 코드 재사용성은 높아졌으나 타입 안정성이 떨어진다.

## 제네릭 적용

```java
public class GenericBox<T> {
    private T value;

    public void set(T value) {
        this.value = value;
    }

    public T get() {
        return value;
    }
}
```

```java
public class BoxMain3 {
    public static void main(String[] args) {
        GenericBox<Integer> integerBox = new GenericBox<Integer>(); // 생성 시점에 T의 타입 결정
        integerBox.set(10);
        // integerBox.set("문자100"); // Integer 타입만 허용, 컴파일 오류

        Integer integer = integerBox.get(); // Integer 타입 반환 (캐스팅 X)
        System.out.println("integer = " + integer);

        GenericBox<String> stringBox = new GenericBox<String>();
        stringBox.set("hello"); // String 타입만 허용
        String str = stringBox.get(); // String 타입만 반환
        System.out.println("str = " + str);

        GenericBox<Double> doubleBox = new GenericBox<Double>();
        doubleBox.set(10.5);
        Double doubleValue = doubleBox.get();
        System.out.println("doubleValue = " + doubleValue);

        // 타입 추론: 생성하는 제네릭 타입 생략 가능
        GenericBox<Integer> integerBox2 = new GenericBox<>();
    }
}
```

- 생성 시점에 원하는 타입을 결정할 수 있다.

### 제네릭 타입

- 클래스, 인터페이스를 정의할 때 타입 매개변수를 사용하는 것
- `GenericBox<T>`를 제네릭 타입이라고 한다.

### 타입 매개변수

- 제네릭 타입이나 메서드에서 사용되는 변수. 실제 타입으로 대체된다.
- `T`를 타입 매개변수라고 한다.

### 타입 인자

- 제네릭 타입을 사용할 떄 제공되는 실제 타입
- `GenericBox<Integer>`의 `Integer`를 일컫는다.

### 명명 관례

- E
    - Element
- K
    - Key
- N
    - Number
- T
    - Type
- V
    - Value
- S,U,V etc.
    - 2nd, 3rd, 4th types

### 기타

```java
class Data<K, V> {
}
```

- 여러 타입 매개변수를 선언할 수 있다.
- 타입 인자로 기본형은 상요할 수 없다. 대신 래퍼 클래스를 사용한다.

### raw 타입

```java
GenericBox integerBox = new GenericBox();
```

- `< >`로 사용 시점에 타입을 지정하지 않는 것
    - Object로 사용된다.
- 낮은 자바 버전과 호환하기 위해 어쩔 수 없이 만들어졌다. 사용하지 않는 게 좋다.