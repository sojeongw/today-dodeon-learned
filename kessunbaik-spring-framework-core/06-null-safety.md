# Null-Safety

- 스프링 프레임워크 5에 추가된 Null 관련 애너테이션
- @NonNull
- @Nullable

```java
@NonNullApi
package me.dodeon;

import org.springframework.lang.NonNullApi;
```

- @NonNullApi
    - 패키지 레벨 설정
- @NonNullFields
    - 패키지 레벨 설정

## 목적 

- 툴의 지원을 받아 컴파일 시점에 최대한 NPE를 방지하기 위한 것이다.
