---
description: DB에 연결해서 SQL문을 실행하는 코드를 작성해보자.
---

# 초난감 DAO

코드를 보면 DB 커넥션과 statement 실행이 add\(\)와 get\(\)에 중복으로 들어가있다. 이제부터 여러 원칙에 따라 리팩토링을 진행해보자.

```java
public class UserDAO {

    public void add(User user){
        // DB 커넥션
        Connection c = DriverManager()...

        // statement 실행
        PreparedStatement ps = ...
    }

    public User get(String id){
         // DB 커넥션
         Connection c = DriverManager()...

         // statement 실행
         PreparedStatement ps = ...       
    }
}
```

