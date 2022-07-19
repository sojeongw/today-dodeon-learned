# Table of contents

* [개발왕, 도던](README.md)

## 스프링 입문 <a id="kimyounghan-spring-introduction"></a>

* [프로젝트 환경설정](kimyounghan-spring-introduction/01-project-settings/README.md)
* [스프링 웹 개발 기초](kimyounghan-spring-introduction/02-spring-basic/README.md)
* [회원 관리 예제 - 백엔드](kimyounghan-spring-introduction/03-example-backend/README.md)
* [스프링 빈과 의존 관계](kimyounghan-spring-introduction/04-spring-bean-dependencies/README.md)
* [회원 관리 예제 - MVC](kimyounghan-spring-introduction/05-example-web-mvc/README.md)
* [스프링 DB 접근 기술](kimyounghan-spring-introduction/06-database/README.md)
    * [JDBC](kimyounghan-spring-introduction/06-database/jdbc.md)
    * [JPA](kimyounghan-spring-introduction/06-database/jpa.md)
* [AOP](kimyounghan-spring-introduction/07-aop/README.md)

## 스프링 핵심 원리 <a id="kimyounghan-spring-core-principle"></a>

* [객체 지향 설계와 스프링](kimyounghan-spring-core-principle/01-oop-spring/README.md)
    * [스프링의 탄생](kimyounghan-spring-core-principle/01-oop-spring/spring.md)
    * [객체 지향 프로그래밍](kimyounghan-spring-core-principle/01-oop-spring/oop.md)
    * [좋은 객체 지향 설계의 원칙](kimyounghan-spring-core-principle/01-oop-spring/oop-principle.md)
    * [객체 지향 설계와 스프링](kimyounghan-spring-core-principle/01-oop-spring/oop-spring.md)
* [스프링 핵심 원리 이해](kimyounghan-spring-core-principle/02-understanding-core-principle/README.md)
    * [회원 도메인 개발](kimyounghan-spring-core-principle/02-understanding-core-principle/example-member.md)
    * [주문 도메인 개발](kimyounghan-spring-core-principle/02-understanding-core-principle/example-order.md)
* [객체 지향 원리 적용](kimyounghan-spring-core-principle/03-oop/README.md)
    * [관심사의 분리](kimyounghan-spring-core-principle/03-oop/concerns.md)
    * [새로운 구조와 정책 적용](kimyounghan-spring-core-principle/03-oop/new-policy.md)
    * [정리](kimyounghan-spring-core-principle/03-oop/summary.md)
    * [IoC, DI, 컨테이너](kimyounghan-spring-core-principle/03-oop/ioc-di-container.md)
    * [스프링으로 전환하기](kimyounghan-spring-core-principle/03-oop/convert-to-spring.md)
* [스프링 컨테이너와 스프링 빈](kimyounghan-spring-core-principle/04-spring-container-and-bean/README.md)
    * [스프링 빈 기본 조회](kimyounghan-spring-core-principle/04-spring-container-and-bean/get-bean.md)
    * [동일 타입이 둘 이상일 때 조회](kimyounghan-spring-core-principle/04-spring-container-and-bean/get-bean-multiple-type.md)
    * [상속일 때 조회](kimyounghan-spring-core-principle/04-spring-container-and-bean/get-bean-inheritance.md)
    * [BeanFactory와 ApplicationContext](kimyounghan-spring-core-principle/04-spring-container-and-bean/beanfactory-applicationcontext.md)
    * [다양한 설정 형식](kimyounghan-spring-core-principle/04-spring-container-and-bean/configuration.md)
    * [스프링 빈 설정 메타 데이터](kimyounghan-spring-core-principle/04-spring-container-and-bean/bean-metadata.md)
* [싱글턴 컨테이너](kimyounghan-spring-core-principle/05-singleton/README.md)
    * [@Configuration과 싱글턴](kimyounghan-spring-core-principle/05-singleton/configuration.md)
* [컴포넌트 스캔](kimyounghan-spring-core-principle/06-component-scan/README.md)
    * [탐색 위치와 기본 탐색 대상](kimyounghan-spring-core-principle/06-component-scan/location.md)
    * [필터와 중복 등록](kimyounghan-spring-core-principle/06-component-scan/filter-and-duplication.md)
* [의존 관계 자동 주입](kimyounghan-spring-core-principle/07-autowire/README.md)
    * [롬복과 최신 트렌드](kimyounghan-spring-core-principle/07-autowire/lombok.md)
    * [조회 빈이 2개 이상일 때](kimyounghan-spring-core-principle/07-autowire/multiple-beans.md)
    * [애너테이션 직접 만들기](kimyounghan-spring-core-principle/07-autowire/custom-annotation.md)
    * [조회한 빈이 모두 필요할 때](kimyounghan-spring-core-principle/07-autowire/retrieve-all-beans.md)
    * [올바른 실무 운영 기준](kimyounghan-spring-core-principle/07-autowire/standard.md)
* [빈 생명 주기 콜백](kimyounghan-spring-core-principle/08-lifecycle/README.md)
    * [인터페이스 방식](kimyounghan-spring-core-principle/08-lifecycle/interface.md)
    * [메서드 지정 방식](kimyounghan-spring-core-principle/08-lifecycle/method.md)
    * [애너테이션 방식](kimyounghan-spring-core-principle/08-lifecycle/annotation.md)
