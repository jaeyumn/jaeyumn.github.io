---
title: "[SQL] ANSI Join 에 대해"
categories:
  - study
---

SQL 문법을 공부하던 도중, 대부분의 문법은 비슷하지만 MySQL과 Oracle에서 사용되는 구문에서 조금 차이가 있다는 점을 알게 되었다.

그럼 각각의 데이터베이스에 대한 구문 정보를 모두 알아야 하나?

물론 모두 알아두면 좋겠다만, 조금 비효율적이라는 생각이 든다.

또한 MySQL로 작업하다 Oracle로 DB를 변경해야 하면 다 찾아서 변경해야되는데 여간 번거로운 일이 아닐 것이다.

ANSI 조인은 위와 같은 상황에서 아주 유용하게 쓰일 것이다.

<br>

### ANSI 조인이 뭔데?

---

ANSI 조인은 데이터베이스에서 테이블을 결합할 때 사용되는 표준 SQL 문법 중 하나이다.

이러한 조인은 명시적으로 조인 조건을 정의하여 테이블을 연결하며, SQL 표준이기 때문에 여러 데이터베이스 시스템에서 지원된다.

그러니까 MySQL에서 ANSI 조인으로 작성하면 Oracle 에서도 정상적으로 동작한다는 것이다.

<br>

### ANSI 조인의 장점

---

1. 명시적인 조인 조건을 사용하기 때문에 테이블 간의 관계가 명확하게 드러나고 가독성이 높아진다. 또한 복잡한 쿼리의 경우 여러 개의 조인 조건을 수행해야 하는데, ANSI 조인을 사용하면 각각의 조인 단계를 명확하게 구분해서 쿼리를 이해하고 유지보수하기 쉽다.

2. 최적화된 실행 계획을 생성하여 쿼리의 성능이 향상될 수 있다.
   DB 시스템은 ANSI 조인의 조건과 데이터 분포를 고려하여 적절한 인덱스를 선택하고 더 효율적인 방식으로 데이터를 결합한다.

3. SQL 표준으로 지정된 문법이기 때문에 DB 간에 이식성이 높다.
   다른 DBMS로 마이그레이션하거나, 다른 프로젝트에서 동일한 쿼리를 사용할 때도 일관성을 유지할 수 있다.

4. 각각의 조인 유형을 구분하기 쉽다.
   INNER JOIN, LEFT JOIN, RIGHT JOIN, FULL OUTER JOIN 등과 같이 조인 유형을 직관적으로 구분하여 사용할 수 있다.

5. 조인 조건을 명시적으로 정의하기 때문에 조인 조건을 빠뜨리는 등의 오류를 방지할 수 있다. 또한 두 개 이상의 조인을 사용하는 경우에도 각각의 조인 조건을 분리하여 작성하므로, 오류 발생 가능성이 줄어든다.

<br>

### 정리

---

ANSI 조인은 다음과 같이 작성한다.

```
-- ANSI 조인을 사용한 INNER JOIN 예시
SELECT c.customerName, o.orderID
FROM Customers c
INNER JOIN Orders o
ON c.CustomerID = o.CustomerID;
```

ANSI 조인은 DB 시스템에서 테이블 간의 결합을 명확하게 정의하는 표준 SQL 문법으로, 가독성과 유지보수성을 향상시켜서 효율적인 실행 계획을 생성하는데 용이하다.

또한 다양한 조인 유형을 지원해서 데이터를 다양한 방식으로 조합할 수 있고, SQL 쿼리의 일관성을 유지하는 데 도움된다.

그리고 다양한 DB에서 지원하기 때문에 DB를 옮길 경우 유지보수하는 데 시간이 크게 단축될 수 있다.
