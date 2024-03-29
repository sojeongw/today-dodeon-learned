# 정리

## V1: 프론트 컨트롤러 도입

- 기존 구조를 최대한 유지하면서 프론트 컨트롤러를 도입한다.
- 애플리케이션 구조를 바꿀 때는 세세한 것과 큰 틀을 바꾸는 걸 섞지 말자.
    - 일단 큰 구조를 바꾼 다음 세세한 걸 하는 식으로 분리해야 한다.

## V2: view 분리

- view 처리 로직이 계속 반복되어 분리한다.

## V3: model 추가

- 서블릿 종속성을 제거한다.
    - httpServletRequest/Response를 빼고 model 객체를 별도로 넣어 처리한다.
- 중복되는 뷰 이름을 제거한다.
    - view를 물리 이름에서 논리 이름으로 반환하면 viewResolver가 실제 view까지 만들도록 한다.

## V4: 단순하고 실용적인 컨트롤러

- 인터페이스를 제공한다.
    - V3와 거의 비슷하나 구현하는 입장에서는 modelView를 직접 생성하고 반환하지 않아 편리하게 사용할 수 있었다.

## V5: 유연한 컨트롤러

- 어댑터를 도입한다.
- 어댑터 덕분에 유연하고 확장성 있게 설계할 수 있다.
    - 핸들러 어댑터만 새로 만들면 애너테이션을 사용하는 컨트롤러로 개선할 수 있다.

## 스프링 MVC

스프링 MVC 또한 이렇게 어댑터를 사용하는 구조로 되어있다. 

예를 들어 @RequestMapping 기능은 RequestMappingHandlerAdapter를 사용한다. @RequestMapping 애너테이션을 처리해주는 어댑터인 것이다.