* [빈 스코프](kimyounghan-spring-core-principle/09-bean-scope/README.md)
    * [프로토타입 스코프](kimyounghan-spring-core-principle/09-bean-scope/prototype.md)
    * [Provider](kimyounghan-spring-core-principle/09-bean-scope/provider.md)
    * [웹 스코프](kimyounghan-spring-core-principle/09-bean-scope/web-scope.md)

## 모든 개발자를 위한 HTTP 웹 기본 지식 <a id="kimyounghan-http-basic"></a>

* [인터넷 네트워크](kimyounghan-http-basic/01-internet-network/README.md)
    * [IP](kimyounghan-http-basic/01-internet-network/ip.md)
    * [TCP, UDP](kimyounghan-http-basic/01-internet-network/tcp-udp.md)
    * [PORT](kimyounghan-http-basic/01-internet-network/port.md)
    * [DNS](kimyounghan-http-basic/01-internet-network/dns.md)
* [URI와 웹 브라우저 요청 흐름](kimyounghan-http-basic/02-uri-and-request/README.md)
* [HTTP 기본](kimyounghan-http-basic/03-http-basic/README.md)
    * [클라이언트-서버 구조](kimyounghan-http-basic/03-http-basic/client-server.md)
    * [stateful, stateless](kimyounghan-http-basic/03-http-basic/state.md)
    * [비 연결성](kimyounghan-http-basic/03-http-basic/connectionless.md)
    * [HTTP 메시지](kimyounghan-http-basic/03-http-basic/message.md)
* [HTTP 메서드](kimyounghan-http-basic/04-http-method/README.md)
* [HTTP 메서드 활용](kimyounghan-http-basic/05-http-method-usage/README.md)
* [HTTP 상태 코드](kimyounghan-http-basic/06-http-status-code/README.md)
* [HTTP 헤더 - 일반](kimyounghan-http-basic/07-http-header/README.md)
    * [표현](kimyounghan-http-basic/07-http-header/representation.md)
    * [콘텐츠 협상](kimyounghan-http-basic/07-http-header/content-negotiation.md)
    * [전송 방식](kimyounghan-http-basic/07-http-header/transfer.md)
    * [정보](kimyounghan-http-basic/07-http-header/information.md)
    * [Authorization](kimyounghan-http-basic/07-http-header/authorization.md)
    * [쿠키](kimyounghan-http-basic/07-http-header/cookie.md)
* [HTTP 헤더 - 캐시](kimyounghan-http-basic/08-http-header-cache/README.md)
    * [검증 헤더와 조건부 요청](kimyounghan-http-basic/08-http-header-cache/validation-and-conditional-request.md)
    * [조건부 요청 헤더](kimyounghan-http-basic/08-http-header-cache/conditional-request-header.md)
    * [프록시 캐시](kimyounghan-http-basic/08-http-header-cache/proxy-cache.md)
    * [캐시 무효화](kimyounghan-http-basic/08-http-header-cache/cache-revalidation.md)

## 스프링 MVC <a id="kimyounghan-spring-mvc"></a>

* [웹 애플리케이션 이해](kimyounghan-spring-mvc/01-web-application/README.md)
    * [서버](kimyounghan-spring-mvc/01-web-application/server.md)
    * [서블릿](kimyounghan-spring-mvc/01-web-application/servlet.md)
    * [멀티 스레드](kimyounghan-spring-mvc/01-web-application/multi-thread.md)
    * [HTML, HTTP API, CSR, SSR](kimyounghan-spring-mvc/01-web-application/html-httpapi-csr-ssr.md)
    * [자바 백엔드 웹 기술 역사](kimyounghan-spring-mvc/01-web-application/history.md)
* [서블릿](kimyounghan-spring-mvc/02-servlet/README.md)
    * [HttpServletRequest](kimyounghan-spring-mvc/02-servlet/http-servlet-request.md)
    * [HTTP 요청 데이터](kimyounghan-spring-mvc/02-servlet/http-request-data.md)
    * [HttpServletResponse](kimyounghan-spring-mvc/02-servlet/http-servlet-response.md)
    * [HTTP 응답 데이터](kimyounghan-spring-mvc/02-servlet/http-response-data.md)
* [서블릿, JSP, MVC 패턴](kimyounghan-spring-mvc/03-servlet-jsp-mvc/README.md)
    * [서블릿으로 만들기](kimyounghan-spring-mvc/03-servlet-jsp-mvc/servlet.md)
    * [JSP로 만들기](kimyounghan-spring-mvc/03-servlet-jsp-mvc/jsp.md)
    * [MVC 패턴](kimyounghan-spring-mvc/03-servlet-jsp-mvc/mvc-pattern.md)
* [MVC 프레임워크 만들기](kimyounghan-spring-mvc/04-create-mvc-framework/README.md)
    * [프론트 컨트롤러 패턴](kimyounghan-spring-mvc/04-create-mvc-framework/front-controller-pattern.md)
    * [View 분리](kimyounghan-spring-mvc/04-create-mvc-framework/view.md)
    * [Model 추가](kimyounghan-spring-mvc/04-create-mvc-framework/model.md)
    * [단순하고 실용적인 컨트롤러](kimyounghan-spring-mvc/04-create-mvc-framework/simple-controller.md)
    * [유연한 컨트롤러](kimyounghan-spring-mvc/04-create-mvc-framework/flexible-controller.md)
    * [정리](kimyounghan-spring-mvc/04-create-mvc-framework/summary.md)
