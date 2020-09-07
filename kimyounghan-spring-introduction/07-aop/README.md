# AOP

## AOP가 필요한 상황

만약 모든 메서드의 호출 시간을 측정하고 싶다면 어떻게 해야할까?

![](../../.gitbook/assets/kimyounghan-spring-introduction/07/스크린샷%202021-03-09%20오전%209.15.42.png)

{% tabs %} {% tab title="MemberService.java" %}

```java

@Transactional
public class MemberService {

  /**
   * 회원가입
   */
  public Long join(Member member) {
    // 시간 측정하는 기능 추가
    long start = System.currentTimeMillis();
    try {
      validateDuplicateMember(member);
      memberRepository.save(member);
      return member.getId();
    } finally {
      long finish = System.currentTimeMillis();
      long timeMs = finish - start;
      System.out.println("join " + timeMs + "ms");
    }
  }

  /**
   *전체 회원 조회
   */
  public List<Member> findMembers() {
    // 반복해서 추가
    long start = System.currentTimeMillis();
    try {
      return memberRepository.findAll();
    } finally {
      long finish = System.currentTimeMillis();
      long timeMs = finish - start;
      System.out.println("findMembers " + timeMs + "ms");
    }
  }
}
```

{% endtab %} {% endtabs %}

- 회원 가입, 조회에서 시간 측정 기능은 핵심 관심 사항이 아니라 공통의 관심 사항이다. 
- 이런 로직이 핵심 비즈니스 로직과 섞이면 유지보수가 어려워진다. 
- 그렇다고 이걸 메서드로 따로 빼서 공통 로직으로 만드는 것도 어렵다. 
- 만약 시간 측정 로직이 변경되면 모든 로직을 찾아가면서 변경해야 한다.

