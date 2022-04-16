# 다형성 쿼리

![](../../.gitbook/assets/kimyounghan-orm-jpa/11/screenshot%202021-05-23%20오후%204.27.15.png)

다형적으로 설계한 경우 특수한 기능을 사용할 수 있다.

## 조회 대상을 특정 자식으로 한정하기

![](../../.gitbook/assets/kimyounghan-orm-jpa/11/screenshot%202021-05-23%20오후%204.28.48.png)

- type()
    - DTYPE으로 특정 자식을 조회할 때 사용한다.

## TREAT

![](../../.gitbook/assets/kimyounghan-orm-jpa/11/screenshot%202021-05-23%20오후%204.34.43.png)

- 자식의 타입으로 불러온다.
    - 자바의 다운 캐스팅과 유사하다.
- 상속 구조에서 부모 타입을 특정 자식 타입으로 다룰 때 사용한다.
- FROM, WHERE, SELECT(하이버네이트)에 사용한다.
