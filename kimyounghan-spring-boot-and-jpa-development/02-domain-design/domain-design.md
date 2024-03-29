# 도메인 분석 설계

## 요구 사항 분석

- 회원 기능
    - 회원 등록
    - 회원 조회
- 상품 기능
    - 상품 등록
    - 상품 수정
    - 상품 조회
- 주문 기능
    - 상품 주문
    - 주문 내역 조회
    - 주문 취소
- 기타 요구사항
    - 상품은 재고 관리가 필요하다.
    - 상품의 종류는 도서, 음반, 영화가 있다.
    - 상품을 카테고리로 구분할 수 있다.
    - 상품 주문시 배송 정보를 입력할 수 있다.

## 도메인 모델과 테이블 설계

![](../../.gitbook/assets/kimyounghan-spring-boot-and-jpa-development/01/screenshot%202022-04-16%20오후%202.33.39.png)

- 회원이 한 번 주문할 때 여러 상품을 주문할 수 있다.
    - 주문 상품 테이블을 중간에 만들어서 다대다를 풀어낸다.
- 상품은 도서, 음반, 영화 타입으로 나뉜다.
- 상품은 카테고리에 다대다로 매핑된다.
    - 상품이 여러 카테고리에 포함될 수 있고 카테고리도 여러 상품에 적용될 수 있다.

![](../../.gitbook/assets/kimyounghan-spring-boot-and-jpa-development/01/screenshot%202022-04-16%20오후%202.33.46.png)

- Member
    - 임베디드 타입인 Address를 가진다.
    - 주문 리스트를 일대다로 가진다.
        - 실무에선 단방향이 좋지만 예제를 위해 양방향을 넣었다.
        - 시스템은 회원을 통해서 주문을 생성하는 게 아니라 주문을 생성할 때 회원이 필요하다.
        - 따라서 실제로는 회원에서 주문을 가지고 있을 필요는 없다.
- Order
    - 한 번 주문할 때 여러 상품을 주문할 수 있다.
    - 중간에 OrderItem을 둬서 일대다, 다대일로 풀어낸다.
        - 중간에 테이블을 두면 각 주문한 아이템에 대한 가격과 갯수를 저장할 수도 있다.
- Delivery
    - 주문과 배송은 일대일 관계다.
- Category
    - 상품과 다대다 관계를 맺는다.
        - 다대다와 양방향은 실무에서 지양해야 하지만, 다양한 관계를 표현하기 위해 사용했다.
    - 부모, 자식 카테고리를 연결한다.
- Item
    - 도서, 음반, 영화에 따라 사용하는 속성이 각각 다르다.

![](../../.gitbook/assets/kimyounghan-spring-boot-and-jpa-development/01/screenshot%202022-04-16%20오후%202.33.54.png)

- Member
    - city, street, zipcode는 임베디드 타입인 Address에서 내려온 것이다.
- Delivery
    - 임베디드 타입인 Address의 필드를 칼럼으로 가진다.
- Item
    - 영화, 도서, 영화 타입을 통합해서 하나로 만드는 싱글 테이블 전략을 사용했다.
        - 단순하고 성능이 잘 나온다.
        - DTYPE 칼럼으로 타입을 구분한다.
- Orders
    - DB 예약어 때문에 관례상 Orders라고 한다.
- Category_Item
    - 객체는 리스트를 가질 수 있지만 DB는 불가능하므로 중간에 매핑 테이블을 둬서 일대다, 다대일로 풀어낸다.

## 연관 관계 매핑 분석

![](../../.gitbook/assets/kimyounghan-spring-boot-and-jpa-development/01/screenshot%202022-04-16%20오후%202.33.46.png)

### 회원과 주문

- 일대다, 다대일의 양방향 관계
- 연관 관계의 주인은 외래키가 있는, 즉 N쪽인 주문으로 정하는 것이 좋다.
    - order가 member_id를 PK로 들고 있기 때문이다.
- 객체로 보면 `Order.member`가 주인이다. 따라서 `Order.member`를 `ORDERS.MEMBER_ID` 외래키와 매핑한다.
- 값을 변경할 때는 연관 관계 주인이 되는 쪽에 세팅해야 한다.
    - 반대편은 단순한 조회용으로만 쓴다.

### 주문 상품과 주문

- 다대일 양방향 관계
- 외래키 `ORDER_ID`가 주문 상품에 있으므로 연관 관계의 주인은 주문 상품이다.
- 즉, `OrderItem.order`를 `ORDER_ITEM.ORDER_ID` 외래 키와 매핑한다.

### 주문 상품과 상품

- 다대일 단방향 관계
- `OrderItem.item`을 `ORDER_ITEM.ITEM_ID`와 매핑한다.

### 주문과 배송

- 일대일 양방향 관계
    - 왜래키를 어디다 둬도 상관없다.
- `Order.delivery`를 `ORDERS.DELIVERY_ID` 외래 키와 매핑한다.

### 카테고리와 상품

- 다대다 양방향 관계
- `@ManyToMany`를 사용해 매핑한다.

## 연관 관계의 주인

- 연관 관계의 주인은 단순히 외래키를 누가 관리하느냐의 문제다.
- 비즈니스 상 우위에 있다고 주인으로 정하면 안된다.
    - 자동차와 바퀴가 있으면 일대다 관계에서 항상 다쪽에 외래키가 있으므로 바퀴가 주인이 된다.
    - 자동차로 정하면 자동차가 관리하지 않는 바퀴 테이블의 외래키 값이 업데이트 되어 관리가 어렵다.
    - 추가적인 업데이트 쿼리가 나가는 성능 상의 이슈도 있다.