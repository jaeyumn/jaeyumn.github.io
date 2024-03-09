---
title: "[Spring] Bean 생명주기 콜백"
categories:
  - study
---

스프링 프레임워크는 객체의 생성과 소멸 시점을 관리하고 제어할 수 있는 `Bean 생명주기` 기능을 제공한다.

크게 **인터페이스(InitializingBean, DisposableBean)**, **설정 정보(초기화, 종료 메서드 지정)**, **애너테이션(@PostConstruct, @PreDestroy)** 방법으로 빈 생명주기 콜백을 지원하며, 이러한 콜백 기능을 활용해서 객체의 초기화와 종료 과정을 관리할 수 있다.

빈 생명주기 콜백은 데이터베이스 연결 풀 초기화, 리소스 관리, 로깅, 캐싱, 스레드 관리 등과 같은 다양한 작업에 활용되며, 초기화와 정리 작업을 적절하게 수행하면 애플리케이션의 안정성과 성능을 향상시킬 수 있다.

<br>

### Bean 생명주기 순서

---

스프링에서 빈의 생명주기는 다음과 같은 순서로 진행된다.

1. 스프링 컨테이너 생성
2. 스프링 빈 생성
3. 의존관계 주입
4. 초기화 콜백 메서드 호출
5. 빈 객체 사용
6. 소멸 콜백 메서드 호출
7. 스프링 종료

<br>

### 1. InitializingBean 및 DisposableBean 인터페이스 사용

---

인터페이스를 구현하는 방법은 스프링에서 제공하는 표준 인터페이스를 사용하기 때문에 구현이 간단하고 빠르게 설정할 수 있다.

하지만 스프링 전용 인터페이스이기 때문에 외부 라이브러리에 적용할 수 없다. 또한 스프링 프레임워크와 강하게 결합하게 되며, 이는 유연성을 제한할 수 있다.

- `InitializingBean` 인터페이스를 구현한 빈은 `afterPropertiesSet()` 메서드를 오버라이드해서 초기화 로직을 구현한다.
- `DisposableBean` 인터페이스를 구현한 빈은 `destroy()` 메서드를 오버라이드해서 소멸 로직을 구현한다.

```java
public class testBean implements InitializingBean, DisposableBean {

    // 초기화 로직
    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("초기화 콜백 메서드 호출");
    }

    // 소멸 로직
    @Override
    public void destroy() throws Exception {
        System.out.println("소멸 콜백 메서드 호출");
    }
}
```

<br>

### 2. 설정 정보에 지정

---

빈 클래스 내부에서 사용자 정의 초기화 및 소멸 메서드를 정의할 수 있다.

빈의 생명주기 콜백 메서드를 빈 설정에서 지정하기 때문에 빈 클래스 자체에 인터페이스나 애너테이션을 추가하지 않아도 된다.

코드가 아니라 설정 정보를 사용하기 때문에 외부 라이브러리에도 적용이 가능하다.

하지만 설정 파일에서 초기화 및 소멸 메서드를 지정해야 하기 때문에 빈 설정이 더 복잡해질 수 있다.

```java
public class testBean {

  // 사용자 정의 초기화 메서드
    public void init() {
        System.out.println("초기화 콜백 메서드 호출");
    }

    // 사용자 정의 소멸 메서드
    public void destroy() {
        System.out.println("소멸 콜백 메서드 호출");
    }
}
```

스프링 설정 파일에서 빈을 정의할 때, 초기화 및 소멸 메서드를 지정한다.

```java
<bean id="myBean" class="com.example.testBean" init-method="init" destroy-method="destroy" />
```

<br>

### 3. @PostConstruct 및 @PreDestroy 애너테이션 사용

---

빈 클래스에 애너테이션만 추가하면 되므로 코드가 깔끔하고 가독성이 좋다. 또한 스프링 컨테이너에서 초기화 및 소멸 메서드를 찾아 자동으로 호출해주므로 빈 클래스 자체를 수정하지 않고도 빈의 생명주기를 제어할 수 있다.

`javax.annotation` 패키지로 자바 표준이기 때문에 스프링이 아닌 다른 컨테이너에서도 동작하지만, 다른 패키지와의 충돌이 있을 수 있다.

- `@PostConstruct` 애너테이션이 붙은 메서드는 빈이 생성된 후에 호출된다.
- `@PreDestroy` 애너테이션이 붙은 메서드는 빈이 소멸되기 직전에 호출된다.

```java
public class testBean {

    // 빈 초기화 메서드
    @PostConstruct
    public void init() {
        System.out.println("초기화 콜백 메서드 호출");
    }

    // 빈 소멸 메서드
    @PreDestroy
    public void destroy() {
        System.out.println("소멸 콜백 메서드 호출");
    }
}
```

<br>

### 정리

---

각 방법은 상황에 따라 적절하게 선택할 수 있고, 개발자의 선호도와 프로젝트 요구 사항에 따라 다를 수 있다.

하지만 `@PostConstruct`, `@PreDestroy` 애너테이션은 최신 스프링에서 권장하는 방법이기도 하고 사용하기 편리해서 나는 해당 방법을 주로 사용할 것 같다.

유일한 단점으로는 외부 라이브러리에 적용하지 못한다는 것인데, 이러한 예외 상황에서는 설정 정보에 직접 정의하는 방법을 사용하면 된다.
