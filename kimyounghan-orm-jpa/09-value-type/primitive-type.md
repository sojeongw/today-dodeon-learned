# 기본값 타입

`String name`, `int age`와 같은 값을 말한다.

- 생명 주기가 Entity에 의존한다.
    - 회원을 삭제하면 이름, 나이도 함께 삭제된다.
- 값 타입은 의존하면 안된다.
    - 회원 이름 변경 시 다른 회원의 이름도 변경되면 안된다.
    
## 참고

자바의 기본 타입(int, double 등)은 절대로 공유되지 않는다. 항상 값을 복사해서 사용한다.

```java
public class ValueMain {

  public static void main(String[] args) {
    int a = 10;
    // a에 있는 값이 복사가 되어서 b에 할당된다.
    // a와 b는 완전히 따로 저장 공간을 가지고 있는 것이다.
    // 즉, 공유되지 않는다.
    int b = a;

    a = 20;

    // b는 20으로 변경되지 않는다.
    System.out.println("a = " + a);
    System.out.println("b = " + b);
  }

}
```

기본 타입은 공유되지 않기 때문에 사이드 이펙트가 없다. 즉, 내 이름을 바꿨다고 다른 사람의 이름이 바뀌지는 않는다. 클래스는 레퍼런스를 끌고 가기 때문에 공유가 된다.

```java
public class ValueMain {

  public static void main(String[] args) {
    Integer c = 10;
    // c의 참조값만 넘어간다.
    Integer d = c;

    // setValue()는 없는 메서드지만 있다고 가정하면,
    // 레퍼런스를 넘겨서 같은 인스턴스를 공유하기 때문에 둘 다 값이 바뀐다.
    // 하지만 값을 변경할 수 있는 방법이 없으므로 역시 사이드 이펙트가 일어나지 않는다.
    c.setValue(20);

    System.out.println("c = " + c);
    System.out.println("d = " + d);
  }

}
```

Integer 같은 래퍼 클래스나 String 같은 특수한 클래스는 공유 가능한 객체지만, 변경 자체를 불가능하게 만들어 사이드 이펙트를 막는다.