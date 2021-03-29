# 새로운 구조와 할인 정책 적용

이제 정액 할인 정책을 정률 할인 정책으로 변경해보자. `FixDiscountPolicy`를 `RateDiscountPolicy`로 변경하면 된다.

![](../../.gitbook/assets/kimyounghan-spring-core-principle/03/screenshot%202021-04-10%20오전%2010.25.51.png)

`AppConfig`의 등장으로 애플리케이션이 `사용 영역`과 `객체를 생성 및 구성하는 영역`으로 분리되었다.

![](../../.gitbook/assets/kimyounghan-spring-core-principle/03/screenshot%202021-04-10%20오전%2010.25.57.png)

기존에는 클라이언트 코드가 영향을 받았지만 이제는 `AppConfig`만 변경하면 된다. 구성 영역만 고치고 사용 영역은 전혀 건들 필요가 없다.

확장엔 열려있고 변경엔 닫혀있는 `OCP`를 지키게 되었다.