# 값 타입의 비교

값 타입은 인스턴스가 달라도 그 안에 있는 값이 같으면 같은 것으로 봐야한다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/09/screenshot%202021-04-03%20오후%201.11.47.png)

하지만 `int a`와 `int b`는 서로 비교하면 `true`, 객체인 `Address a`, `Address b`는 `false`로 나온다. 객체는 참조값으로 비교하기 때문이다.

## 동일성(identity) 비교 

- 인스턴스의 참조 값을 비교한다.
- `==`을 사용한다.

## 동등성(equivalence) 비교

- 인스턴스의 값을 비교한다.
- `equals()`를 사용한다.

따라서 값 타입은 `a.equals(b)`를 사용해서 동등성 비교를 해야 한다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/09/screenshot%202021-04-03%20오후%201.35.09.png)

그런데 `equals()`로 비교해도 여전히 `false`로 나온다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/09/screenshot%202021-04-03%20오후%201.35.22.png)

`equals()` 자체가 `==` 비교를 하기 때문이다. 그래서 override 해서 사용한다.

```java
class Address {
  @Override
  public boolean equals(Object o) {
    if (this == o) {
      return true;
    }
    if (o == null || getClass() != o.getClass()) {
      return false;
    }
    Address address = (Address) o;
    return Objects.equals(city, address.city) && Objects
        .equals(street, address.street) && Objects.equals(zipcode, address.zipcode);
  }

  // equals()를 구현하면 hashCode()도 만들어줘야 한다. 해시 맵 등의 컬렉션에서 효율적으로 사용할 수 있다.
  @Override
  public int hashCode() {
    return Objects.hash(city, street, zipcode);
  }
}
```

`equals()`를 만들 때는 웬만하면 자동으로 생성해주는 코드를 사용한다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/09/screenshot%202021-04-03%20오후%201.39.58.png)

재정의 해주면 `true`로 나오게 된다.

