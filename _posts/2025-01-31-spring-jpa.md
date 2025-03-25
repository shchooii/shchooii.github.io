---
title: Spring JPA
date: 2025-01-31 18:33:05 +09:00
categories: ['spring']
tags: ['SpringJPA', '@Entity', '@XToOne', '@OneToMany']
---


JPA (Java Persistent API)는 자바 객체와 관계형 데이터베이스 간의 매핑을 도와줍니다. 
이를 통해 SQL 문을 작성하지 않고 객체를 다루는 것처럼 데이터를 관리할 수 있습니다.
API 설계 과정에서 엔티티를 직접 사용하는 것은 지양해야 합니다. 
엔티티를 API 요청/응답에 그대로 활용하면 다음과 같은 문제가 발생합니다.

1. API 구조 유지: 엔티티가 변경되면 API 구조가 변경되어 코드를 변경해야 합니다.
2. 캡슐화: 엔티티는 내부 비즈니스 로직을 포함하므로, 캡슐화도어 있어야 합니다.
3. 유효성 검증: 엔티티 내부에 API 검증 로직이 추가되면 도메인 로직과 프레젠테이션 로직이 뒤섞입니다.

## DTO 사용

DTO (Data Transfer Object)를 별도로 설계하여 위 문제를 해결할 수 있습니다. 
DTO를 통해 API 로직와 엔티티 로직을 분리하여 독립적으로 관리할 수 있습니다.
아래 예시를 살펴보겠습니다.

### Entity를 RequestBody에 매핑하는 방법

```java
@RestController
@RequiredArgsConstructor
public class MemberApiController {
    private final MemberService memberService;

    @PostMapping("/api/v1/members")
    public CreateMemberResponse saveMemberV1(@RequestBody @Valid Member member) {
        Long id = memberService.join(member);
        return new CreateMemberResponse(id);
    }

    @Data
    static class CreateMemberRequest {
        private String name;
    }

    @Data
    static class CreateMemberResponse {
        private Long id;
        public CreateMemberResponse(Long id) {
            this.id = id;
        }
    }
}
```

다음과 같이 사용하려면 엔티티 코드에 API 검증 로직이 추가되어야 합니다. 
그리고 엔티티가 변경되면 API 구조도 같이 변경됩니다.
API 요구사항은 끊임없이 변합니다. 변할 때마다 매번 엔티티 코드, API 구조를 수정해야 합니다.
> 이러한 이유로 DTO를 활용해 API를 설계해야 합니다.

### DTO API 설계
 
```java
@PostMapping("/api/v2/members")
public CreateMemberResponse saveMemberV2(@RequestBody @Valid CreateMemberRequest request) {
    Member member = new Member();
    member.setName(request.getName());
    Long id = memberService.join(member);
    return new CreateMemberResponse(id);
}
```

이렇게 작성하면 엔티티 로직과 API 구조를 독립적으로 관리할 수 있습니다.
이러한 방법은 데이터를 저장하고, 삭제하고, 수정하는 데에 많이 사용됩니다.
데이터를 저장하고, 삭제하고, 수정하는 것은 대체로 Query문 1번이면 수행되기 때문입니다.
> 예를 들어, @OneToMany 관계가 있는 데이터를 조회해야 하는 경우는 어떨까요? 위 방법 그대로 사용해도 될까요?

## JPA Optimization

DTO를 통해 엔티티와 API 구조를 분리하는 것은 중요한 설계 방식입니다. 
하지만 데이터 조회 시에는 단순히 DTO를 활용하는 것만으로 충분하지 않을 수 있습니다. 
JPA를 사용할 때는 지연 로딩(Lazy Loading)을 기본값으로 설정하고, 필요할 때만 Fetch Join을 활용하는 것이 좋습니다.

### No Eager Loading

