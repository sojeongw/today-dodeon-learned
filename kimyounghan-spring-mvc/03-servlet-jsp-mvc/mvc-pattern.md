# MVC 패턴
## 개요
### 너무 많은 역할

서블릿이나 JSP 하나로 비즈니스 로직과 뷰 렌더링까지 다 처리하면 너무 많은 역할을 해서 유지 보수가 어렵다.

### 변경의 라이프 사이클

둘 사이에 변경 라이프 사이클이 다르다는 것도 큰 문제다. UI 일부를 수정하는 일과 비즈니스 로직을 수정하는 일은 서로 다르게 발생할 확률이 높고 대부분 서로 영향을 주지 않는다.

### 기능 특화

JSP 같은 뷰 템플릿은 화면을 렌더링하는 데에 최적화되어 있기 때문에 이 업무만 담당하는 것이 효과적이다.

## Model View Controller
### Model

- 뷰에 출력할 데이터를 담는다.
- 뷰에 필요한 데이터를 모두 모델에 담아 전달한다.
- 뷰는 비즈니스 로직이나 데이터 접근에 대해 몰라도 된다.

### View

- 모델에 담긴 데이터를 사용해 화면을 그리는 일에 집중한다.

### Controller

- HTTP 요청을 받아서 파라미터를 검증한다.
- 비즈니스 로직을 실행한다.
- 뷰에 전달할 결과 데이터를 조회해 모델에 담는다.

![](../../.gitbook/assets/kimyounghan-spring-mvc/03/screenshot%202021-06-19%20오후%207.21.02.png)

MVC 이전에는 모든 로직을 하나로 처리했다.

![](../../.gitbook/assets/kimyounghan-spring-mvc/03/screenshot%202021-06-19%20오후%207.21.13.png)

고객에 요청을 하면 컨트롤러에서 비즈니스 로직을 진행하고 그 데이터를 모델에 담아 뷰 로직에 넘긴다. 이때 뷰가 실행되면서 모델에 담긴 데이터로 화면에 뿌려진다.

![](../../.gitbook/assets/kimyounghan-spring-mvc/03/screenshot%202021-06-19%20오후%207.21.18.png)

실제로는 비즈니스 로직이 서비스, 리포지토리에서 처리된다. 컨트롤러에 두면 너무 많은 역할을 담당하게 되기 때문이다. 그래서 서비스 계층을 별도로 만들어 비즈니스 로직을 처리한다. 컨트롤러는 비즈니스 로직이 있는 서비스를 호출하는 역할을 한다. 

1. 사용자가 호출한다.
2. 컨트롤러는 HTTP 스펙과 파라미터를 확인하고 서비스를 호출해 비즈니스 로직을 수행한다. 
3. 그 결과를 받아 모델에 데이터를 저장하고 뷰에 넘긴다. 
4. 뷰는 그 값을 가져다 쓴다.
5. 결과가 출력된다.

## 적용

서블릿이 컨트롤러, JSP가 뷰로 사용되는 MVC 패턴을 적용해본다. 

모델은 HttpServletRequest 객체가 담당한다. request.setAttribute(), request.getAttribute()를 사용하면 데이터를 보관, 조회할 수 있다.

### 회원 등록

{% tabs %} {% tab title="MvcMemberFormServlet.java" %}

```java
@WebServlet(name = "mvcMemberFormServlet", urlPatterns = "/servlet-mvc/members/new-form")
public class MvcMemberFormServlet extends HttpServlet {

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        String viewPath = "/WEB-INF/views/new-form.jsp";
        RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);

        // 다른 서블릿이나 JSP로 이동하는 기능
        // 서버 내부에서 다시 호출이 발생한다.
        dispatcher.forward(request, response);
    }
}
```

{% endtab %} {% tab title="new-form.jsp" %}

```html
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<!-- 상대경로 사용. [현재 URL이 속한 계층 경로 + /save] -->
<form action="save" method="post">
    username: <input type="text" name="username"/> age: <input type="text" name="age"/>
    <button type="submit">전송</button>
</form>
</body>
</html>
```

{% endtab %} {% endtabs %}

redirect는 클라이언트인 브라우저에 응답이 나갔다가 클라이언트가 redirect 경로로 다시 요청한다. 따라서 클라이언트가 인지할 수 있고 URL 경로도 실제로 변경된다.

반면, forward는 서버 내부에서 일어나는 호출이기 때문에 클라이언트가 전혀 인지하지 못한다.

JSP는 `/WEB-INF`에 생성한다. 이 경로에 있으면 외부에서 직접 JSP를 호출하지 못하고 컨트롤러를 통해야만 한다.

### 회원 저장

{% tabs %} {% tab title="MvcMemberSaveServlet.java" %}

```java
@WebServlet(name = "mvcMemberSaveServlet", urlPatterns = "/servlet-mvc/members/save")
public class MvcMemberSaveServlet extends HttpServlet {
    private MemberRepository memberRepository = MemberRepository.getInstance();

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        String username = request.getParameter("username");
        int age = Integer.parseInt(request.getParameter("age"));

        Member member = new Member(username, age);
        System.out.println("member = " + member);
        memberRepository.save(member);

        // Model에 데이터를 보관한다. 
        request.setAttribute("member", member);
        String viewPath = "/WEB-INF/views/save-result.jsp";
        RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
        
        dispatcher.forward(request, response);
    }
}
```

{% endtab %} {% tab title="save-result.jsp" %}

```html
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <meta charset="UTF-8">
</head>
<body> 성공
<ul>
    <li>id=${member.id}</li>
    <li>username=${member.username}</li>
    <li>age=${member.age}</li>
</ul>
<a href="/index.html">메인</a>
</body>
</html>
```

{% endtab %} {% endtabs %}

### 회원 목록 조회

{% tabs %} {% tab title="MvcMemberListServlet.java" %}

```java
@WebServlet(name = "mvcMemberListServlet", urlPatterns = "/servlet-mvc/members")
public class MvcMemberListServlet extends HttpServlet {
    private MemberRepository memberRepository = MemberRepository.getInstance();

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        System.out.println("MvcMemberListServlet.service");

        List<Member> members = memberRepository.findAll();
        request.setAttribute("members", members);

        String viewPath = "/WEB-INF/views/members.jsp";
        RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);

        dispatcher.forward(request, response);
    }
}

```

{% endtab %} {% tab title="members.jsp" %}

```html
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>
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
    <c:forEach var="item" items="${members}">
    <tr>
        <td>${item.id}</td>
        <td>${item.username}</td>
        <td>${item.age}</td>
    </tr>
    </c:forEach>
    </tbody>
</table>
</body>
</html>
```

{% endtab %} {% endtabs %}