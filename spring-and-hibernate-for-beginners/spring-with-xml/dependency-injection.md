---
description: 클라이언트가 자신의 의존성을 다른 오브젝트에게 맡기기 위해 호출하는 것이다.
---

말이 어렵다. 예를 들어보자.

![](../../.gitbook/assets/20200107112413.png)
차를 공장에 주문한다고 생각해보자. 자동차는 각기 다른 엔진, 타이어, 의자를 가지며 엔지니어가 이들을 조합해서 전달한다. 공장이 대신해서 부품을 `주입` 하는 것이다.

내 오브젝트를 만들고 주입하는 일을 `오브젝트 팩토리`에 아웃소싱 맡긴다고 생각하면 된다. `의존성`이란 오브젝트 만드는 일을 잘 처리하기 위한 `helper object`다.

![](../../.gitbook/assets/20200107112445.png)

내가 손수 오브젝트를 조립하고 의존성을 해결하는 대신 스프링 팩토리가 모든 것을 처리해준다. `BaseballCoach`, `Hockey Coach`, `CricketCoach` 등의 오브젝트들을 사용할 수 있도록 미리 준비해준다.

![](../../.gitbook/assets/20200107112511.png)

의존성 주입에는 다양한 방법이 있다.

- Constructor Injection
- Setter Injection
- `autowired` Annotation

## 생성자 주입
### 인터페이스와 클래스 정의
![](../../.gitbook/assets/20200107112534.png)

`dependency`로 사용할 인터페이스 `FortuneService`와 그것을 구현할 클래스 `HappyFortuneService`를 정의한다.

### 생성자 생성
![](../../.gitbook/assets/20200107112543.png)

`의존성`을 주입하기 위해 클래스 `BaseballCoach`에 `FortuneService`를 받는 생성자를 만든다.

### 의존성 주입 설정
![](../../.gitbook/assets/20200107112553.png)

스프링 설정 파일에 `의존성 주입`을 설정한다. `의존성` 즉, `helper class`이자 실제 인터페이스를 구현하는 `HappyFortuneService`를 빈으로 등록한 뒤, 직접 사용할 `BaseballCoach` 클래스에 생성자 argument로 넣는다고 명시한다. 이것이 바로 생성자를 이용한 `의존성 주입`이다.