* [스프링 MVC의 구조 이해](kimyounghan-spring-mvc/05-spring-mvc-architecture/README.md)
    * [스프링 MVC 전체 구조](kimyounghan-spring-mvc/05-spring-mvc-architecture/architecture.md)
    * [핸들러 매핑과 핸들러 어댑터](kimyounghan-spring-mvc/05-spring-mvc-architecture/handler.md)
    * [뷰 리졸버](kimyounghan-spring-mvc/05-spring-mvc-architecture/view-resolver.md)
    * [스프링 MVC 시작하기](kimyounghan-spring-mvc/05-spring-mvc-architecture/introduction.md)
    * [스프링 MVC 컨트롤러 통합](kimyounghan-spring-mvc/05-spring-mvc-architecture/controller-combination.md)
    * [스프링 MVC 실용적인 방식](kimyounghan-spring-mvc/05-spring-mvc-architecture/practice.md)
* [스프링 MVC 기본 기능](kimyounghan-spring-mvc/06-basic/README.md)
    * [프로젝트 생성](kimyounghan-spring-mvc/06-basic/initializing-project.md)
    * [로깅](kimyounghan-spring-mvc/06-basic/logging.md)
    * [요청 매핑](kimyounghan-spring-mvc/06-basic/request-mapping.md)
    * [HTTP 요청의 기본 및 헤더 조회](kimyounghan-spring-mvc/06-basic/http-request-basic.md)
    * [HTTP 요청 파라미터](kimyounghan-spring-mvc/06-basic/http-request-parameter.md)
    * [HTTP 요청 메시지](kimyounghan-spring-mvc/06-basic/http-request-message.md)
    * [HTTP 응답](kimyounghan-spring-mvc/06-basic/http-response.md)
    * [HTTP 메시지 컨버터](kimyounghan-spring-mvc/06-basic/http-message-converter.md)
    * [요청 매핑 핸들러 어댑터](kimyounghan-spring-mvc/06-basic/request-mapping-handler-adapter.md)
* [스프링 MVC 웹 페이지 만들기](kimyounghan-spring-mvc/07-example/README.md)
* [메시지, 국제화](kimyounghan-spring-mvc/08-message-and-internationalization/README.md)
    * [스프링 메시지 소스](kimyounghan-spring-mvc/08-message-and-internationalization/spring-message-source.md)
* [Validation](kimyounghan-spring-mvc/09-validation/README.md)
    * [BindingResult](kimyounghan-spring-mvc/09-validation/binding-result.md)
    * [FieldError, ObjectError](kimyounghan-spring-mvc/09-validation/field-error-obejct-error.md)
    * [오류 코드와 메시지 처리](kimyounghan-spring-mvc/09-validation/error-code-and-message.md)
    * [Validator 분리](kimyounghan-spring-mvc/09-validation/validator.md)
* [Bean Validation](kimyounghan-spring-mvc/10-bean-validation/README.md)
    * [Form 전송 객체 분리](kimyounghan-spring-mvc/10-bean-validation/html-form.md)
    * [HTTP 메시지 컨버터](kimyounghan-spring-mvc/10-bean-validation/http-message-converter.md)
* [로그인](kimyounghan-spring-mvc/11-login/README.md)
    * [쿠키](kimyounghan-spring-mvc/11-login/cookie.md)
    * [세션](kimyounghan-spring-mvc/11-login/session.md)
    * [서블릿 HTTP 세션](kimyounghan-spring-mvc/11-login/servlet-http-session.md)
    * [서블릿 필터](kimyounghan-spring-mvc/11-login/servlet-filter.md)
    * [스프링 인터셉터](kimyounghan-spring-mvc/11-login/spring-interceptor.md)
    * [ArgumentResolver 활용](kimyounghan-spring-mvc/11-login/argument-resolver.md)
* [예외 처리와 오류 페이지](kimyounghan-spring-mvc/12-exception/README.md)
    * [오류 화면 제공](kimyounghan-spring-mvc/12-exception/error-page.md)
    * [필터](kimyounghan-spring-mvc/12-exception/filter.md)
    * [인터셉터](kimyounghan-spring-mvc/12-exception/interceptor.md)
    * [스프링 부트 오류 페이지](kimyounghan-spring-mvc/12-exception/spring-boot-error-page.md)
* [API 예외 처리](kimyounghan-spring-mvc/13-api-exception/api-exception-handling.md)
    * [스프링 부트 기본 오류 처리](kimyounghan-spring-mvc/13-api-exception/spring-boot-basic-exception-handling.md)
    * [HandlerExceptionResolver](kimyounghan-spring-mvc/13-api-exception/handler-exception-resolver.md)
    * [ExceptionResolver](kimyounghan-spring-mvc/13-api-exception/exception-resolver.md)
    * [ControllerAdvice](kimyounghan-spring-mvc/13-api-exception/controller-advice.md)
* [스프링 타입 컨버터](kimyounghan-spring-mvc/14-spring-type-converter/README.md)
    * [Converter](kimyounghan-spring-mvc/14-spring-type-converter/converter.md)
    * [ConversionService](kimyounghan-spring-mvc/14-spring-type-converter/conversion-service.md)
    * [뷰 템플릿에 적용하기](kimyounghan-spring-mvc/14-spring-type-converter/view-template.md)
    * [Formatter](kimyounghan-spring-mvc/14-spring-type-converter/formatter.md)
* [파일 업로드](kimyounghan-spring-mvc/15-file-upload/README.md)
    * [서블릿과 파일 업로드](kimyounghan-spring-mvc/15-file-upload/servlet-file-upload.md)
    * [스프링과 파일 업로드](kimyounghan-spring-mvc/15-file-upload/spring-file-upload.md)
    * [파일 업로드 및 다운로드 예제](kimyounghan-spring-mvc/15-file-upload/file-upload-download-example.md)

