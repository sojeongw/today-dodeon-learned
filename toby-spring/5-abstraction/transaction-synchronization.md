# 트랜잭션 동기화

## Connection 파라미터 제거

`Connection`을 파라미터로 계속 전달해서 DAO를 호출하는 것만은 피해보자.

![](../../.gitbook/assets/toby/screenshot%202020-04-07%20오전%2012.13.45.png)

트랜잭션 동기화를 사용하면 Connection 오브젝트를 특별한 저장소에 보관해두고 사용할 수 있다.

1. `UserService`가 `Connection`을 생성한다.
2. 트랜잭션 동기화 저장소에 저장해두고 `Connection`의 `setAutoCommit(false)`를 호출해 트랜잭션을 시작시킨다.
3. 첫 번째 `update()`가 호출된다.
4. `JdbcTemplate` 메서드에서 트랜잭션 동기화 저장소에 현재 트랜잭션을 가진 `Connection` 오브젝트가 있는지 확인하고 가져온다.
5. `Connection`으로 `PreparedStatement`를 만들어 SQL을 실행한다. 첫번째 DB 작업을 마쳐도 여전히 `Connection`은 열려있고 트랜잭션이 저장소에 저장되어 있다.
6. 두 번째 update()를 호출한다.
7. `Connection`을 가져온다.
8. 사용한다.
9. 마지막 `update()`를 호출한다.
10. `Connection`을 가져온다.
11. 사용한다.
12. 모든 작업이 끝나면 commit()으로 트랜잭션을 완료시킨다.
13. Connection을 제거한다.

트랜잭션 동기화 저장소는 작업 스레드마다 독립적으로 Connection 오브젝트를 저장하고 관리하므로 다중 사용자를 처리하는 서버의 멀티스레드 환경에서도 충돌날 염려가 없다.