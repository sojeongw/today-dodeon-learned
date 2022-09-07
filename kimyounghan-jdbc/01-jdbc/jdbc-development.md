# JDBC 개발

## 등록

{% tabs %} {% tab title="Member.java" %}

```java

@Data
public class Member {
    private String memberId;
    private int money;

    public Member() {
    }

    public Member(String memberId, int money) {
        this.memberId = memberId;
        this.money = money;
    }
}
```

{% endtab %} {% tab title="MemberRepositoryV0.java" %}

```java

@Slf4j
public class MemberRepositoryV0 {
    public Member save(Member member) throws SQLException {
        String sql = "insert into member(member_id, money) values(?, ?)";

        Connection con = null;
        PreparedStatement pstmt = null;

        try {
            // 커넥션 획득
            con = getConnection();
            pstmt = con.prepareStatement(sql);

            // SQL 첫 번째 값 지정
            pstmt.setString(1, member.getMemberId());
            // SQL 두 번째 값 지정
            pstmt.setInt(2, member.getMoney());

            // 실제 DB에 전달한 뒤 영향받은 row 수를 int로 반환
            pstmt.executeUpdate();
            return member;
        } catch (SQLException e) {
            log.error("db error", e);
            throw e;
        } finally {
            // 리소스 정리
            close(con, pstmt, null);
        }
    }

    private void close(Connection con, Statement stmt, ResultSet rs) {
        if (rs != null) {
            try {
                rs.close();
            } catch (SQLException e) {
                log.info("error", e);
            }
        }
        if (stmt != null) {
            try {
                stmt.close();
            } catch (SQLException e) {
                log.info("error", e);
            }
        }
        if (con != null) {
            try {
                con.close();
            } catch (SQLException e) {
                log.info("error", e);
            }
        }
    }

    private Connection getConnection() {
        return DBConnectionUtil.getConnection();
    }

}
```

{% endtab %}{% tab title="MemberRepositoryV0Test.java" %}

```java
class MemberRepositoryV0Test {
    MemberRepositoryV0 repository = new MemberRepositoryV0();

    @Test
    void crud() throws SQLException {
        // save
        Member member = new Member("memberV0", 10000);
        repository.save(member);
    }
}
```

{% endtab %} {% endtabs %}

- 외부 리소스는 반드시 정리해준다.
    - finally에 구현해 언제든 실행할 수 있게 한다.
    - 만약 이 부분을 놓치게 되면 커넥션이 끊어지지 않고 계속 유지된다.
        - 이런 것을 리소스 누수라고 한다.
        - 커넥션 부족으로 장애가 발생할 수 있다.
- PreparedStatement
    - Statement의 자식 타입
    - ?로 파라미터 바인딩을 하게 해준다.
    - SQL Injection 공격을 예방하려면 PreparedStatement를 통한 파라미터 바인딩을 해야 한다.

## 조회

{% tabs %} {% tab title="MemberRepositoryV0.java" %}

```java

@Slf4j
public class MemberRepositoryV0 {

    public Member findById(String memberId) throws SQLException {
        String sql = "select * from member where member_id = ?";

        Connection con = null;
        PreparedStatement pstmt = null;
        ResultSet rs = null;

        try {
            con = getConnection();
            pstmt = con.prepareStatement(sql);

            pstmt.setString(1, memberId);
            // 조회용 메서드
            // 결과를 ResultSet에 담아 반환
            rs = pstmt.executeQuery();

            if (rs.next()) {
                Member member = new Member();
                member.setMemberId(rs.getString("member_id"));
                member.setMoney(rs.getInt("money"));

                return member;
            } else {
                throw new NoSuchElementException("member not found memberId=" + memberId);
            }
        } catch (SQLException e) {
            log.error("db error", e);
            throw e;
        } finally {
            close(con, pstmt, rs);
        }
    }
}
```

{% endtab %} {% endtabs %}

### ResultSet

```text
ResultSet executeQuery() throws SQLException;
```

- select 쿼리의 결과가 순서대로 들어간다.
    - `select member_id, money`
    - member_id, money 라는 이름으로 데이터가 저장된다.
- 내부 Cursor를 사용해 이동하면서 다음 데이터를 조회한다.
    - rs.next()
        - 다음 커서로 이동한다.
        - 최초에 데이터를 가리키고 있지 않기 때문에 한 번은 꼭 실행해야 한다.
        - 결과가 true면 데이터가 있고 false면 없다.
    - rs.getString("member_id")
        - 커서가 가리키고 있는 위치의 member_id를 String으로 반환한다.

![](../../.gitbook/assets/kimyounghan-jdbc/01/스크린샷%202022-09-17%20오후%204.49.09.png)

- 1-1
    - rs.next() 를 호출한다.
- 1-2
    - cursor가 다음으로 이동한다.
    - cursor가 가리키는 데이터가 있으므로 true를 반환한다.
- 2-1
    - rs.next() 를 호출한다.
- 2-2
    - cursor가 다음으로 이동한다.
    - cursor가 가리키는 데이터가 있으므로 true를 반환한다.
- 3-1
    - rs.next() 를 호출한다.
- 3-2
    - cursor가 다음으로 이동한다.
    - cursor가 가리키는 데이터가 없으므로 false를 반환한다.

{% tabs %} {% tab title="MemberRepositoryV0Test.java" %}

```java
@Slf4j
class MemberRepositoryV0Test {
    MemberRepositoryV0 repository = new MemberRepositoryV0();

    @Test
    void crud() throws SQLException {
        // save
        Member member = new Member("memberV0", 10000);
        repository.save(member);

        // findById
        Member findMember = repository.findById(member.getMemberId());
        log.info("findMember={}", findMember);

        assertThat(findMember).isEqualTo(member);
    }
}
```

{% endtab %} {% endtabs %}

```text
MemberRepositoryV0Test - findMember=Member(memberId=memberV0, money=10000)
```