## 자바 ORM 표준 JPA 프로그래밍 <a id="kimyounghan-orm-jpa"></a>

* [JPA 소개](kimyounghan-orm-jpa/01-jpa-introduction/README.md)
* [JPA 시작하기](kimyounghan-orm-jpa/02-starting-jpa/README.md)
* [영속성 관리](kimyounghan-orm-jpa/03-persistence/README.md)
    * [영속성 컨텍스트](kimyounghan-orm-jpa/03-persistence/persistence-context.md)
    * [플러시](kimyounghan-orm-jpa/03-persistence/flush.md)
    * [준영속 상태](kimyounghan-orm-jpa/03-persistence/detach.md)
* [Entity 매핑](kimyounghan-orm-jpa/04-entity-mapping/README.md)
    * [객체와 테이블 매핑](kimyounghan-orm-jpa/04-entity-mapping/object-and-table-mapping.md)
    * [데이터베이스 스키마 자동 생성](kimyounghan-orm-jpa/04-entity-mapping/auto-generating-schema.md)
    * [필드와 칼럼 매핑](kimyounghan-orm-jpa/04-entity-mapping/column-mapping.md)
    * [기본 키 매핑](kimyounghan-orm-jpa/04-entity-mapping/primary-key-mapping.md)
    * [실전 예제](kimyounghan-orm-jpa/04-entity-mapping/example.md)
* [연관 관계 매핑](kimyounghan-orm-jpa/05-relationship-mapping/README.md)
    * [단방향 연관 관계](kimyounghan-orm-jpa/05-relationship-mapping/unidirectional.md)
    * [양방향 연관 관계](kimyounghan-orm-jpa/05-relationship-mapping/bidirectional.md)
    * [실전 예제](kimyounghan-orm-jpa/05-relationship-mapping/example.md)
* [다양한 연관 관계 매핑](kimyounghan-orm-jpa/06-various-relationship-mapping/README.md)
    * [다대일](kimyounghan-orm-jpa/06-various-relationship-mapping/n-1.md)
    * [일대다](kimyounghan-orm-jpa/06-various-relationship-mapping/1-n.md)
    * [일대일](kimyounghan-orm-jpa/06-various-relationship-mapping/1-1.md)
    * [다대다](kimyounghan-orm-jpa/06-various-relationship-mapping/n-n.md)
    * [실전 예제](kimyounghan-orm-jpa/06-various-relationship-mapping/example.md)
* [고급 매핑](kimyounghan-orm-jpa/07-advanced-mapping/README.md)
    * [상속 관계 매핑](kimyounghan-orm-jpa/07-advanced-mapping/inheritance.md)
    * [매핑 정보 상속](kimyounghan-orm-jpa/07-advanced-mapping/mapping-info-inheritance.md)
    * [실전 예제](kimyounghan-orm-jpa/07-advanced-mapping/example.md)
* [프록시와 연관관계 관리](kimyounghan-orm-jpa/08-proxy/README.md)
    * [프록시](kimyounghan-orm-jpa/08-proxy/01-proxy.md)
    * [즉시 로딩과 지연 로딩](kimyounghan-orm-jpa/08-proxy/02-eager-and-lazy.md)
    * [영속성 전이와 고아 객체](kimyounghan-orm-jpa/08-proxy/03-cascade.md)
    * [실전 예제](kimyounghan-orm-jpa/08-proxy/04-example.md)
* [값 타입](kimyounghan-orm-jpa/09-value-type/README.md)
    * [기본값 타입](kimyounghan-orm-jpa/09-value-type/primitive-type.md)
    * [임베디드 타입](kimyounghan-orm-jpa/09-value-type/embedded-type.md)
    * [값 타입과 불변 객체](kimyounghan-orm-jpa/09-value-type/immutable-value.md)
    * [값 타입의 비교](kimyounghan-orm-jpa/09-value-type/comparison.md)
    * [값 타입 컬렉션](kimyounghan-orm-jpa/09-value-type/collection.md)
    * [실전 예제](kimyounghan-orm-jpa/09-value-type/example.md)
* [객체 지향 쿼리 언어 - 기본](kimyounghan-orm-jpa/10-object-oriented-query/README.md)
    * [기본 문법과 쿼리 API](kimyounghan-orm-jpa/10-object-oriented-query/basic.md)
    * [프로젝션](kimyounghan-orm-jpa/10-object-oriented-query/select.md)
    * [페이징](kimyounghan-orm-jpa/10-object-oriented-query/paging.md)
    * [조인](kimyounghan-orm-jpa/10-object-oriented-query/join.md)
    * [서브 쿼리](kimyounghan-orm-jpa/10-object-oriented-query/sub-query.md)
    * [JPQL 타입 표현과 기타 식](kimyounghan-orm-jpa/10-object-oriented-query/type.md)
    * [조건식](kimyounghan-orm-jpa/10-object-oriented-query/conditional-expression.md)
    * [JPQL 함수](kimyounghan-orm-jpa/10-object-oriented-query/function.md)
