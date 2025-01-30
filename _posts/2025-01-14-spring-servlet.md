---
title: Spring Servlet
date: 2025-01-14 16:22:05 +09:00
categories: ['spring']
tags: ['HttpServlet', 'DispatcherServlet']
---

# Servlet
Servlet은 Java 기반의 웹 애플리케이션에서 클라이언트의 요청을 처리하고 응답을 생성하는 Server-side components입니다.
다음은 chatgpt 웹사이트 request 요청입니다.

```text
POST /v1/chat/completions HTTP/1.1
Host: https://chatgpt.com
Authorization: Bearer YOUR_API_KEY
Content-Type: text/plain;charset=UTF-8
accept: */*
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko)
content-length: 3362
```

HTTP 요청은 클라이언트가 서버에 데이터를 보내는 방식으로, 주요 속성은 다음과 같습니다.

1. Request Line : `POST /v1/chat/completions HTTP/1.1` → 요청 메서드(POST), URL 경로, HTTP 버전 포함.
2. Headers:
  - `Host`: 요청 대상 서버 (`chatgpt.com`)
  - `Authorization`: 인증 정보 (Bearer 토큰)
  - `Content-Type`: 데이터 유형 (`text/plain; charset=UTF-8`)
  - `accept`: 응답 형식 ex) \*/*, application/json
  - `User-Agent`: 요청을 보내는 클라이언트 정보
  - `content-length`: 요청 본문의 길이

웹 애플리케이션에서 HTTP 요청을 직접 처리하려면 요청을 일일이 파싱하고 응답을 생성해야 합니다. Servlet은 이러한 작업을 자동화하여 개발자가 핵심 로직에 집중할 수 있도록 도와줍니다. Servlet은 HTTP 요청을 받아 적절한 비즈니스 로직을 수행한 후, HTML, JSON 등의 형태로 응답을 반환합니다. 웹 서버(Tomcat, Jetty 등) 내에서 실행되며, HttpServlet 클래스를 상속하여 구현됩니다.

## HttpServlet

다음은 Servlet Container 동작 방식입니다.

1. Client Request: 사용자가 웹 브라우저에서 URL을 입력하거나, API 호출을 보냅니다. 
1. Servlet Container
   1. HttpServletRequest 객체와 HttpServletResponse 객체를 생성합니다.
   1. HttpServletRequest와 HttpServletResponse 객체를 적절한 Servlet 객체로 전달합니다.
      1. ex) "/hello"로 request가 오면 해당하는 Servlet 객체에 전달합니다.
1. service() 메서드 호출됩니다.
   1. HTTP 메서드 따라 적절한 doGet(), doPost() 메소드 실행됩니다.
1. HttpServletResponse를 통해 Client에게 응답이 반환됩니다.

아래는 doPost()의 코드입니다. HttpServletReqeust, HttpServletResponse를 받습니다. doGet(), doPost(), 등을 오버라이딩해 핵심 비즈니스 로직을 작성하게 실행합니다.

```java
protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        String msg = lStrings.getString("http.method_post_not_supported");
        this.sendMethodNotAllowed(req, resp, msg);
    }
```

### HttpServletRequest
HTTP 요청 메시지를 개발자가 직접 파싱해서 사용해도 되지만 권장되지 않습니다. 서블릿은 개발자가 HTTP 요청 메시지를 편리하게 사용할 수 있도록 개발자 대신에 HTTP 요청 메시지를 파싱해줍니다. 아래는 Http Reqeust를 보내는 3가지 방법입니다.

1. GET
   1. `www.hello.com/hi?username=jo&age=20`
   1. ? 이후 쿼리파라미터에 데이터를 포함해서 전달합니다.
1. Post - HTML Form
   1. content-type: application/x-www-form-urlencoded
   1. HTTP Message Body에 쿼리 파리미터 형식으로 전달 username=jo&age=20
1. HTTP Message Body
   1. ex) JSON, XML, TEXT

Client 입장에서는 두 방식에 차이가 있지만, Server 입장에서는 둘의 형식이 동일하므로, 1번과 2번은 `request.getParameter()`로 조회할 수 있습니다.

```java
String username = request.getParameter("username");
String age = request.getParameter("age");
```

3번은 다음과 같습니다.
Response 부분은 HttpServletResponse 부분에서 설명드리겠습니다.

```java
@WebServlet("/json")
public class JsonRequestServlet extends HttpServlet {
    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        BufferedReader reader = request.getReader();
        String jsonBody = reader.lines().collect(Collectors.joining());
        response.setContentType("application/json");
        response.getWriter().write("{\"received\": " + jsonBody + "}");
    }
}
```

### HttpServletResponse

`doPost()`에서 HTTP 응답을 반환할 때, `response.getOutputStream().write()`를 사용합니다. HTML 응답의 경우, Content-Type을 text/html로 설정한 후, HTML 문자열을 UTF-8로 변환하여 OutputStream을 통해 반환합니다. 이렇게 하면 Client (웹 브라우저)가 응답을 HTML로 해석하고 렌더링할 수 있습니다. 

```java
@WebServlet(name = "responseHtmlServlet", urlPatterns = "/response-html")
public class ResponseHtmlServlet extends HttpServlet {
    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        response.setContentType("text/html");
        response.setCharacterEncoding("utf-8");

        String htmlResponse = "<html><body><div>POST 요청 응답</div></body></html>";
        response.getOutputStream().write(htmlResponse.getBytes("utf-8"));
    }
}
```

JSON 응답의 경우, ObjectMapper를 이용해 Java 객체를 JSON 문자열로 변환한 후, `response.getOutputStream().write()`로 반환합니다.

```java
@WebServlet(name = "responseJsonServlet", urlPatterns = "/response-json")
public class ResponseJsonServlet extends HttpServlet {
    private final ObjectMapper objectMapper = new ObjectMapper();

    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        response.setContentType("application/json");
        response.setCharacterEncoding("utf-8");

        HelloData data = new HelloData();
        data.setUsername("jo");
        data.setAge(20);

        String jsonResponse = objectMapper.writeValueAsString(data);
        response.getOutputStream().write(jsonResponse.getBytes("utf-8"));
    }
}
```


## DispatcherServlet

Spring MVC는 내부적으로 Servlet을 기반으로 동작하며, `DispatcherServlet`이 모든 HTTP 요청을 받아 컨트롤러로 전달하는 역할을 합니다.
`HttpServletRequest`와 `HttpServletResponse` 객체를 활용하여, `HandlerMapping`을 통해 적절한 컨트롤러를 찾습니다.
해당 컨트롤러의 메서드를 실행하고, 이후 `ViewResolver`를 사용하여 적절한 뷰를 렌더링합니다. 또는 `@ResponseBody`가 설정된 경우 JSON 응답을 생성합니다.

| 비교 항목 | Servlet                     | Spring MVC                       |
|-------| --------------------------- | -------------------------------- |
| 요청 처리 방식 | `doGet()`, `doPost()` 직접 구현 | `@Controller`와 `@RequestMapping` 사용 |
| URL 매핑 | `web.xml` 또는 `@WebServlet`  | `@RequestMapping` 사용             |
| 응답 처리 | `HttpServletResponse` 직접 설정 | `@ResponseBody`로 설정              |

다음은 SpringMVC 사용 예시입니다.

```java
@RestController
public class HelloController {
@GetMapping("/hello")
public String hello(@RequestParam String name) {
    return "Hello, " + name;
  }
}
```

