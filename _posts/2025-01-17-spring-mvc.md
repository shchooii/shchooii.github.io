---
title: Spring MVC
date: 2025-01-17 17:15:07 +09:00
categories: ['spring']
tags: ['SpringMVC', 'FrontController', 'DispatcherServlet']
---

# Spring MVC
## FrontController pattern
### Background
웹 애플리케이션에서 사용자의 요청을 처리하는 기본적인 방식은 서블릿(Servlet)을 활용하는 것입니다.
> 만약 우리가 사용자 요청을 처리하기 위해 각 기능별로 서블릿을 만든다면 어떻게 될까요?

다음과 같은 3가지 요청을 처리하는 애플리케이션을 만들려고 합니다.
1. `/users` → 사용자 정보를 조회하는 기능
1. `/products` → 상품 정보를 조회하는 기능
1. `/orders` → 주문 정보를 처리하는 기능

```java
@WebServlet("/users")
public class UserServlet extends HttpServlet {
    protected void doGet(HttpServletRequest request, HttpServletResponse response) {
        // 사용자 조회 로직
    }
}
```
```java
@WebServlet("/products")
public class ProductServlet extends HttpServlet {
    protected void doGet(HttpServletRequest request, HttpServletResponse response) {
        // 상품 조회 로직
    }
}
```
```java
@WebServlet("/orders")
public class OrderServlet extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) {
        // 주문 처리 로직
    }
}
```
위 방식은 아래 3가지 문제점이 있습니다.
1. 중복 코드 증가
   1. 모든 서블릿에서 `request`를 파싱하고, 공통적인 로깅, 보안 검증 등의 로직을 중복해서 작성해야 함.
1. 유지보수의 어려움
   1. 요청마다 개별 서블릿을 만들어야 하므로 확장성이 떨어짐.
   1. 공통적인 기능(예: 인증, 로깅)을 추가해야 하면 모든 서블릿을 수정해야 함.
1. 컨트롤러(Handler) 관리가 어려움
   1. 요청 URL을 직접 서블릿으로 매핑해야 하므로 유연성이 부족함
   1. URL 변경 시 서블릿 수정이 필요함.

### FrontController 패턴의 도입

이러한 문제를 해결하기 위해 Front Controller 패턴이 등장했습니다.
Front Controller 패턴을 적용하면 하나의 서블릿(Front Controller)이 모든 요청을 받아 처리한 후, 
적절한 컨트롤러(Handler)로 요청을 전달할 수 있습니다.

Spring MVC에서는 `DispatcherServlet`이 이 역할을 수행합니다.

## Spring MVC DispatcherServlet

### 1️⃣ DispatcherServlet이 모든 요청을 받아 처리함
Spring Boot에서는 `DispatcherServlet`이 자동으로 설정돼 있습니다. 
아래 코드는 수동 설정 코드입니다.

```java
@Bean
public DispatcherServlet dispatcherServlet() {
    return new DispatcherServlet();
}
```

### 2️⃣ Controller 등록 (Handler)
컨트롤러가 `Request`을 처리하는 부분입니다.

```java
@RestController
public class HelloController {

    @GetMapping("/hello")
    public String hello() {
        return "Hello, Spring MVC!";
    }
}
```

### 3️⃣ HandlerMapping이 컨트롤러를 찾음
Spring에서는 여러 가지 `HandlerMapping``을 제공합니다. 아래 2가지가 자주 쓰입니다.
1. `RequestMappingHandlerMapping` → `@RequestMapping` 기반 컨트롤러 찾음.
2. `SimpleUrlHandlerMapping` → 수동으로 설정한 URL을 찾음.

Spring Boot에서는 자동으로 `RequestMappingHandlerMapping`을 사용해서 컨트롤러를 찾습니다.

```java
@Bean
public RequestMappingHandlerMapping requestMappingHandlerMapping() {
    return new RequestMappingHandlerMapping();
}
```

### 4️⃣ HandlerAdapter가 컨트롤러 실행
HandlerAdapter가 컨트롤러를 실행합니다.

```java
@Bean
public RequestMappingHandlerAdapter requestMappingHandlerAdapter() {
    return new RequestMappingHandlerAdapter();
}
```

### 5️⃣ Controller -> Json Response or ModelAndView
JSON 응답을 반환하는 경우 `@RestController`를 사용하고, 뷰를 반환하려면 `ModelAndView`를 사용합니다.
```java
@Controller
public class HomeController {

    @GetMapping("/home")
    public ModelAndView home() {
        ModelAndView mav = new ModelAndView("home"); // home.html로 이동
        mav.addObject("message", "Welcome to Spring MVC!");
        return mav;
    }
}
```

### 6️⃣ ViewResolver가 View를 찾아서 렌더링
`return "home";`을 반환하면, Spring Boot의 `ViewResolver`가 적절한 뷰 파일을 찾아줍니다. `Thymeleaf`로 예를 들어 보겠습니다.

```properties
spring.thymeleaf.prefix=classpath:/templates/
spring.thymeleaf.suffix=.html
```

**home.html (Thymeleaf)**
```html
<!DOCTYPE html>
<html>
<head>
    <title>Home</title>
</head>
<body>
    <h1 th:text="${message}"></h1>
</body>
</html>
```

ViewResolver가 home.html을 찾아서 렌더링합니다.

### Front Controller 패턴 적용 시 개선점

| 구분 | 기존 방식 (각 서블릿 개별 처리) | Front Controller 패턴 적용 |
|------|--------------------------------|----------------------------|
| **공통 기능 처리** | 각 서블릿에서 중복 작성 | DispatcherServlet에서 공통 처리 |
| **확장성** | 새로운 기능 추가 시 서블릿 추가 필요 | 새로운 컨트롤러만 만들면 됨 |
| **유지보수성** | 서블릿 개수 증가로 복잡해짐 | 하나의 진입점(DispatcherServlet)으로 관리 용이 |
| **유연성** | URL 변경 시 서블릿 코드 수정 필요 | HandlerMapping을 통해 유연한 URL 관리 가능 |


Spring MVC에서 Front Controller 패턴을 적용하면, `Request`를 한 곳에서 관리하면서 중복 코드와 유지보수 문제를 해결할 수 있습니다.

> `DispatcherServlet`에서 모든 Request를 처리합니다.
> `HandlerMapping`을 통해 요청을 적절한 컨트롤러로 연결하고, 
> `HandlerAdapter`가 컨트롤러를 실행한 후 
> `ViewResolver`가 뷰를 찾아 렌더링합니다.

>이를 통해 확장성과 유지보수성을 높이고, 공통 로직을 효율적으로 관리할 수 있습니다.
>Spring MVC를 활용해, 보다 유연하고 구조적인 웹 애플리케이션을 개발할 수 있습니다. 😊