* [객체 지향 쿼리 언어 - 중급](kimyounghan-orm-jpa/11-object-oriented-query-intermidiate/README.md)
    * [경로 표현식](kimyounghan-orm-jpa/11-object-oriented-query-intermidiate/graph.md)
    * [fetch join](kimyounghan-orm-jpa/11-object-oriented-query-intermidiate/fetch-join.md)
    * [다형성 쿼리](kimyounghan-orm-jpa/11-object-oriented-query-intermidiate/polymorphism.md)
    * [Entity 직접 사용](kimyounghan-orm-jpa/11-object-oriented-query-intermidiate/using-entity.md)
    * [Named 쿼리](kimyounghan-orm-jpa/11-object-oriented-query-intermidiate/named-query.md)
    * [벌크 연산](kimyounghan-orm-jpa/11-object-oriented-query-intermidiate/bulk-calculation.md)

## 스프링 부트와 JPA 활용 - 웹 애플리케이션 개발 <a id="kimyounghan-spring-boot-and-jpa-development"></a>

* [프로젝트 환경설정](kimyounghan-spring-boot-and-jpa-development/01-project-settings/README.md)
* [도메인 분석 설계](kimyounghan-spring-boot-and-jpa-development/02-domain-design/README.md)
    * [도메인 분석 설계](kimyounghan-spring-boot-and-jpa-development/02-domain-design/domain-design.md)
    * [Entity 클래스 개발](kimyounghan-spring-boot-and-jpa-development/02-domain-design/entity.md)
    * [Entity 설계 시 주의점](kimyounghan-spring-boot-and-jpa-development/02-domain-design/cautions.md)
* [애플리케이션 아키텍처](kimyounghan-spring-boot-and-jpa-development/03-architecture/README.md)
* [회원 도메인 개발](kimyounghan-spring-boot-and-jpa-development/04-member/README.md)
* [상품 도메인 개발](kimyounghan-spring-boot-and-jpa-development/05-product/README.md)
* [주문 도메인 개발](kimyounghan-spring-boot-and-jpa-development/06-order/README.md)
    * [Entity, 리포지토리, 서비스 개발](kimyounghan-spring-boot-and-jpa-development/06-order/development.md)
    * [주문 기능 테스트](kimyounghan-spring-boot-and-jpa-development/06-order/test.md)
    * [주문 검색 기능 개발](kimyounghan-spring-boot-and-jpa-development/06-order/search.md)
* [웹 계층 개발](kimyounghan-spring-boot-and-jpa-development/07-web-layer/README.md)
    * [변경 감지와 병합](kimyounghan-spring-boot-and-jpa-development/07-web-layer/dirty-checking-and-merge.md)

## 스프링 부트와 JPA 활용 - API 개발과 성능 최적화 <a id="kimyounghan-spring-boot-and-jpa-optimization"></a>

* [API 개발 기본](kimyounghan-spring-boot-and-jpa-optimization/01-api-development/README.md)
    * [회원 등록 API](kimyounghan-spring-boot-and-jpa-optimization/01-api-development/register-member.md)
    * [회원 수정 API](kimyounghan-spring-boot-and-jpa-optimization/01-api-development/modify-member.md)
    * [회원 조회 API](kimyounghan-spring-boot-and-jpa-optimization/01-api-development/search-member.md)
* [지연 로딩과 조회 성능 최적화](kimyounghan-spring-boot-and-jpa-optimization/02-lazy-loading-and-optimization/README.md)
    * [Entity 직접 노출](kimyounghan-spring-boot-and-jpa-optimization/02-lazy-loading-and-optimization/entity.md)
    * [Entity를 DTO로 변환](kimyounghan-spring-boot-and-jpa-optimization/02-lazy-loading-and-optimization/entity-to-dto.md)
    * [JPA에서 DTO 직접 조회](kimyounghan-spring-boot-and-jpa-optimization/02-lazy-loading-and-optimization/dto.md)
* [컬렉션 조회 최적화](kimyounghan-spring-boot-and-jpa-optimization/03-collection-optimization/README.md)
    * [Entity 직접 노출](kimyounghan-spring-boot-and-jpa-optimization/03-collection-optimization/entity.md)
    * [Entity를 DTO로 변환: 페치 조인](kimyounghan-spring-boot-and-jpa-optimization/03-collection-optimization/entity-to-dto.md)
    * [Entity를 DTO로 변환: 페이징과 한계 돌파](kimyounghan-spring-boot-and-jpa-optimization/03-collection-optimization/paging.md)
    * [DTO 직접 조회](kimyounghan-spring-boot-and-jpa-optimization/03-collection-optimization/dto.md)
    * [DTO 직접 조회: 컬렉션 조회 최적화](kimyounghan-spring-boot-and-jpa-optimization/03-collection-optimization/collection-optimization.md)
    * [DTO 직접 조회: 플랫 데이터 최적화](kimyounghan-spring-boot-and-jpa-optimization/03-collection-optimization/flat.md)
    * [정리](kimyounghan-spring-boot-and-jpa-optimization/03-collection-optimization/summary.md)
* [OSIV와 성능 최적화](kimyounghan-spring-boot-and-jpa-optimization/04-osiv/README.md)

## 스프링 데이터 JPA <a id="kimyounghan-spring-data-jpa"></a>

* [예제 도메인 모델](kimyounghan-spring-data-jpa/01-domain-model/README.md)
* [공통 인터페이스 기능](kimyounghan-spring-data-jpa/02-common-interface/README.md)
    * [순수 JPA 기반 리포지토리](kimyounghan-spring-data-jpa/02-common-interface/01-pure-jpa-repository.md)
    * [공통 인터페이스 설정](kimyounghan-spring-data-jpa/02-common-interface/02-common-interface.md)