`Eager Loading`을 사용하면 필요하지 않은 연관 엔티티까지 즉시 로딩되어 성능 저하를 초래합니다.
대신에 `Lazy Loading`을 사용하면 필요한 시점에만 데이터를 로딩할 수 있어 성능 최적화가 가능합니다.
`Lazy Loading`을 사용할 경우 N+1 문제가 발생할 수 있으며, 이를 방지하기 위해 `Fetch Join`을 적절히 활용해야 합니다.

### Lazy Loading & Fetch Join

- `em.createQuery()`를 통해 `Fetch Join` 활용해야 함.
- `@XToOne`: @OneToOne, ManyToOne
  - ToOne의 경우, row 수가 증가되지 않음. 한 번의 쿼리로 조회하는 것이 효율적임.
- `@OneToMany`
  - `N + 1 문제`: 지연 로딩을 사용하면 연관된 데이터를 가져올 때 개별적인 쿼리를 하나씩 실행함.
  - `@BatchSize`를 설정하면, 여러 개의 데이터를 한 번의 IN 쿼리로 가져올 수 있음.

### @XToOne

`findOrders()`
주문과 관련된 1:1, N:1 관계의 데이터를 조회합니다.
- Order 엔티티를 기준으로 member(N:1)와 delivery(1:1) 정보를 join.
- 단일 쿼리로 필요한 데이터를 모두 조회할 수 있음.
- DTO로 필요한 데이터만 선택적으로 가져옴.

```java
// 1:N 관계를 제외한 나머지를 한번에 조회
private List<OrderQueryDto> findOrders() {
        return em.createQuery(
                        "select new jpabook.jpashop.repository.order.query.OrderQueryDto(o.id, m.name, o.orderDate, o.status, d.address)" +
                        " from Order o" + 
                        " join o.member m" +
                        " join o.delivery d", OrderQueryDto.class)
                .getResultList();
    }
```

`findOrderItems()`
1:N 관계인 주문상품 정보를 별도로 조회합니다.
- 특정 주문(orderId)에 속한 주문상품(OrderItem)과 상품(Item) 정보를 조회.
- IN 쿼리를 통해 N+1 문제를 최소화할 수 있음.
- DTO를 통해 필요한 데이터만 선택적으로 조회함.

```java
// 1:N 관계인 orderItems 조회
private List<OrderItemQueryDto> findOrderItems(Long orderId) {
        return em.createQuery(
                        "select new jpabook.jpashop.repository.order.query.OrderItemQueryDto(oi.order.id, i.name, oi.orderPrice, oi.count)" +
                        " from OrderItem oi" +
                        " join oi.item i" +
                        " where oi.order.id = : orderId", OrderItemQueryDto.class)
                .setParameter("orderId", orderId)
                .getResultList();
    }
```

## 최적화 전략 정리

> JPA를 사용하면서 성능 최적화를 위해 알아두면 좋은 몇 가지 방법들을 살펴보았습니다. 

>>API를 만들 때는 엔티티 대신 `DTO`를 사용하는 것이 좋습니다. 이렇게 하면 API 스펙과 엔티티를 따로 관리할 수 있어서, 나중에 변경사항이 생겨도 유연하게 대응할 수 있습니다. 
필요한 데이터만 골라서 가져올 수 있어서 효율적입니다. 두 번째는 `지연 로딩(Lazy Loading)`을 활용하는 것입니다. 즉시 로딩을 사용하면 어떤 SQL이 실행될지 예측하기가 어렵습니다. 
차라리 지연 로딩을 기본으로 두고, 필요할 때만 페치 조인으로 최적화하는 것이 더 안전한 방법입니다.
마지막으로, 페치 조인도 전략적으로 사용해야 합니다. `ToOne` 관계는 한 번에 조회해도 데이터가 늘어나지 않아서 페치 조인을 활용하기 좋습니다. 
반면에 컬렉션은 지연 로딩을 유지하면서 필요한 경우에만 `@BatchSize`를 사용하는 것이 효과적입니다.

> Spring JPA를 프로젝트에 적용해보는 것은 어떨까요? 이 글이 여러분의 Spring JPA 프로젝트에 작은 도움이 되었으면 좋겠습니다.
