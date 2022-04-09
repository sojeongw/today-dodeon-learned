# 값 타입과 불변 객체

- 값 타입은 복잡한 객체를 단순화하기 위해 만든 개념이다.
    - 따라서 값 타입을 단순하고 안전하게 다룰 수 있어야 한다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/09/screenshot%202021-03-31%20오후%208.37.46.png)

- 임베디드 타입 등 값 타입을 여러 Entity에서 공유하면 위험하다.
- 회원 1과 회원 2가 같은 city를 갖고 있을 때 회원 1의 city를 바꾸면 회원 2의 city도 바뀐다.

{% tabs %} {% tab title="JpaMain.java" %}

```java
public class JpaMain {

    public static void main(String[] args) {
        // 같은 Address 값을 세팅한다.
        Address address = new Address("city", "street", "10012");

        Member member = new Member();
        member.setUsername("member1");
        member.setAddress(address);
        em.persist(member);

        Member member2 = new Member();
        member2.setUsername("member2");
        member2.setAddress(address);
        em.persist(member2);

        tx.commit();
    }
}
```

{% endtab %} {% endtabs %}

![](../../.gitbook/assets/kimyounghan-orm-jpa/09/screenshot%202021-03-31%20오후%209.11.26.png)

일단 같은 주소를 회원 1, 2에게 넣어준다.

{% tabs %} {% tab title="JpaMain.java" %}

```java
public class JpaMain {

    public static void main(String[] args) {
        Address address = new Address("city", "street", "10012");

        Member member = new Member();
        member.setUsername("member1");
        member.setAddress(address);
        em.persist(member);

        Member member2 = new Member();
        member2.setUsername("member2");
        member2.setAddress(address);
        em.persist(member2);

        // 첫번째 멤버의 주소만 변경하려고 한다.
        member.getAddress().setCity("new City");

        tx.commit();
    }
}
```

{% endtab %} {% endtabs %}

그리고 나서 회원 1의 주소를 바꾼다면?

![](../../.gitbook/assets/kimyounghan-orm-jpa/09/screenshot%202021-03-31%20오후%209.15.10.png)

업데이트 쿼리가 두 번 나간 것을 볼 수 있다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/09/screenshot%202021-03-31%20오후%209.14.39.png)

실제 데이터도 회원 1, 2 모두가 바뀌었다. 이런 버그는 찾기가 굉장히 어렵다.

- 값 타입의 실제 인스턴스 값을 공유하는 것은 매우 위험하다.
- 일부러 데이터를 공유해서 사용하고 싶다면 임베디드 타입이 아니라 Entity로 만들어서 공유해야 한다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/09/screenshot%202021-03-31%20오후%208.37.50.png)

대신, 값(인스턴스)를 복사해서 사용해야 한다.

{% tabs %} {% tab title="JpaMain.java" %}

```java
public class JpaMain {

    public static void main(String[] args) {
        Address address = new Address("city", "street", "10012");

        Member member = new Member();
        member.setUsername("member1");
        member.setAddress(address);
        em.persist(member);

        // 값을 복사해서 사용한다.
        Address address2 = new Address(address.getCity(), address.getStreet(), address.getZipcode());

        Member member2 = new Member();
        member2.setUsername("member2");
        member2.setAddress(address2);
        em.persist(member2);

        // 이제 회원 1만 update 쿼리가 나간다.
        member.getAddress().setCity("new City");

        tx.commit();
    }
}
```

{% endtab %} {% endtabs %}

![](../../.gitbook/assets/kimyounghan-orm-jpa/09/screenshot%202021-03-31%20오후%209.19.16.png)

값을 복사해서 사용해야 의도대로 데이터가 돌아간다.

## 객체 타입의 한계

- 항상 값을 복사해서 사용하면 공유 참조 때문에 발생하는 부작용을 피할 수 있다.
- 그런데 임베디드 타입 등 직접 정의한 타입은 자바 기본 타입이 아니라 **객체 타입**이다.
    - primitive 같은 값 타입은 `=`으로 할당해도 복사해서 들어가서 두 변수를 한 번에 수정하는 사이드 이펙트가 발생하지 않는다.
    - 객체 타입은 참조 값을 직접 가지므로 둘 다 값이 바뀌는 사이드 이펙트를 막을 방법이 없다.
- 즉, 객체의 공유 참조는 피할 수 없다.

{% tabs %} {% tab title="JpaMain.java" %}

```java
public class JpaMain {

    public static void main(String[] args) {
        Address address = new Address("city", "street", "10012");

        Member member = new Member();
        member.setUsername("member1");
        member.setAddress(address);
        em.persist(member);

        Address address2 = new Address(address.getCity(), address.getStreet(), address.getZipcode());

        Member member2 = new Member();
        member2.setUsername("member2");
        // 실수로 기존의 address를 삽입한다면?
        member2.setAddress(address);
        em.persist(member2);

        member.getAddress().setCity("new City");

        tx.commit();
    }
}
```

{% endtab %} {% endtabs %}

만약 실수로 복사한 값 대신 이전 값을 집어넣는다면, 컴파일 단계에서 이 문제를 짚어낼 수 없다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/09/screenshot%202021-03-31%20오후%209.25.32.png)

기본 타입은 값을 복사하기 때문에 문제가 없지만, 객체 타입은 참조를 전달하기 때문에 변경하면 둘 다 반영되는 부작용이 발생한다.

## 불변 객체

- 생성 시점 이후 절대 값을 변경할 수 없는 객체
    - Integer, String은 자바가 제공하는 대표적인 불변 객체다.
- 생성자로만 값을 설정하고 수정자(setter)를 만들지 않으면 된다.
    - 객체 타입을 수정할 수 없게 만들면 부작용을 원천 차단할 수 있다.
- 값 타입은 불변 객체로 설계해야 한다.

{% tabs %} {% tab title="JpaMain.java" %}

```java
public class JpaMain {

    public static void main(String[] args) {
        Address address = new Address("city", "street", "10012");

        Member member = new Member();
        member.setUsername("member1");
        member.setAddress(address);
        em.persist(member);

        // city를 바꾸고 싶다면 setCity() 대신 
        // Address 객체를 새로 만들어서 통으로 갈아 끼운다.
        Address newAddress = new Address("new city", address.getStreet(), address.getZipcode());
        member.setAddress(newAddress);

        tx.commit();
    }
}
```

{% endtab %} {% endtabs %}

값을 바꾸고 싶다면 새롭게 객체를 생성해서 넣어주어야 한다.