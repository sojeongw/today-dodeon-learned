# 다형성 쿼리

![](../../.gitbook/assets/kimyounghan-orm-jpa/11/screenshot%202021-05-23%20오후%204.27.15.png)

다형적으로 설계한 경우 특수한 기능을 사용할 수 있다.

## 조회 대상을 특정 자식으로 한정하기

![](../../.gitbook/assets/kimyounghan-orm-jpa/11/screenshot%202021-05-23%20오후%204.28.48.png)

Item 중에 Book, Movie를 조회한다면 위와 같이 조회할 수 있다.

## TREAT

- 자바의 타입 캐스팅과 유사하다.
- 상속 구조에서 부모 타입을 특정 자식 타입으로 다룰 때 사용한다.
- FROM, WHERE, SELECT(하이버네이트)에 사용한다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/11/screenshot%202021-05-23%20오후%204.34.43.png)

부모인 Item과 Book이 있다면 위와 같이 사용한다.