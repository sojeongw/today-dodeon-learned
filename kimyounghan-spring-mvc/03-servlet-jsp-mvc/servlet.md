# 서블릿으로 회원 관리 앱 만들기

- 회원 저장
- 회원 목록 조회

## 회원 도메인 모델

{% tabs %} {% tab title="Member.java" %}

```java
@Getter @Setter
@NoArgsConstructor
public class Member {
    private Long id;
    private String username;
    private int age;

    public Member(String username, int age) {
        this.username = username;
        this.age = age;
    }
}

```

{% endtab %} {% tab title="MemberRepository.java" %}

```java
/**
 * 동시성 문제가 고려되어 있지 않음, 실무에서는 ConcurrentHashMap, AtomicLong 사용 고려
 */
public class MemberRepository {

    // static이어야 딱 하나만 생성된다.
    private static Map<Long, Member> store = new HashMap<>();
    private static long sequence = 0L;

    // 싱글턴으로 생성한다.
    // private으로 선언해 아무도 새로 생성하지 못하게 한다.
    private static final MemberRepository instance = new MemberRepository();

    public static MemberRepository getInstance() {
        return instance;
    }

    // 싱글턴은 객체를 단 하나만 생성해야 하므로 생성자를 private으로 막는다.
    private MemberRepository() {
    }

    public Member save(Member member) {
        member.setId(++sequence);
        store.put(member.getId(), member);
        return member;
    }

    public Member findById(Long id) {
        return store.get(id);
    }

    public List<Member> findAll() {
        // store의 모든 values를 꺼내서 기존 리스트를 건드리지 않고 새로운 리스트로 반환한다.
        return new ArrayList<>(store.values());
    }

    public void clearStore() {
        store.clear();
    }
}
```

{% endtab %} {% tab title="MemberRepositoryTest.java" %}

```java
class MemberRepositoryTest {

    MemberRepository memberRepository = MemberRepository.getInstance();

    @AfterEach
    void afterEach() {
        memberRepository.clearStore();
    }

    @Test
    void save() {
        // given
        Member member = new Member("hello", 20);

        // when
        Member savedMember = memberRepository.save(member);

        // then
        Member findMember = memberRepository.findById(savedMember.getId());
        AssertionsForClassTypes.assertThat(findMember).isEqualTo(savedMember);
    }

    @Test
    void findAll() {
        // given
        Member member1 = new Member("member1", 20);
        Member member2 = new Member("member2", 30);

        memberRepository.save(member1);
        memberRepository.save(member2);

        // when
        List<Member> result = memberRepository.findAll();

        // then
        assertThat(result.size()).isEqualTo(2);
        assertThat(result).contains(member1, member2);
    }
}
```

{% endtab %} {% endtabs %}

## 회원 등록 폼

```java
@WebServlet(name = "memberFormServlet", urlPatterns = "/servlet/members/new-form")
public class MemberFormServlet extends HttpServlet {

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        response.setContentType("text/html");
        response.setCharacterEncoding("utf-8");

        PrintWriter w = response.getWriter();
        w.write("<!DOCTYPE html>\n" +
                "<html>\n" +
                "<head>\n" +
                "    <meta charset=\"UTF-8\">\n" +
                "    <title>Title</title>\n" +
                "</head>\n" +
                "<body>\n" +
                "<form action=\"/servlet/members/save\" method=\"post\">\n" +
                "    username: <input type=\"text\" name=\"username\" />\n" +
                "    age:      <input type=\"text\" name=\"age\" />\n" +
                " <button type=\"submit\">전송</button>\n" + "</form>\n" +
                "</body>\n" +
                "</html>\n");
    }
}

```

HTML을 자바 코드로 작성해야 해서 불편하다.

## 회원 저장

```java
@WebServlet(name = "memberSaveServlet", urlPatterns = "/servlet/members/save")
public class MemberSaveServlet extends HttpServlet {

    private MemberRepository memberRepository = MemberRepository.getInstance();

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        System.out.println("MemberSaveServlet.service");
        
        // 파라미터로 받은 값 꺼내기
        String username = request.getParameter("username");
        int age = Integer.parseInt(request.getParameter("age"));

        Member member = new Member(username, age);
        System.out.println("member = " + member);
        memberRepository.save(member);

        response.setContentType("text/html");
        response.setCharacterEncoding("utf-8");

        PrintWriter writer = response.getWriter();
        writer.write("<html>\n" +
                "<head>\n" +
                " <meta charset=\"UTF-8\">\n" + "</head>\n" +
                "<body>\n" +
                "성공\n" +
                "<ul>\n" +
                "    <li>id="+member.getId()+"</li>\n" +
                "    <li>username="+member.getUsername()+"</li>\n" +
                " <li>age="+member.getAge()+"</li>\n" + "</ul>\n" +
                "<a href=\"/index.html\">메인</a>\n" + "</body>\n" +
                "</html>");
    }
}
```

1. 파라미터를 조회해서 Member 객체를 만든다.
2. Member 객체를 MemberRepository를 통해 저장한다.
3. Member 객체를 사용해 결과 화면용 HTML을 동적으로 만들어서 응답한다.

## 회원 목록

```java
@WebServlet(name = "memberListServlet", urlPatterns = "/servlet/members")
public class MemberListServlet extends HttpServlet {

    private MemberRepository memberRepository = MemberRepository.getInstance();

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        response.setContentType("text/html");
        response.setCharacterEncoding("utf-8");

        List<Member> members = memberRepository.findAll();

        PrintWriter writer = response.getWriter();

        writer.write("<html>");
        writer.write("<head>");
        writer.write("    <meta charset=\"UTF-8\">");
        writer.write("    <title>Title</title>");
        writer.write("</head>");
        writer.write("<body>");
        writer.write("<a href=\"/index.html\">메인</a>");
        writer.write("<table>");
        writer.write("    <thead>");
        writer.write("    <th>id</th>");
        writer.write("    <th>username</th>");
        writer.write("    <th>age</th>");
        writer.write("    </thead>");
        writer.write("    <tbody>");

        for (Member member : members) {
            writer.write("    <tr>");
            writer.write("      <td>" + member.getId() + "</td>");
            writer.write("      <td>" + member.getUsername() + "</td>");
            writer.write("      <td>" + member.getAge() + "</td>");
            writer.write("    </tr>");
        }

        writer.write("    </tbody>");
        writer.write("</table>");
        writer.write("</body>");
        writer.write("</html>");
    }
}
```

1. memberRepository.findAll()로 모든 회원을 조회한다.
2. 회원 목록 HTML을 for 루프로 회원 수 만큼 동적으로 생성하고 응답한다.

## 템플릿 엔진

서블릿과 자바 코드만으로 HTML을 만들어보았다. 서블릿 덕에 동적으로 원하는 HTML을 만들 수 있었다. 정적인 HTML은 회원의 저장 결과나 회원 목록같이 계속 바뀌는 결과를 HTML로 만들지 못할 것이다.

하지만 코드는 굉장히 복잡하다. 이때 템플릿 엔진을 이용하면 HTML 문서에서 필요한 곳만 코드를 적용해 동적으로 변경할 수 있다. 여기서는 스프링과 잘 통합되는 Thymeleaf를 사용한다.