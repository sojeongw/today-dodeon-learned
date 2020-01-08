## 인터페이스와 클래스 정의
![](../../.gitbook/assets/udemy/20200107112534.png)

`dependency`로 사용할 인터페이스 `FortuneService`와 그것을 구현할 클래스 `HappyFortuneService`를 정의한다.

## 생성자 생성
![](../../.gitbook/assets/udemy/20200107112543.png)

`의존성`을 주입하기 위해 클래스 `BaseballCoach`에 `FortuneService`를 받는 생성자를 만든다.

## 의존성 주입 설정
![](../../.gitbook/assets/udemy/20200107112553.png)

스프링 설정 파일에 `의존성 주입`을 설정한다. `의존성` 즉, `helper class`이자 실제 인터페이스를 구현하는 `HappyFortuneService`를 빈으로 등록한 뒤, 직접 사용할 `BaseballCoach` 클래스에 생성자 argument로 넣는다고 명시한다.

![](../../.gitbook/assets/udemy/20200107133800.png)

그럼 bean id에 적은 `myFortuneService`라는 변수로 `HappyFortuneService`를 만든다. 그리고 `myCoach`에는 생성자로 방금 만든 `myFortuneService`를 넘겨 준다.
