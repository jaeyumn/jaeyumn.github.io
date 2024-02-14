---
title: "[JPA] No default constructor for entity"
categories:
  - server
---

JPA를 처음 사용할 때 만났던 에러이다.

Entity 클래스에 기본 생성자를 만들어주지 않으면 발생한다.

해결방법은 Lombok을 통해 `@NoArgsConstructor` 애터네이션을 Entity 클래스에 붙여주거나 Entity 기본 생성자를 만들어주면 된다.

<br>

### 왜 기본 생성자를 만들어줘야 할까?

---

<br>
우선 JPA는 Entity 클래스를 통해 DB와 매핑한다.

이때, 데이터베이스에서 조회한 데이터를 Entity 객체에 매핑할 때 Reflection을 통해 객체를 생성하고 데이터를 할당한다.

여기서 Reflaction이 객체를 생성할 때 기본 생성자를 필요로 하기 때문에 Entity 클래스에 기본 생성자를 만들어 주는 것이다.

Reflaction은 클래스 이름만으로 생성자, 필드, 메서드 등 접근이 가능하지만 **생성자의 매개변수 정보**는 접근할 수 없다.

즉, `AllArgsConstructor`가 있어도 해당 생성자를 호출할 수 없기 때문에 Reflection은 기본 생성자로 객체를 생성하고 필드 값을 강제로 매핑해주는 방식을 사용한다.

<br>

### Reflection을 사용하는 이유

---

<br>
Entity를 생성할 때 Reflection을 사용하는 이유는 `Entity`가 어떤 타입인지 JPA는 알 수 없기 때문이다.

그래서 Reflection을 통해 동적으로 객체를 생성하는 것이다.

<br>

### 결론

---

JPA를 사용할 때는 Entity에 기본 생성자를 만들어 주자.
