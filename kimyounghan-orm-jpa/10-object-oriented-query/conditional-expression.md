# 조건식

## 기본 조건식

```sql
select case
       when m.age <= 10 then '학생요금'
       when m.age >= 60 then '경로요금'
       else '일반요금'
       end
from Member m
```

쿼리에 `<=`같은 조건을 넣는다.

```java
public class JpaMain {

  public static void main(String[] args) {

    Member member = new Member();
    member.setName("member");
    em.persist(member);

    em.flush();
    em.clear();

    String query = "select\n"
        + "case when m.age <= 10 then '학생요금' when m.age >= 60 then '경로요금'\n"
        + "else '일반요금'\n"
        + "end\n"
        + "from Member m";

    List<String> result = em.createQuery(query, String.class).getResultList();

    for (String s : result) {
      System.out.println("s = " + s);
    }

    tx.commit();
  }
}
```

![](../../.gitbook/assets/kimyounghan-orm-jpa/10/screenshot%202021-05-23%20오후%2012.54.15.png)

## 단순 조건식

```sql
select case t.name
       when '팀A' then '인센티브110%'
       when '팀B' then '인센티브120%'
       else '인센티브105%'
       end
from Team t
```

정확하게 매칭을 해서 쿼리를 뽑아내는 단순한 식이다.

## COALESCE

```sql
select coalesce(m.username, '이름 없는 회원')
from Member m
```

- 하나씩 조회해서 null이 아니면 반환

```java
public class JpaMain {

  public static void main(String[] args) {
   
      Member member = new Member();
      member.setName("member");
      em.persist(member);

      em.flush();
      em.clear();

      String query = "select coalesce(m.name, '이름 없는 회원')\n"
          + "from Member m";

      List<String> result = em.createQuery(query, String.class).getResultList();

      for (String s : result) {
        System.out.println("s = " + s);
      }

      tx.commit();
  }
}

```

![](../../.gitbook/assets/kimyounghan-orm-jpa/10/screenshot%202021-05-23%20오후%2012.58.04.png)

`setName()`을 해주면 이름을 출력한다.

```java
public class JpaMain {

  public static void main(String[] args) {
   
      Member member = new Member();
      em.persist(member);

      em.flush();
      em.clear();

      String query = "select coalesce(m.name, '이름 없는 회원')\n"
          + "from Member m";

      List<String> result = em.createQuery(query, String.class).getResultList();

      for (String s : result) {
        System.out.println("s = " + s);
      }

      tx.commit();
  }
}

```

![](../../.gitbook/assets/kimyounghan-orm-jpa/10/screenshot%202021-05-23%20오후%2012.58.19.png)

`setName()`을 해주지 않으면 이름 없는 회원으로 출력된다.

## NULLIF

```sql
select NULLIF(m.username, '관리자')
from Member m
```

- 두 값이 같으면 null 반환, 다르면 첫번째 값 반환

```java
public class JpaMain {

  public static void main(String[] args) {
    
      Member member = new Member();
      member.setName("관리자");
      em.persist(member);

      em.flush();
      em.clear();

      String query = "select NULLIF(m.name, '관리자')\n"
          + "from Member m";

      List<String> result = em.createQuery(query, String.class).getResultList();

      for (String s : result) {
        System.out.println("s = " + s);
      }

      tx.commit();
  }
}

```

![](../../.gitbook/assets/kimyounghan-orm-jpa/10/screenshot%202021-05-23%20오후%201.01.17.png)

`setName("관리자")`를 하면 두 값이 같으므로 null을 반환한다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/10/screenshot%202021-05-23%20오후%201.01.03.png)

다르면 `m.name`을 반환한다.

관리자의 이름을 숨겨야할 때 사용할 수 있다.