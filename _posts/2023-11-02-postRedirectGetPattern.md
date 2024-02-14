---
title: "[Spring] PRG(Post-Redirect-Get) 패턴"
categories:
  - server
---

`PRG(Post-Redirect-Get)` 패턴은 웹 애플리케이션에서 사용되는 디자인 패턴 중 하나로, 중복 데이터 제출 문제와 사용자 경험을 향상시키기 위한 패턴이다.

<br>

### 언제 사용할까?

---

예시로 사용자가 어떤 제품을 구매하는 요청을 보냈다고 가정한다.

이때 요청은 POST 요청이며, 어떠한 이유로 인해 페이지가 새로고침 됐다.

그러면 웹 브라우저에서는 마지막에 서버에 전송된 데이터를 요청하게 되고 POST 요청이 다시 전송될 것이다.

이로 인해 중복 데이터 제출이 일어나고 의도치 않은 동작(중복 등록, 중복 구매 등)이 발생할 수 있다.

이러한 상황을 방지하기 위해 사용할 수 있는 패턴 중 하나가 PRG 패턴이다.

PRG(Post-Redirect-Get), 말 그대로 `POST` 요청 후 비즈니스 로직을 처리하고 다시 `GET` 요청을 수행하게 하는 것이다.

그러면 페이지가 새로고침 되어도 웹 브라우저는 마지막 요청인 `GET` 요청을 전송하게 된다.

<br>

### 구현 예시

---

- PRG 패턴을 적용하기 전 코드

```java
@Controller
public class TestController {

  /**
  * 장바구니 페이지 이동
  */
  @GetMapping("/cart")
  public String cartForm() {
    return "cart";
  }

  /**
   * 주문
   */
  @PostMapping("/order")
  public String order() {
    return "order-detail";
  }
}
```

- PRG 패턴을 적용한 코드

```java
@Controller
public class TestController {

  /**
  * 장바구니 페이지 이동
  */
  @GetMapping("/cart")
  public String cartForm() {
    return "cart";
  }

  /**
   * 주문 처리 후 /result로 redirect
   */
  @PostMapping("/order")
  public String order() {
    return "redirect:/result";
  }

  /**
   * 주문 완료 페이지 호출
   */
  @GetMapping("/result")
  public String orderResultForm() {
    return "order-detail";
  }
}
```
