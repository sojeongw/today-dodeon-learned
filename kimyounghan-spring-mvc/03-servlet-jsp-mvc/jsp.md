# JSP로 회원 관리 만들기

```groovy
    // JSP
implementation 'org.apache.tomcat.embed:tomcat-embed-jasper'
implementation 'javax.servlet:jstl'
```

JSP를 사용하기 위해 라이브러리를 추가한다.

## 회원 등록 폼

```html
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
<form action="/jsp/members/save.jsp" method="post"> username: <input type="text" name="username"/> age: <input
        type="text" name="age"/>
    <button type="submit">전송</button>
</form>
</body>
</html>
```

맨 첫줄은 JSP 문서라는 의미다. JSP 문서는 이렇게 시작한다.

첫 줄을 제외하고는 HTML과 완전히 똑같다. JSP는 서버 내부에서 서블릿으로 변환되며, 이전에 만들었던 MemberFormServlet과 거의 비슷한 모습으로 변환된다.

```text
http://localhost:8080/jsp/members/new-form.jsp
```

접속할 땐 jsp를 붙여줘야 한다.

## 회원 저장

```html
<%@ page import="hello.servlet.domain.member.MemberRepository" %>
<%@ page import="hello.servlet.domain.member.Member" %>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%
// request, response 사용 가능
MemberRepository memberRepository = MemberRepository.getInstance();
System.out.println("save.jsp");
String username = request.getParameter("username");
int age = Integer.parseInt(request.getParameter("age"));
Member member = new Member(username, age);
System.out.println("member = " + member);
memberRepository.save(member);
%>
<html>
<head>
    <meta charset="UTF-8">
</head>
<body>
성공
<ul>
    <li>id=<%=member.getId()%>
    </li>
    <li>username=<%=member.getUsername()%>
    </li>
    <li>age=<%=member.getAge()%>
    </li>
</ul>
<a href="/index.html">메인</a>
</body>
</html>
```

- `<%@ page import="hello.servlet.domain.member.MemberRepository" %>`는 import 문이다. 
- `<% ~~ %>`는 자바 코드를 입력한다.
- `<%= ~~ %>`는 바바 코드를 출력한다.

## 회원 목록

```html
<%@ page import="java.util.List" %>
<%@ page import="hello.servlet.domain.member.MemberRepository" %>
<%@ page import="hello.servlet.domain.member.Member" %>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%
    MemberRepository memberRepository = MemberRepository.getInstance();
    List<Member> members = memberRepository.findAll();
%>
<html>
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<a href="/index.html">메인</a>
<table>
    <thead>
    <th>id</th>
    <th>username</th>
    <th>age</th>
    </thead>
    <tbody>
    <%
        for (Member member : members) {
            out.write("    <tr>");
            out.write("         <td>" + member.getId() + "</td>");
            out.write("         <td>" + member.getUsername() + "</td>");
            out.write("         <td>" + member.getUsername() + "</td>");
            out.write("    </tr>");
        }
    %>
    </tbody>
</table>
</body>
</html>
```

## 서블릿과 JSP의 한계

서블릿은 HTML 작업이 자바 코드에 섞여서 지저분했다. JSP는 뷰 생성 작업을 JSP가 가져가고 중간에 동적으로 변경이 필요한 부분만 자바 코드를 사용한다.

하지만 회원 저장을 보면 회원을 저장하는 비즈니스 로직과 HTML 뷰 영역이 같이 있다. JSP가 너무 많은 역할을 한다.

## MVC 패턴의 등장

비즈니스 로직은 서블릿처럼 다른 곳에서 처리하고 JSP는 화면을 그리는 일에 집중하기 위해 MVC 패턴이 등장했다.