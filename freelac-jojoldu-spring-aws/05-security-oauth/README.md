# 스프링 시큐리티와 OAuth 2.0으로 로그인 기능 구현하기

스프링 시큐리티는 막강한 인증과 인가 기능을 가진 프레임워크다. 사실상 스프링 진영에서 보안 표준이라고 보면 된다. 인터셉터, 필터 기반보다 스프링 시큐리티로 구현하는 걸 권장한다.

## 스프링 시큐리티와 스프링 시큐리티 OAuth2 클라이언트

소셜 로그인을 사용하지 않는다면 우리는 아래의 기능들을 직접 구현해야만 한다.

- 로그인 시 보안
- 회원가입 시 이메일, 전화번호 인증
- 비밀번호 찾기
- 비밀번호 변경
- 회원정보 변경 

OAuth로 구현하면 이런 기능을 모두 구글, 페이스북, 네이버 등에 맡기면 되므로 서비스 개발에 집중할 수 있다.

## 스프링 부트 1.5와 2.0

스프링 1.5와 달리 2.0에서는 OAuth 연동 방법이 크게 달라졌다. 하지만 `spring-security-oauth2-autoconfigure` 라이브러리 덕분에 1.5의 설정을 그대로 2.0에서 사용할 수 있다.

하지만 여기서는 스프링부트 2.0 방식인 `Spring security Oauth2 Client` 라이브러리를 사용한다. 이유는 다음과 같다.

- 신규 기능은 새 Oauth2 라이브러리에만 지원한다.
- 기존 방식은 확장이 적절하게 오픈되어 있지 않아 직접 상속하거나 오버라이딩 해야한다.

### 1.5와 2.0의 구분 방법

#### 라이브러리 사용

`spring-security-oauth2-autoconfigure`를 사용했다면 1.5 버전이다.

#### 설정값 확인

{% tabs %}
{% tab title="스프링부트 1.5" %}
```yaml
google: 
    client:
      clientId: 인증정보
      clientSecret: 인증정보
      accessTokenUri: https://...
      userAuthorizationUri: https://...
    resource:
      userInfoUri: https://...
```
{% endtab %}
{% tab title="스프링부트 2.0" %}
```yaml
spring:
    security:
      oauth2:
        client:
          clientId: 인증정보
          clinetSecret: 인증정보
```
{% endtab %}
{% tab title="CommonOAuth2Provider" %}
```java
public enum CommonOAuth2Provider {

	GOOGLE {

		@Override
		public Builder getBuilder(String registrationId) {
			ClientRegistration.Builder builder = getBuilder(registrationId,
					ClientAuthenticationMethod.BASIC, DEFAULT_REDIRECT_URL);
			builder.scope("openid", "profile", "email");
			builder.authorizationUri("https://accounts.google.com/o/oauth2/v2/auth");
			builder.tokenUri("https://www.googleapis.com/oauth2/v4/token");
			builder.jwkSetUri("https://www.googleapis.com/oauth2/v3/certs");
			builder.issuerUri("https://accounts.google.com");
			builder.userInfoUri("https://www.googleapis.com/oauth2/v3/userinfo");
			builder.userNameAttributeName(IdTokenClaimNames.SUB);
			builder.clientName("Google");
			return builder;
		}
	},

	GITHUB {

		@Override
		public Builder getBuilder(String registrationId) {
            ...
		}
	},

	FACEBOOK {

		@Override
		public Builder getBuilder(String registrationId) {
            ...
	},

	OKTA {

		@Override
		public Builder getBuilder(String registrationId) {
            ...
		}
	};
}
```
{% endtab %}
{% endtabs %}

`application.properties` 혹은 `application.yml`을 보면 기존 방식은 url 주소를 모두 명시해야 하지만 2.0에서는 client 인증 정보만 입력하면 된다. 해당 값들은 `CommonOAuth2Provider`라는 enum으로 대체되었기 때문이다.

해당 url의 enum 값은 구글, 깃헙, 페이스북, 옥타를 제공하며 네이버, 카카오 등을 하려면 직접 추가해줘야 한다.