* [쿼리 메서드 기능](kimyounghan-spring-data-jpa/03-query-method/README.md)
    * [JPA Named Query](kimyounghan-spring-data-jpa/03-query-method/01-named-query.md)
    * [@Query](kimyounghan-spring-data-jpa/03-query-method/02-query.md)
    * [파라미터 바인딩](kimyounghan-spring-data-jpa/03-query-method/03-parameter-binding.md)
    * [반환 타입](kimyounghan-spring-data-jpa/03-query-method/04-return-type.md)
    * [페이징과 정렬](kimyounghan-spring-data-jpa/03-query-method/05-paging-and-sorting.md)
    * [벌크성 수정 쿼리](kimyounghan-spring-data-jpa/03-query-method/06-bulk-update-query.md)
    * [@EntityGraph](kimyounghan-spring-data-jpa/03-query-method/07-entity-graph.md)
    * [JPA Hint & Lock](kimyounghan-spring-data-jpa/03-query-method/08-jpa-hint-and-lock.md)
* [확장 기능](kimyounghan-spring-data-jpa/04-extension/README.md)
    * [사용자 정의 리포지토리](kimyounghan-spring-data-jpa/04-extension/01-custom-repository.md)
    * [Auditing](kimyounghan-spring-data-jpa/04-extension/02-auditing.md)
    * [Web 확장](kimyounghan-spring-data-jpa/04-extension/03-web-extension.md)
* [스프링 데이터 JPA 분석](kimyounghan-spring-data-jpa/05-spring-data-jpa/README.md)
* [나머지 기능](kimyounghan-spring-data-jpa/06-etc/README.md)
    * [Specifications](kimyounghan-spring-data-jpa/06-etc/01-specifications.md)
    * [Query By Example](kimyounghan-spring-data-jpa/06-etc/02-query-by-example.md)
    * [Projections](kimyounghan-spring-data-jpa/06-etc/03-projections.md)
    * [Native Query](kimyounghan-spring-data-jpa/06-etc/04-native-query.md)

## Querydsl <a id="querydsl"></a>

* [프로젝트 환경 설정](kimyounghan-querydsl/01-project-settings/README.md)
* [예제 도메인 모델](kimyounghan-querydsl/02-example-domain-model/README.md)
* [기본 문법](kimyounghan-querydsl/03-basic/README.md)
    * [JPQL vs Querydsl](kimyounghan-querydsl/03-basic/jpql-vs-querydsl.md)
    * [Q-Type 활용](kimyounghan-querydsl/03-basic/q-type.md)
    * [검색 조건](kimyounghan-querydsl/03-basic/search-condition.md)
    * [결과 조회](kimyounghan-querydsl/03-basic/find-result.md)
    * [정렬](kimyounghan-querydsl/03-basic/sorting.md)
    * [페이징](kimyounghan-querydsl/03-basic/paging.md)
    * [집합 함수](kimyounghan-querydsl/03-basic/aggregate.md)
    * [조인](kimyounghan-querydsl/03-basic/join.md)
    * [서브 쿼리](kimyounghan-querydsl/03-basic/subquery.md)
    * [Case 문](kimyounghan-querydsl/03-basic/case-statement.md)
    * [상수, 문자 더하기](kimyounghan-querydsl/03-basic/constant-and-string.md)
* [중급 문법](kimyounghan-querydsl/04-advanced/README.md)
    * [프로젝션과 결과 반환](kimyounghan-querydsl/04-advanced/projection-and-return-result.md)
    * [동적 쿼리](kimyounghan-querydsl/04-advanced/dynamic-query.md)
    * [수정, 삭제 벌크 연산](kimyounghan-querydsl/04-advanced/batch.md)
    * [SQL Function 호출](kimyounghan-querydsl/04-advanced/sql-function.md)
* [순수 JPA와 Querydsl](kimyounghan-querydsl/04-jpa-querydsl/README.md)
    * [순수 JPA 리포지토리와 Querydsl](kimyounghan-querydsl/04-jpa-querydsl/jpa-querydsl.md)
    * [동적 쿼리와 성능 최적화 조회](kimyounghan-querydsl/04-jpa-querydsl/dynamic-query.md)
    * [조회 API 컨트롤러 개발](kimyounghan-querydsl/04-jpa-querydsl/query-api-controller.md)
* [스프링 데이터 JPA와 Querydsl](kimyounghan-querydsl/05-spring-data-jpa-querydsl/README.md)
    * [스프링 데이터 페이징 활용](kimyounghan-querydsl/05-spring-data-jpa-querydsl/using-spring-data-paging.md)
    * [스프링 데이터 JPA가 제공하는 Querydsl 기능](kimyounghan-querydsl/05-spring-data-jpa-querydsl/querydsl.md)

## 백엔드 시스템 실무 <a id="backend-system-practice"></a>

* [CPU bound 애플리케이션](backend-system-practice/01-cpu-bound-application/README.md)
    * [CPU를 극단적으로 사용하는 애플리케이션](backend-system-practice/01-cpu-bound-application/cpu-bound-application.md)
    * [스트레스 테스트 툴로 성능 측정](backend-system-practice/01-cpu-bound-application/performance-benchmark.md)
    * [Dockerized 애플리케이션 GCP 배포](backend-system-practice/01-cpu-bound-application/dockerized-application-gcp-deployment.md)
    * [Jenkins를 이용한 배포](backend-system-practice/01-cpu-bound-application/jenkins-deployment.md)
