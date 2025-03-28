---
title: Java Polymorphism
date: 2025-03-28 11:32:15 +09:00
categories: ['java']
tags: ['abstract', 'interface', 'polymorphism', 'OCP']
---

## 머리말
소프트웨어 설계에서 **역할**과 **구현**을 분리하는 것은 매우 중요합니다. 
이 둘을 분리하면, 구현체를 손쉽게 교체할 수 있고 기존 코드를 최소한으로만 수정할 수 있기 때문입니다. 
이러한 설계 원칙을 **OCP(Open-Closed Principle)**라고 부릅니다.
> **Open for extension**(확장에는 열려 있음), **Closed for modification**(수정에는 닫혀 있음)

마치 **공연 무대**에서 특정 “역할”을 연기할 **배우**를 자유롭게 교체할 수 있는 것과 같습니다. 
한 번 무대를 만들어 두면, 상황이나 극의 필요에 따라 배우를 교체(새로운 구현 클래스 추가)하기만 하면 되기 때문에, 기존 무대(코드)는 크게 바꿀 필요가 없습니다.

이번 글에서는 역할-구현 분리와 OCP에 도움을 주는 **다형성(Polymorphism)**, **추상 클래스(abstract class)**, 그리고 **인터페이스(interface)**에 대해 살펴보겠습니다.

---

## 다형적 참조(Polymorphic Reference)
### 기본 개념
- 하나의 **부모 타입**으로 여러 **자식 인스턴스**를 참조할 수 있는 기능
- 공연 무대에서 “로미오” 역할을 할 수 있는 배우가 여러 명인 것과 유사합니다. 
- `Actor`(부모 타입)라는 역할을 두고, 실제 배우가 여러 명(`RomeoActor`, `JulietActor` 등 자식 클래스) 있을 수 있습니다.

```java
Actor romeo = new RomeoActor();  // 다형적 참조
```

### 예시
```java
public class StageExample {
    public static void main(String[] args) {
        Actor actor = new RomeoActor();
        actor.act();  // 실제로는 RomeoActor의 act()가 호출됨

        actor = new JulietActor();
        actor.act();  // 이번에는 JulietActor의 act()가 호출됨
    }
}
```

> `actor`라는 하나의 변수를 통해 여러 실제 인스턴스를 참조하며, 각각 다른 연기를 수행하도록 만들 수 있습니다.

## 업캐스팅(Upcasting)과 다운캐스팅(Downcasting)
### 업캐스팅(Upcasting)
- **자식 타입** → **부모 타입**으로의 변환
- 항상 **안전**하기 때문에 명시적 캐스팅을 생략해도 됩니다.
- 예: `RomeoActor` → `Actor`

```java
Actor actor = new RomeoActor();  // 업캐스팅 (생략 가능)
```

### 다운캐스팅(Downcasting)
- **부모 타입** → **자식 타입**으로의 변환
- 실제 인스턴스가 자식 타입에 해당하지 않는다면 **`ClassCastException`**(런타임 오류) 발생
- **위험**하기 때문에 반드시 명시적 캐스팅 필요
- 예: `Actor` → `(RomeoActor)Actor`

```java
Actor actor = new RomeoActor();
RomeoActor romeo = (RomeoActor) actor;  // 다운캐스팅
romeo.playRomeoLines();                // 자식 클래스에서만 존재하는 메서드 호출
```

---

## 컴파일 오류 vs 런타임 오류
- **컴파일 오류**: 프로그램이 **실행되기 전**에 IDE 또는 컴파일러가 잡아내는 오류 (잘못된 변수명, 클래스명, 문법 오류 등)
- **런타임 오류**: 프로그램이 **실행 도중**에 발생하는 오류 (`NullPointerException`, `ClassCastException` 등)
  - 보통 사용자가 프로그램을 사용하는 시점에 발생하기 때문에 치명적

---

## 메서드 오버라이딩(Method Overriding)
- **부모 타입**에서 정의한 메서드를 **자식 타입**에서 **재정의**하는 것
- 실제 실행 시점에는 **오버라이딩된 메서드**가 우선적으로 호출됨

```java
abstract class Actor {
    abstract void act();
}

class RomeoActor extends Actor {
    @Override
    void act() {
        System.out.println("로미오 대사를 연기합니다.");
    }
}

class JulietActor extends Actor {
    @Override
    void act() {
        System.out.println("줄리엣 대사를 연기합니다.");
    }
}
```
- `Actor` 타입을 통해 `act()` 메서드를 호출하더라도, 실제로는 오버라이딩된 `RomeoActor`나 `JulietActor`의 메서드가 실행됩니다.

---

## 추상 클래스(abstract class) & 추상 메서드(abstract method)
### 추상 클래스
- **인스턴스화할 수 없는** 클래스 (new로 생성 불가)
- 상속을 통한 **부모 클래스** 역할만 수행
- ‘배우(Actor)’처럼 추상적인 개념을 정의하기에 적합

```java
public abstract class Actor {
    String name;

    public Actor(String name) {
        this.name = name;
    }
    
    // 추상 메서드
    public abstract void act();
}
```

### 추상 메서드
- **메서드 바디**가 없으며, 자식 클래스가 반드시 **오버라이딩** 해야 함
- 부모 클래스에 **추상 메서드**가 하나라도 있으면, 그 부모는 **추상 클래스**여야 함


## 인터페이스(Interface)
- **순수 추상 클래스**의 또 다른 형태
- 모든 메서드가 추상 메서드로 구성될 수 있고, 다른 클래스와 **다중 구현**이 가능

