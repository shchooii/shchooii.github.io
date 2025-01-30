---
title: Spring Start
date: 2024-12-29 14:25:03 +09:00
categories: ['spring']
tags: ['SpringBootApplication', 'Bean']
---

## Spring Start


Spring이란 Java 기반 오픈소스 웹 Framework입니다. Framework는 Library와 달리 IoC(Inversion of Control)를 통해 코드 변경을 최소화해주고, 다양한 상황에 맞도록 코드 구성을 도와줍니다. Spring을 통해 개발자는 비지니스 로직에 집중하여 효율적으로 개발할 수 있습니다.


Spring Framework는 Configuration 설정, Dependency를 하나하나 직접 관리해야 하는 어려움이 있습니다. Spring Boot는 이를 자동으로 관리해주고, Convention을 통해 코드의 가독성과 일관성에 도움을 줍니다.


다음은 Spring Boot에서 Application을 실행하는 부분입니다.

```java
@SpringBootApplication
public class StartSpringApplication {

    public static void main(String[] args) {
        SpringApplication.run(StartSpringApplication.class, args);
    }
}
```


Framework는 코드 변경을 최소화해주지만, 제공되는 흐름에 맞게 코드를 구성해야 합니다. Java 클래스의 main 메서드 안에 `SpringApplication.run()`를 통해 Application이 실행됩니다. 여기서 생략된 과정이 있는데, Annotation 부분입니다. @SpringBootApplication Annotation에서 Configuration을 설정하고, ComponentScan을 수행합니다.


```java
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan
public @interface SpringBootApplication {}
```


1. @SpringBootConfiguration
   1. Spring Boot Application이 실행되기 위한 기본 설정를 수행합니다.
   2. @Bean 메서드들을 정의하고, Spring Container가 이를 읽어 애플리케이션 Bean으로 등록합니다.
2. @EnableAutoConfiguration 
   1. 클래스에 명시되어 있는  Configuration 설정과 Dependency를 기반으로 Application에 필요한 환경을 자동으로 설정합니다.
3. @ComponentScan
   1. ComponentScan을 수행하고 Bean을 등록합니다.

"Bean을 등록한다"는 Spring Container에 Bean을 등록하고, Bean으로 등록된 객체는 Spring Container의 관리 대상이 됩니다. Container에서는 객체를 생성하고 생명 주기 관리, DI (Depenendcy Injection), AOP, 등의 기능을 제공합니다. 


@Bean은 method level에서 사용되며, @Configuration annotation에 있는 클래스에 있어야 합니다. Spring이 제공하는 자동 스캔 기능을 사용하지 않고, 개발자가 직접 Bean을 정의하는 방법입니다.


@Component는 class level에서 사용되며, Spring이 자동으로 Bean을 등록합니다. @Service, @Repository, @Controller도 내부에 @Component를 포함하고 있어, 자동으로 Bean이 등록됩니다. 


> 지금까지 @SpringBootApplication 코드가 어떻게 구성되어 있는지 살펴보았습니다. 
> >Spring Boot는 개발자가 일일이 설정을 하지 않아도 자동으로 Configuration을 적용하고, @ComponentScan을 통해 필요한 Bean을 등록하며, @EnableAutoConfiguration을 통해 적절한 환경을 설정해 줍니다. 이를 통해 개발자는 Boilerplate Code를 줄이고, 비즈니스 로직에 더욱 집중할 수 있습니다.

