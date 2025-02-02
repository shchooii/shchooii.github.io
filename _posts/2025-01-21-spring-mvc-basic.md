---
title: Spring MVC Basic
date: 2025-01-21 14:25:15 +09:00
categories: ['spring']
tags: ['SpringMVC', '@RequestMapping', '@RequestBody', '@ResponseBody']
---

# Spring MVC Basic

이번 글에서는 Spring MVC의 컨트롤러 기본 기능을 정리해보겠습니다.
기본 기능으로는 **요청을 컨트롤러에 매핑하는 기능**과 **데이터를 주고받는 기능**이 있습니다.
> 아를 위해 @RequestMapping을 비롯한 다양한 매핑 어노테이션과 요청 데이터를 처리하는 방법을 중심으로 살펴보겠습니다.

## @RequestMapping

@RequestMapping 에 method 속성으로 HTTP 메서드를 지정하지 않으면 HTTP 메서드와 무관하게 호출됩니다.
HTTP 메소드 종류는 다음과 같습니다.
1. `GET`: 리소스 조회
1. `POST`: 데이터 추가
1. `PUT`: 리소스 수정
1. `DELETE`: 리소스 삭제
1. `PATCH'`: 리소스 부분 변경

`PUT`과 `PATCH'`의 차이점은 리소스 데이터가 A, B로 이루어져있다고 가정했을 때 `PUT`의 경우는 A, B가 C, D로 전부 대체되고,
`PATCH`는 일부 B만 C로 변경되는 경우를 말합니다.

```java
@RequestMapping(value = "/hello", method = RequestMethod.GET)
public String hello() {
    return "Hello, Spring!";
}
```

이 endpoint에 POST 요청을 하면 스프링 MVC는 HTTP 405 상태코드(Method Not Allowed)를 반환합니다.

```java
@GetMapping("/hello")
public String hello() {
    return "Hello, Spring!";
}
```

`@RequestMapping(value = "/hello", method = RequestMethod.GET)`와 `@GetMapping("/hello")`는 같은 코드입니다.
`@PostMapping`, `@PutMapping`, `@DeleteMapping`, `@PatchMapping`도 사용 가능합니다.

## @RequestBody
Spring Servlet 포스팅에서 요청 데이터 형식 3가지를 정의했습니다.

> 1. GET
>    1. `www.hello.com/hi?username=jo&age=20`
>    1. ? 이후 쿼리파라미터에 데이터를 포함해서 전달합니다. 
> 1. Post - HTML Form 
>    1. content-type: application/x-www-form-urlencoded 
>    1. HTTP Message Body에 쿼리 파리미터 형식으로 전달 username=jo&age=20 
> 1. HTTP Message Body
>    1. ex) JSON, XML, TEXT


1번, 2번의 경우는 다음 3가지 annotation을 사용합니다.
- @RequestParam : 쿼리 파라미터나 폼 데이터를 받음.
- @PathVariable : URL 경로에서 값을 추출.
- @ModelAttribute : 데이터를 객체로 바인딩.


```java
// @RequestParam: ?name=Spring
@GetMapping("/greet")
public String greet(@RequestParam String name) {
    return "Hello, " + name;
}

// @PathVariable: /user/123
@GetMapping("/user/{id}")
public String getUser(@PathVariable Long id) {
    return "User ID: " + id;
}

// @ModelAttribute
@PostMapping("/register")
public String registerUser(@ModelAttribute User user) {
    return "User Registered: " + user.getName();
}
```

3번의 경우, HTTP 요청 본문(body)을 직접 읽고 쓸 때 @HttpEntity, @RequestBody를 사용합니다.

1. `InputStream / Reader` : 원시 데이터를 읽는 가장 기본적인 방식 
2. `HttpEntity<T>` : HTTP 요청 본문을 객체로 변환 (헤더 + 바디 포함)
3. `@RequestBody` : 요청 본문을 객체로 직접 매핑

대부분 3번 `@RequestBody`을 사용하고, 필요에 따라 2번도 사용합니다.

```java
// HttpEntity 사용: HTTP 헤더도 함께 다룰 수 있음
@PostMapping("/create-entity")
public ResponseEntity<String> createUser(HttpEntity<User> requestEntity) {
    User user = requestEntity.getBody();
    return ResponseEntity.ok("Created User: " + user.getName());
}

// @RequestBody 사용: JSON 요청을 Java 객체로 변환
@PostMapping("/create")
public String createUser(@RequestBody User user) {
    return "Created User: " + user.getName();
}
```

## @ResponseBody

다음은 `Response`를 반환하는 3가지 방법입니다.

1. `OutputStream / Writer` : 기본적인 출력 스트림. 
2. `HttpEntity<T> / ResponseEntity<T>` : 응답 헤더와 바디를 다룰 수 있음.
3. `@ResponseBody` : 컨트롤러 메서드의 반환값을 HTTP 응답 본문으로 직접 반환.

대부분 3번 `@ResponseBody`를 사용하고 필요에 따라 2번을 사용합니다.

```java 
// ResponseEntity 사용: 응답 상태 코드 + 데이터 반환
@GetMapping("/json")
public ResponseEntity<User> getJson() {
    User user = new User("SpringUser", 25);
    return ResponseEntity.ok(user);
}

// @ResponseBody
@GetMapping("/text")
@ResponseBody
public String getText() {
    return "Hello, Spring!";
}

```

## Controller Return

컨트롤러에서 데이터를 반환하는 방법에는 `ModelAndView`와 `String`이 있습니다.
- `ModelAndView` : 뷰 이름과 모델 데이터를 함께 전달.
- `String` : 뷰 이름을 반환. (기본적으로 ViewResolver가 처리.)
- `@ResponseBody`가 붙으면 `String`을 응답 본문으로 직접 반환.

```java
// View를 반환하는 경우 (Thymeleaf, JSP 등 사용)
@GetMapping("/view")
public ModelAndView getView() {
    ModelAndView mav = new ModelAndView("hello");
    mav.addObject("message", "Hello, Spring!");
    return mav;
}

// @ResponseBody가 없으면 ViewResolver가 실행됨
@GetMapping("/hello")
public String helloPage() {
    return "hello"; // templates/hello.html (Thymeleaf 기준)
}

// @ResponseBody가 있으면 문자열를 반환 (REST API 응답처럼 동작)
@GetMapping("/text-response")
@ResponseBody
public String textResponse() {
    return "Hello, Spring!";
}

```

> Spring MVC의 기본적인 컨트롤러 기능을 정리했습니다.

> Spring MVC에서는 요청을 적절한 컨트롤러 메서드로 매핑하고, 다양한 방식으로 데이터를 주고받을 수 있습니다.
> `@RequestMapping`을 비롯한 HTTP 메서드 매핑 어노테이션과 `@RequestParam`, `@PathVariable`, `@RequestBody` 등의 어노테이션을 활용하면 효율적으로 요청 데이터를 처리할 수 있습니다.
> `@ResponseBody`를 사용하면 JSON이나 텍스트 데이터를 직접 응답으로 반환할 수 있으며, `ResponseEntity`를 활용하면 HTTP 상태 코드와 함께 보다 정교한 응답을 구성할 수도 있습니다. 

> 이러한 기본 기능 정리글이 Spring MVC 프로젝트에 도움이 되길 바랍니다. 😊
