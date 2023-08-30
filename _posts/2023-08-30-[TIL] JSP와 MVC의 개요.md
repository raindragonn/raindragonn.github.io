---
layout: "post"
title: "JSP와 MVC의 개요"
subtitle: "JSP와 MVC의 개요"
date: 2023-08-30
author: "raindragonn"
lang: ko
categories: [Web]
tags:
  - MVC
  - JSP
---

## 템플릿 엔진

- 서블릿 덕분에 동적으로 원하는 HTML을 마음껏 만들 수 있다.
    - 정적인 HTML 문서라면 화면이 계속 달라지는 회원의 저장 결과라던가, 회원 목록같은 동적인 HTML을 만드는 일은 불가능 할 것이다.
- 템플릿 엔진이 나온 이유
    - 자바 코드로 HTML을 만들어 내는 것 보다 차라리 HTML 문서에 동적으로 변경해야 하는 부분만 자바 코드를 넣을 수 있다면 더 편리할 것이다.
    - 템플릿 엔진을 사용하면 HTML 문서에서 필요한 곳만 코드를 적용해서 동적으로 변경할 수 있다.
- 템플릿 엔진의 종류
    - JSP, Thymeleaf, Freemarker, Velocity등이 있다.

### JSP

- 성능과 기능면에서 다른 템플릿 엔진과의 경쟁에서 밀리면서, 점점 사장되어 가는 추세이다.

**[ 라이브러리 추가 ]** 

```groovy
//JSP 추가 시작
implementation 'org.apache.tomcat.embed:tomcat-embed-jasper'
implementation 'javax.servlet:jstl'

//JSP 추가 - spring 3.0~
implementation 'org.apache.tomcat.embed:tomcat-embed-jasper'
implementation 'jakarta.servlet:jakarta.servlet-api'
implementation 'jakarta.servlet.jsp.jstl:jakarta.servlet.jsp.jstl-api'
implementation 'org.glassfish.web:jakarta.servlet.jsp.jstl'
```

**[ 활용법 ]**

- JSP는 자바 코드를 그대로 다 사용할 수 있다.
- 첫 줄은 JSP문서라는 뜻이다. JSP 문서는 이렇게 시작해야 한다.
    - `<%@ page contentType="text/html;charset=UTF-8" language="java" %>`
- 자바의 import 문과 같다.
    - `<%@ page import="hello.servlet.domain.member.MemberRepository" %>`
- 이 부분에는 자바 코드를 입력할 수 있다.
    - `<%= ~~ %>` `<% ~~ %>`

**[ 서블릿과 JSP의 한계 ]**

- 서블릿으로 개발하는 경우 View(화면)를 위한 html을 만드는 작업이 자바 코드에 섞여 지저분하고 복잡하다.
- JSP를 사용하는 경우 HTML 작업을 깔끔하게, 동적으로 필요한 부분에는 자바 코드를 적용할 수 있다.
    - 비즈니스 로직 절반, VIew 절반으로 너무 많은 코드가 JSP에 노출되어있어 너무 많은 역할을한다.

## MVC 패턴의 개요

- **너무 많은 역할**
    - 하나의 서블릿이나 JSP만으로 비즈니스 로직과 뷰 렌더링을 모두 처리하는 것은 너무 많은 역할을 하게되어 결과적으로 유지보수가 어려워진다.
- **변경의 라이프 사이클**
    - UI의 일부를 수정하는 일과 비즈니스 로직을 수정하는 일은 **각각 다르게 발생할 가능성이 매우 높고 대부분 서로에게 영향을 주지 않는다.**
    - 이 때문에, 변경의 라이프 사이클이 다른 경우 하나의 코드로 관리하는 것은 유지보수에 좋지 못하다.
- **********************기능 특화**********************
    - JSP와 같은 뷰 템플릿은 화면을 렌더링하는 것에 최적화 되어 있기 때문에 해당 부분만 담당하는 것이 가장 효율적이다.

### Model, View, Controller

- MVC패턴은 하나의 서블릿이나, JSP로 처리하던 것을 컨트롤러와 View라는 영역으로 서로 역할을 나눈 것을 말한다.

**[ Controller ]**

- HTTP 요청을 받아서 파라미터를 검증하고, 비즈니스 로직을 실행한다.
- 뷰에 전달할 결과를 데이터를 조회해서 모델에 담는다.
- **참고**
    - 컨트롤러에 비즈니스 로직을 둘 수도 있지만, 너무 많은 역할을 담당한다. 그래서 일반적으로 비즈니스 로직은 서비스(Service)라는 계층을 별도로 만들어서 처리한다.
    - 컨트롤러는 비즈니스 로직이 있는 서비스를 호출하는 역할을 담당한다.