* [CPU bound 애플리케이션 무중단 배포](backend-system-practice/02-zero-downtime-deployment/README.md)
    * [nginx를 통한 로드밸런싱 구성](backend-system-practice/02-zero-downtime-deployment/nginx-deployment.md)
    * [서버를 늘려서 성능 측정](backend-system-practice/02-zero-downtime-deployment/scale-out-performance.md)
* [배포 자동화와 협업을 위한 Git](backend-system-practice/03-deployment-automation/README.md)
    * [GitHub Webhook과 jenkins로 배포 자동화](backend-system-practice/03-deployment-automation/github-webhook.md)

## 백기선 스프링 강의 <a id="spring-keesun-baik"></a>

* [예제로 배우는 스프링 입문(개정판)](spring-keesun-baik/spring-framework-introduction/README.md)
    * [PetClinic 예제](spring-keesun-baik/spring-framework-introduction/01-pet-clinic.md)
    * [스프링 IoC](spring-keesun-baik/spring-framework-introduction/02-spring-ioc.md)
    * [스프링 AOP](spring-keesun-baik/spring-framework-introduction/03-spring-aop.md)
    * [스프링 PSA](spring-keesun-baik/spring-framework-introduction/04-spring-psa.md)

* [스프링 프레임워크 핵심 기술](spring-keesun-baik/spring-framework-core/README.md)
    * [IoC 컨테이너와 빈](spring-keesun-baik/spring-framework-core/01-ioc-bean/README.md)
        * [스프링 IoC 컨테이너와 빈](spring-keesun-baik/spring-framework-core/01-ioc-bean/01-ioc-container-and-bean.md)
        * [ApplicationContext와 빈 설정](spring-keesun-baik/spring-framework-core/01-ioc-bean/02-application-context.md)
        * [@Autowired](spring-keesun-baik/spring-framework-core/01-ioc-bean/03-autowired.md)
        * [@Component와 컴포넌트 스캔](spring-keesun-baik/spring-framework-core/01-ioc-bean/04-component.md)
        * [빈의 스코프](spring-keesun-baik/spring-framework-core/01-ioc-bean/05-bean-scope.md)
        * [Environment](spring-keesun-baik/spring-framework-core/01-ioc-bean/06-environment.md)
        * [MessageSource](spring-keesun-baik/spring-framework-core/01-ioc-bean/07-message-source.md)
        * [ApplicationEventPublisher](spring-keesun-baik/spring-framework-core/01-ioc-bean/08-publisher.md)
        * [ResourceLoader](spring-keesun-baik/spring-framework-core/01-ioc-bean/09-resource-loader.md)
    * [Resource/Validation](spring-keesun-baik/spring-framework-core/02-resource-validation/README.md)
        * [Resource 추상화](spring-keesun-baik/spring-framework-core/02-resource-validation/01-resource-abstraction.md)
        * [validation 추상화](spring-keesun-baik/spring-framework-core/02-resource-validation/02-validation-abstraction.md)
    * [데이터 바인딩](spring-keesun-baik/spring-framework-core/03-data-binding/README.md)
    * [SpEL](spring-keesun-baik/spring-framework-core/04-spel.md)
    * [스프링 AOP](spring-keesun-baik/spring-framework-core/05-spring-aop.md)
    * [Null-Safety](spring-keesun-baik/spring-framework-core/06-null-safety.md)

## 백기선 테스트 코드 강의 <a id="inflearn-the-java-test"></a>

* [JUnit 5](inflearn-the-java-test/01-junit-5)
    * [JUnit 시작하기](inflearn-the-java-test/01-junit-5/01-introduction.md)
    * [JUnit 시작하기](inflearn-the-java-test/01-junit-5/02-assertion.md)
* [Mockito](inflearn-the-java-test/02-mockito.md)

## 백기선 THE JAVA <a id="inflearn-the-java"></a>

* [JVM 이해하기](inflearn-the-java/01-understanding-of-jvm)
    * [자바, JVM, JDK, JRE](inflearn-the-java/01-understanding-of-jvm/01-java-jvm-jdk-jre.md)
    * [JVM 구조](inflearn-the-java/01-understanding-of-jvm/02-jvm-structure.md)
    * [클래스 로더](inflearn-the-java/01-understanding-of-jvm/03-class-loader.md)
    * [Heap](inflearn-the-java/01-understanding-of-jvm/04-heap.md)
    * [Garbage Collector](inflearn-the-java/01-understanding-of-jvm/05-garbage-collector.md)
* [리플렉션](inflearn-the-java/03-reflection)
    * [클래스 정보 조회](inflearn-the-java/03-reflection/01-class-info.md)

