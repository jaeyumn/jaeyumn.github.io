---
layout: single
title: "Queue 자료구조에서 add와 offer의 차이"
---

Java에서 Queue 인터페이스를 사용하는 도중 add와 offer는 같은 동작을 하는 것 같은데, 왜 분리되어 있는 걸까? 라는 생각이 들었다.

```java
public interface Queue<E> extends Collection<E> {
    boolean add(E var1);

    @Contract(mutates = "this")
    boolean offer(E var1);

    ...
}
```

소스코드를 봐도 둘 다 boolean을 반환하는데..?

보통 Queue는 LinkedList 자료구조를 활용하기 때문에 LinkedList 에서는 어떻게 구현하고 있는지를 확인해봤다.

```java
public boolean add(E e) {
    this.linkLast(e);
    return true;
}
```

```java
public boolean offer(E e) {
    return this.add(e);
}
```

..?

소스 코드로 구현 원리를 파악할 수는 있지만 offer도 결국 add를 사용하는 것처럼 보였다.

add와 offer가 분리된 명확한 이유를 모르겠어서 구글링의 단계로 돌입했다.

</br>

### add와 offer의 차이

우선 'add'와 'offer' 모두 요소를 컬렉션에 추가하는 데 사용된다.

그럼 차이는 뭘까?

'Collection' 인터페이스에서 'add' 메서드의 반환 값은 항상 'true'이다. 따라서 Queue 인터페이스를 구현한 클래스에서 add를 사용하면 요소가 성공적으로 추가되었는지에 대한 정보를 알 수 없다. 하지만 Queue의 제한된 용량을 초과하여 요소를 추가하는 경우 예외를 발생시킨다.

'offer' 메서드 역시 Queue 인터페이스를 구현한 클래스에서 사용하는 메서드로 요소가 성공적으로 추가되었을 때는 'true'를 반환한다. 그리고 Queue의 용량을 초과하여 요소를 추가할 경우에는 'false'를 반환한다. 따라서 요소가 추가되었는지에 대한 성공 여부를 확인할 수 있다.

</br>

### 그럼 어떤 메서드를 사용해야 할까?

나는 알고리즘 문제를 풀 때 Queue를 사용했는데, 사실 이정도 단계에서는 어떤 메서드를 쓰든 상관이 없을 것 같다.

만약 Queue에 용량 제한이 있는 경우, 데이터를 추가하기 전에 용량을 확인하고자 하면 'offer' 메서드를 사용하면 될 것 같다.

'offer' 메서드는 데이터의 추가 성공 여부를 반환하기 때문에 용량을 체크할 수 있기 때문이다.