```java
public interface Performer {
    void perform();
}

public class Singer implements Performer {
    @Override
    public void perform() {
        System.out.println("노래를 부릅니다.");
    }
}

public class Dancer implements Performer {
    @Override
    public void perform() {
        System.out.println("춤을 춥니다.");
    }
}
```

- `Performer` 인터페이스를 구현하는 클래스들은 반드시 `perform()` 메서드를 구현해야 합니다.
- 하나의 클래스가 여러 인터페이스를 동시에 구현할 수도 있습니다.


## OCP 원칙(Open-Closed Principle)과 전략 패턴(Strategy Pattern)
### OCP 원칙
- **확장에는 열려 있고**, **수정에는 닫혀 있다**는 원칙
- 공연 무대가 이미 완성되어 있을 때, 새로운 배우(구현체)를 영입해도 무대를 갈아엎지 않아도 되는 것과 같습니다.

### 전략 패턴(Strategy Pattern)
- 알고리즘(또는 로직)을 **클라이언트 코드(무대 스태프)**의 변경 없이 쉽게 교체할 수 있는 패턴
- `perform()` 같은 추상 메서드를 **전략(Strategy)**으로 보고, 이를 구체적으로 구현(`Singer`, `Dancer`)하는 방식

```java
public interface Performer {
    void perform();
}

public class StageDirector {
    // Performer 타입을 주입받아 사용 (전략 교체 가능)
    private Performer performer;
    
    public StageDirector(Performer performer) {
        this.performer = performer;
    }

    public void setPerformer(Performer performer) {
        this.performer = performer;
    }

    public void startShow() {
        performer.perform();
    }
}
```
- `StageDirector`는 `Performer`라는 **역할**에만 의존하며, 실제 구현이 `Singer`든 `Dancer`든 상관없이 활용 가능
- 새로운 공연자가 필요하다면 `Performer`를 구현하는 새로운 클래스를 만들고 주입만 해주면 되므로 **OCP 원칙** 충족

---

## 예제 코드 (종합)
아래 예제는 앞서 설명한 개념들을 하나로 연결해놓은 간단한 코드입니다.

```java
// 1) 추상 클래스: 배우(Actor)
public abstract class Actor {
    private String name;

    public Actor(String name) {
        this.name = name;
    }

    // 추상 메서드: 구체적인 연기(대사)를 자식이 구현
    public abstract void act();
}

// 2) 구체 클래스: 로미오, 줄리엣
public class RomeoActor extends Actor {
    public RomeoActor(String name) {
        super(name);
    }

    @Override
    public void act() {
        System.out.println("로미오 대사를 연기합니다.");
    }
}

public class JulietActor extends Actor {
    public JulietActor(String name) {
        super(name);
    }

    @Override
    public void act() {
        System.out.println("줄리엣 대사를 연기합니다.");
    }
}

// 3) 인터페이스: 공연 기획(Director) - 전략 패턴처럼 설계
public interface Director {
    void direct(Actor actor);
}

// 4) 구체 감독 클래스: 연출 방법(전략)
public class TragedyDirector implements Director {
    @Override
    public void direct(Actor actor) {
        System.out.println("비극적인 분위기로 연출합니다.");
        actor.act();
    }
}

public class ComedyDirector implements Director {
    @Override
    public void direct(Actor actor) {
        System.out.println("유쾌하고 웃긴 분위기로 연출합니다.");
        actor.act();
    }
}

// 5) 메인: 실제 공연 무대
public class MainStage {
    public static void main(String[] args) {
        Actor romeo = new RomeoActor("김로미오");
        Actor juliet = new JulietActor("이줄리엣");

        // 감독(Director) 전략 교체
        Director tragedyDirector = new TragedyDirector();
        Director comedyDirector = new ComedyDirector();

        // TragedyDirector로 비극 연출
        tragedyDirector.direct(romeo);
        tragedyDirector.direct(juliet);

        System.out.println("---- 분위기 전환 ----");

        // ComedyDirector로 코미디 연출
        comedyDirector.direct(romeo);
        comedyDirector.direct(juliet);
    }
}
```

- `Actor`라는 추상 클래스에 공통 로직을 담고, 연기(`act()`)를 구체 클래스(`RomeoActor`, `JulietActor`)에서 **오버라이딩**
- `Director` 인터페이스가 **전략(Strategy)** 역할을 하며, 구체 구현(비극/코미디)이 자유롭게 교체 가능
- `MainStage`(메인)에서 `Actor`와 `Director` 조합을 바꾸는 것만으로 다양한 연출 실현
- **역할(추상, 인터페이스)**과 **구현(구체 클래스)**을 분리하여, OCP 원칙을 지키는 구조로 설계

---

## 맺음말
**공연 무대에서 배우를 교체하듯이**, 자바에서 **다형성**을 사용하면 부모 타입을 통해 여러 자식 인스턴스를 유연하게 운영할 수 있습니다. 
**추상 클래스**와 **인터페이스**를 적절히 활용하여, 누가 어떤 기능을 “반드시” 구현해야 하는지를 명확히 제시하고, **역할과 구현의 분리**를 철저히 지킬 수 있습니다.

> **OCP(Open-Closed Principle)**를 준수하여 새로운 기능(새 배우, 새 감독, 새 연출 기법 등)을 추가할 때 기존 코드를 최소한으로만 수정하도록 만들 수 있습니다. 

> 나아가, 유지보수가 쉬운, 확장 가능한 구조를 갖춘 소프트웨어를 구현할 수 있습니다.