[comment]: <> (## 지식 벌크업 <a id="interview"></a>)

[comment]: <> (* [JAVA]&#40;interview/jvm-and-java/README.md&#41;)

[comment]: <> (  * [JVM과 메모리]&#40;interview/jvm-and-java/jvm.md&#41;)

[comment]: <> (  * [Garbage Collector]&#40;interview/jvm-and-java/garbage-collector.md&#41;)

[comment]: <> (* [스프링]&#40;interview/spring/README.md&#41;)

[comment]: <> (  * [IoC와 DI]&#40;interview/spring/ioc-di.md&#41;)

[comment]: <> (  * [JPA]&#40;interview/spring/jpa.md&#41;)

[comment]: <> (  * [WebFlux]&#40;interview/spring/web-flux.md&#41;)

[comment]: <> (* [디자인패턴]&#40;interview/design-pattern/README.md&#41;)

[comment]: <> (  * [싱글톤 패턴]&#40;interview/design-pattern/singleton-pattern.md&#41;)

[comment]: <> (  * [템플릿 메소드 패턴]&#40;interview/design-pattern/template-method-pattern.md&#41;)

[comment]: <> (* [자료구조 & 알고리즘]&#40;interview/algorithm/README.md&#41;)

[comment]: <> (  * [시간/공간 복잡도]&#40;interview/algorithm/complexity.md&#41;)

[comment]: <> (  * [리스트]&#40;interview/algorithm/list.md&#41;)

[comment]: <> (  * [우선 순위 큐]&#40;interview/algorithm/priority-queue.md&#41;)

[comment]: <> (  * [해시 테이블]&#40;interview/algorithm/hash-table.md&#41;)

[comment]: <> (  * [매칭 알고리즘]&#40;interview/algorithm/matching.md&#41;)

[comment]: <> (  * [랭킹 알고리즘]&#40;interview/algorithm/ranking.md&#41;)

[comment]: <> (* [데이터 베이스]&#40;interview/database/README.md&#41;)

[comment]: <> (  * [데이터 베이스 설계]&#40;interview/database/dbms-design.md&#41;)

[comment]: <> (  * [NoSQL]&#40;interview/database/no-sql.md&#41;)

[comment]: <> (  * [서버 확장]&#40;interview/database/scaling.md&#41;)

[comment]: <> (  * [샤딩]&#40;interview/database/sharding.md&#41;)

[comment]: <> (  * [GraphQL]&#40;interview/database/graphql.md&#41;)

[comment]: <> (* [네트워크]&#40;interview/network/README.md&#41;)

[comment]: <> (  * [CDN]&#40;interview/network/cdn.md&#41;)

[comment]: <> (  * [OSI 7계층]&#40;interview/network/osi-layers.md&#41;)

[comment]: <> (  * [HTTP 프로토콜]&#40;interview/network/http-protocol.md&#41;)

[comment]: <> (  * [HTTP 상태 코드]&#40;interview/network/status-code.md&#41;)

[comment]: <> (  * [웹 서버]&#40;interview/network/web-server.md&#41;)

[comment]: <> (  * [WAS]&#40;interview/network/was.md&#41;)

[comment]: <> (  * [REST API]&#40;interview/network/rest-api.md&#41;)

[comment]: <> (  * [서블릿]&#40;interview/network/servlet.md&#41;)

[comment]: <> (  * [상태 정보]&#40;interview/network/state.md&#41;)

[comment]: <> (* [아키텍처]&#40;interview/architecture/README.md&#41;)

[comment]: <> (  * [Layered Architecture]&#40;interview/architecture/layered-architecture.md&#41;)

[comment]: <> (  * [MSA]&#40;interview/architecture/micro-service/README.md&#41;)

[comment]: <> (    * [MSA의 구성]&#40;interview/architecture/micro-service/architecture.md&#41;)

[comment]: <> (* [클라우드 & 기타]&#40;interview/cloud/README.md&#41;)

[comment]: <> (  * [AWS]&#40;interview/cloud/aws.md&#41;)

[comment]: <> (  * [Elastic Beanstalk]&#40;interview/cloud/elastic-beanstalk.md&#41;)

[comment]: <> (  * [ElasticCache]&#40;interview/cloud/elasticache-redis.md&#41;)

[comment]: <> (  * [ECS]&#40;interview/cloud/ecs.md&#41;)

[comment]: <> (  * [SQS]&#40;interview/cloud/sqs.md&#41;)

[comment]: <> (  * [docker]&#40;interview/cloud/docker.md&#41;)

[comment]: <> (  * [kubernetes]&#40;interview/cloud/kubernetes.md&#41;)

[comment]: <> (## Spring & Hibernate for Beginners <a id="spring-hibernate-for-beginners"></a>)

[comment]: <> (* [Spring Overview]&#40;spring-hibernate-for-beginners/spring-overview.md&#41;)

[comment]: <> (* [Spring with XML]&#40;spring-hibernate-for-beginners/spring-with-xml/README.md&#41;)

[comment]: <> (    * [Inversion of Control]&#40;spring-hibernate-for-beginners/spring-with-xml/inversion-of-control.md&#41;)

[comment]: <> (    * [Dependency Injection]&#40;spring-hibernate-for-beginners/spring-with-xml/dependency-injection.md&#41;)

[comment]: <> (    * [Constructor Injection]&#40;spring-hibernate-for-beginners/spring-with-xml/constructor-injection.md&#41;)

[comment]: <> (    * [Setter Injection]&#40;spring-hibernate-for-beginners/spring-with-xml/setter-injection.md&#41;)

[comment]: <> (    * [Bean Scopes]&#40;spring-hibernate-for-beginners/spring-with-xml/bean-scopes.md&#41;)

[comment]: <> (* [Spring with Annotations]&#40;spring-hibernate-for-beginners/spring-with-annotations/README.md&#41;)

[comment]: <> (    * [Dependency Injection]&#40;spring-hibernate-for-beginners/spring-with-annotations/dependency-injection.md&#41;)

[comment]: <> (    * [Constructor Injection]&#40;spring-hibernate-for-beginners/spring-with-annotations/constructor-injection.md&#41;)

[comment]: <> (## 실전 자바 스프트웨어 개발 <a id="real-world-software-development"></a>)

[comment]: <> (* [2장. 입출금 내역 분석기]&#40;real-world-software-development/01-bank-transaction-analyzer/README.md&#41;)