**[ Model ]**

- 뷰에 보여줄 데이터를 담아둔다.
- 뷰가 필요한 데이터를 모두 모델에 담아서 전달해주는 덕분에 뷰는 비즈니스 로직이나 데이터 접근을 몰라도 되며, 화면 렌더링에 집중할 수 있다.

**[ View ]** 

- 모델에 담겨있는 데이터를 사용해서 화면을 그리는 일에 집중한다.

## MVC 패턴 적용

- 서블릿은 컨트롤러로, JSP는 뷰로 사용해서 MVC패턴을 적용.
- Model은 HttpServletRequest객체를 사용한다.
    - `setAttribute(), getAttribute()` 를 통해 데이터를 보관하고, 조회할 수 있다.

**[ /WEB-INF ]**

- 해당 경로안에 JSP가 있으면 외부에서 직접 JSP를 호출할 수 없다.
    - ~/WEB-INF/AA.JSP 등
- 항상 컨트롤러를 통해서 JSP를 호출하는 것을 기대하기에 사용.

**[ Controller ]**

```java
@WebServlet(name = "mvcMemberFormServlet", urlPatterns = "/servlet-mvc/members/new-form")
public class MvcMemberFormServlet extends HttpServlet {

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        String viewPath = "/WEB-INF/views/new-form.jsp";

        // 다른 서브릿이나 JSP로 이동할 수 있는 기능이다. 서버 내부에서 다시 호출이 발생한다.
        // Redirect가 아님 즉, URL이 변경되는 일이 없어 사용자가 인지할 수 없다.
        RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
        dispatcher.forward(request, response);
    }
}
```

```java
@WebServlet(name = "mvcMemberSaveServlet", urlPatterns = "/servlet-mvc/members/save")
public class MvcMemberSaveServlet extends HttpServlet {
    private final MemberRepository memberRepository = MemberRepository.getInstance();

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

        String username = request.getParameter("username");
        int age = Integer.parseInt(request.getParameter("age"));

        Member member = new Member(username, age);
        memberRepository.save(member);

        // Model에 데이터를 보관
        request.setAttribute("member", member);

        String viewPath = "/WEB-INF/views/save-result.jsp";
        RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
        dispatcher.forward(request, response);
    }
}
```

- `dispatcher.forward()` : 다른 서블릿이나 JSP로 이동할 수 있는 기능이다. 서버 내부에서 다시 호출이 발생한다.
    
    ```java
    // 다른 서브릿이나 JSP로 이동할 수 있는 기능이다. 서버 내부에서 다시 호출이 발생한다.
    // Redirect가 아님 즉, URL이 변경되는 일이 없어 사용자가 인지할 수 없다.
    RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
    dispatcher.forward(request, response);
    ```
    

> redirect vs forward : 리다이렉트는 실제 클라이언트에 응답이 나갔다가 클라이언트가 리다이렉트 경로로 다시 요청을 하지만 포워드는 서버 내부에서 일어나는 호출이기 때문에 클라이언트가 인식할 수 없다.
> 

**[ View ]**

```html
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
	<body>
    성공
     <ul>
        <li>id=${member.id}</li>
        <li>username=${member.username}</li>
        <li>age=${member.age}</li>
     </ul>
     <a href="/index.html">메인</a>
	</body>
</html>
```

- <%= request.getAttribute("member")%> 로 모델에 저장한 member 객체를 꺼낼 수 있지만,너무 복잡해진다.
- **JSP는 ${} 문법을 제공**하는데, 이 문법을 사용하면 request의 attribute에 담긴 데이터를 편리하게 조회할 수 있다.

## MVC 패턴 한계

- ******************포워드 중복******************
    - VIew 로 이동하는 코드가 항상 중복 호출되어야한다.
- **ViewPath에 중복**
    - prefix : /WEB-INF
    - suffix: .jsp
- **사용하지 않는 코드**
    - response는 현재 코드에서 사용되지않는다.
    - 이런 `HttpServletRequest` , `HttpServletResponse` 를 사용하는 코드는 테스트 케이스를 작성하기도 어렵다.
- **공통 처리의 어려움**
    - 기능이 복잡해질 수 록 컨트롤러에서 공통적으로 처리해야하는 부분이 증가할 것이다.
    - 컨트롤러 호출 전에 먼저 공통 기능을 처리해야 한다.
    - 프론트 컨트롤러(Front Controller)패턴을 도입하면 문제를 해결할 수 있다.