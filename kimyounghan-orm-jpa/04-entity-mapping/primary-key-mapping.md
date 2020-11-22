# 기본 키 매핑

## 직접 할당

`@Id`만 사용한다.

## 자동 생성

`@GeneratedValue`를 추가한다.

### IDENTITY

![](../../.gitbook/assets/kimyounghan-orm-jpa/04/screenshot%202021-03-16%20오후%203.25.38.png)

- 기본 키 생성을 데이터베이스에 위임한다.
- 주로 MySQL, PostgreSQL, SQL Server, DB2에서 사용한다.
    - ex. MySQL의 AUTO_INCREMENT

JPA는 보통 트랜잭션 커밋 시점에 insert SQL을 실행한다. AUTO_INCREMENT는 데이터베이스에 insert SQL을 실행한 뒤에 ID 값을 알 수 있다. em.persist() 시점에 즉시
insert SQL을 실행하고 DB에서 식별자를 조회한다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/03/스크린샷%202021-03-13%20오후%206.15.26.png)

auto increment의 Id 값은 DB에 데이터가 실제 들어가야 알 수 있다. 그런데 영속성 컨텍스트에서 관리되려면 PK 값이 필요하다. 그림에서도 보듯 1차 캐시의 Id 값에 PK가 들어가야 한다.

그래서 JPA는 identity 전략에서만 `em.persist()`로 호출하자 마자 바로 insert 쿼리를 날려버린다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/04/screenshot%202021-03-16%20오후%205.17.34.png)

코드를 실행해보면 `persist()`시에 insert 쿼리가 나가고 id를 조회할 수 있는 걸 확인할 수 있다. jdbc 드라이버에 insert 쿼리 직후 바로 리턴을 받을 수 있는 기능이 있어서, id 조회 때는
select를 치지 않는다.

전략을 sequence나 따로 지정할 수 있는 방법으로 바꾸면 commit 시점에 쿼리가 날아간다.

### SEQUENCE

![](../../.gitbook/assets/kimyounghan-orm-jpa/04/screenshot%202021-03-16%20오후%203.25.31.png)

![](../../.gitbook/assets/kimyounghan-orm-jpa/04/screenshot%202021-03-16%20오후%204.41.11.png)

- 데이터베이스에 시퀀스 오브젝트를 만들어서 이 값을 통해 세팅한다.
- 오라클, PostgreSQL, DB2, H2에서 사용한다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/04/screenshot%202021-03-16%20오후%205.26.44.png)

`persist()` 할 때는 무조건 PK가 존재해야 한다. JPA는 id의 전략을 보고 전략이 sequence면 값을 DB에서 먼저 가져온다. 그래서 `persist()` 시점에 id를 조회할 수 있는 것이다. 이와 동시에 다음 시퀀스 값으로 넘어간다. 그 상태로 영속성 컨텍스트에 남아있다가 실제 커밋하는 시점에 insert 쿼리가 호출된다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/04/screenshot%202021-03-16%20오후%203.25.46.png)

계속 시퀀스를 받으러 DB와 통신을 해야하는 것이 부담스러울 수 있다. 이때 allocationSize를 사용한다.
 
여러 데이터를 이어서 persist() 하면서 next value를 call 하는 대신, 기본값 50개를 메모리 상에 일단 준비해놓고 거기서 사용한다. 다 쓰고 다시 call 하면 그 다음 50개를 가져온다. 여러 웹 서버가 있어도 동시성 문제 없이 다양한 이슈가 해결된다. 

![](../../.gitbook/assets/kimyounghan-orm-jpa/04/screenshot%202021-03-16%20오후%205.32.38.png)

실행 전에는 1이 되기 전에 -49로 세팅되어있다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/04/screenshot%202021-03-16%20오후%205.32.58.png)

데이터가 추가되면 1이 된다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/04/screenshot%202021-03-16%20오후%205.36.41.png)

처음에 dummy로 호출해서 1이 된 뒤에 next call이 51이 되도록 해서 50개를 확보한 다음 그 안에서 더 이상 call을 하지 않고도 id를 추가하고 있는 걸 볼 수 있다. 이후 51을 만나는 순간 call을 한 번 더 하게 될 것이다.

### Table

![](../../.gitbook/assets/kimyounghan-orm-jpa/04/screenshot%202021-03-16%20오후%204.44.31.png)

![](../../.gitbook/assets/kimyounghan-orm-jpa/04/screenshot%202021-03-16%20오후%204.46.46.png)

![](../../.gitbook/assets/kimyounghan-orm-jpa/04/screenshot%202021-03-16%20오후%204.48.48.png)

![](../../.gitbook/assets/kimyounghan-orm-jpa/04/screenshot%202021-03-16%20오후%204.48.34.png)

키 생성 전용 테이블을 만들어서 데이터베이스 시퀀스를 흉내내는 전략이다. 데이터베이스마다 어떤 건 identity, 어떤 건 sequence 이렇게 다양해서 나온 방법이다.

- 장점
    - 모든 데이터베이스에 적용할 수 있다.
- 단점
    - 테이블이 따로 생기다보니 락이 걸리는 등 성능이 좋지 않다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/04/screenshot%202021-03-16%20오후%204.50.15.png)

속성은 위와 같다. `allocationSize`는 위에서 말했던 내용과 같은 기능을 한다. 미리 값을 올려두는 식이기 때문에 서버 여러 대가 동시에 호출해도 자기가 쓸 값 범위만 값이 쭉 올라가있어 문제가 없다.

## 권장하는 식별자 전략

기본 키 제약 조건은 null 아니면서 유일하고 변하면 안된다는 것이다. 이 중에 변하면 안된다는 조건이 미래까지 지켜질 자연키(주민번호, 전화번호 등 비즈니스적으로 의미있는 키)를 찾기 힘들다.

예제에서 살펴본 것과 같은 대리키(대체키)를 사용하자. Long 타입 + 대체키 + 키 생성전략 사용을 권장한다.