# 기본 용어 정리

## 자바 빈

아래의 두 가지를 가지고 만들어진 오브젝트

* 디폴트 생성자: 파라미터가 없는 디폴트 생성자
* 프로퍼티: 자바 빈이 노출하는 이름을 가진 것. setter와 getter로 수정 또는 조회한다.

## 리팩토링

기존에 동작하는 방식은 그대로 두고 내부 구조를 변경해서 재구성하는 작업 또는 기술을 의미한다.

* 코드를 이해하기가 편리해진다.
* 변화에 효율적으로 대응할 수 있다.
* 생산성이 올라간다.
* 코드의 품질이 높아진다.
* 유지보수가 용이해진다.

참고도서

[리팩토링](https://book.naver.com/bookdb/book_detail.nhn?bid=7047630)

## 디자인 패턴

소프트웨어를 만들 때 자주 발생하는 문제를 해결하기 위해 사용할 수 있는 솔루션이다. 주로 객체지향적 설계 원칙을 따른다. `클래스 상속`과 `오브젝트 합성` 두 가지 구조로 정리된다.

참고도서 

[GoF의 디자인 패턴](https://book.naver.com/bookdb/book_detail.nhn?bid=8942623) 

[Head First Design Patterns](https://book.naver.com/bookdb/book_detail.nhn?bid=1882446)

## 객체 지향 설계 원칙\(SOLID\)

객체 지향의 특징을 잘 살릴 수 있는 원칙이다. 절대적인 기준이라고 보다는 예외는 있겠지만 대부분의 상황에 잘 들어맞는 가이드라인 같은 것이다. `디자인 패턴`이 특별한 상황에서 사용하는 좀 더 구체적인 솔루션이라면 `객체 지향 설계 원칙`은 좀 더 일반적인 설계 기준이라고 할 수 있다.

* SRP\(The Single Responsibility Principle\): 단일 책임 원칙
* OCP\(Open/Closed Principle\): 개방 폐쇄 원칙
* LSP\(Liskov Substitution Principle\): 리스코프 치환 원칙
* ISP\(Interface Segregation Principle\): 인터페이스 분리 원칙
* DIP\(Dependency Inversion Principle\): 의존관계 역전 원칙

참고도서 

[UML 실전에서는 이것만 쓴다](https://book.naver.com/bookdb/book_detail.nhn?bid=6439362) 

[소프트웨어 개발의 지혜](https://book.naver.com/bookdb/book_detail.nhn?bid=144677)

