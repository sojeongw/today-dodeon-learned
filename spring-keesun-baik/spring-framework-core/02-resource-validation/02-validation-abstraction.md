# Validation 추상화

> org.springframework.validation.Validator

애플리케이션에서 사용하는 객체 검증용 인터페이스

## 특징

- 어떤 계층과도 관계가 없다.
    - 웹 계층만을 위한 검증 뭐 이런 개념이 아니다.
    - 웹, 서비스, 데이터 등 모든 계층에서 사용할 수 있다.
- Bean Validation을 지원한다.
    - Bean Validation은 자바 표준 스펙 즉, Java EE 표준 스펙이다.


**Reference**

[Bean Validation](https://beanvalidation.org)
