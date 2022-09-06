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