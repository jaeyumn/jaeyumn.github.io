---
layout: single
title: "String concatenation as argument to 'StringBuilder.append()' call"
---

```java
StringBuilder sb = new StringBuilder();
sb.append(index + "\n");
```
인텔리제이로 알고리즘 문제를 풀던 도중 index 라는 변수를 줄 별로 출력하기 위해 다음과 같이 코드를 작성하니까 'append'에 노란 밑줄이 생겼다.

<br/>

```text
String concatenation as argument to 'StringBuilder.append()' call
```

그리고 다음과 같이 친절하게 교체할 것을 권장하면서 이유를 말해줬다.

```java
sb.append(index).append("\n");
```

<br/>

두 코드는 사실 동일한 결과를 만들어준다.

하지만 수정된 코드를 권장하는 이유는 해당 방법이 성능 면에서 더 효율적이기 때문이다.

sb.append(index + "\n"); 는 index 와 "\n"을 연결한 후에 그 결과를 StringBuilder에 추가한다.따라서 연결하는 과정에서 연결 연산자인 '+' 를 사용하여 작업을 수행한다.

하지만 sb.append(index).append("\n"); 는 index를 StringBuilder에 추가한 다음 "\n" 문자열을 주가한다. 따라서 중간 결과로 임시 문자열을 생성하는 첫 번째 방법과 달리, 추가 메소드 호출을 통해 중간 결과 없이 직접 StringBuilder에 추가하기 때문에 성능상 이점이 생기는 것이다

일반적인 상황에서는 이 차이가 미미하고 두 가지 방법 모두 잘 동작한다.

하지만 개발을 공부하는 데 있어서 항상 극한의 상황을 고려하라는 글을 많이 봤고 동의하기 때문에, 나는 미미한 성능의 이점이라도 챙기기 위해 아래 방법을 사용하고자 한다.

<br/>

### 추가로 궁금했던 부분
---

sb.append(index + "\n") 을 작성했을 때는 교체를 권장하는 문구가 떴지만 sb.append(index + '\n') 으로 교체해보니까 아무런 문구가 뜨지 않았다.

둘의 차이점은 \n 을 감싸는 따옴표의 종류이다.

"\n"은 문자열로 해석되고 해당 문자열이 그대로 처리되며, '\n'은 개행 문자를 나타내는 특수 문자이기 때문에 이스케이프 문자로 해석되고 단일 문자로 해석될 것으로 예상된다.

여기서 궁금한 점은 둘 다 임시 문자열을 만들어서 중간 결과를 만들어내고 StringBuilder에 추가되는 것일 텐데 왜 인텔리제이는 이 둘을 차별(?) 하는 것인지 잘 모르겠다.

<br/>

### 궁금했던 부분 해결!
---

아주 간단한 이유였다. 여러 실험을 하던 도중 진짜 생각지도 못한 곳에서 이유를 찾게 되었다.

<br/>

```java
StringBuilder sb = new StringBuilder();

sb.append(1 + '\n');
sb.append(2 + '\n');
sb.append("builder" + '\n');
sb.append(3 + '\n' + "\n");
sb.append(4 + '\n');

System.out.println(sb);
```

처음 문제가 발생했던 코드에서 index 변수의 타입은 int 였다.

실험했던 코드의 결과를 보면 첫 번째 줄에서 112가 나온다. 내가 예상했던 결과는 1 다음 줄에 2가 나오는 것이었다.

순간 머리가 띵했다. (이걸 생각 못했다고..?)

다시 정리해보면, sb.append(1 + '\n') 과 sb.append(2 + "\n") 이 다른 점은 문자열 결합 연산에 차이가 있기 때문이다.

sb.append(1 + '\n') 에서 1은 정수형으로 처리되고 '\n' 은 문자 그 자체로 처리되므로 '1' 다음에 줄 바꿈이 즉시 이루어지지 않는다.

반면에 sb.append(2 + "\n") 에서는 '2' 가 문자열로 처리되므로 '2' 다음에 바로 줄 바꿈이 이루어진다.

즉, 마지막 출력에서 sb.append(4 + '\n')이 14로 출력되는 이유는 '\n' 이 줄 바꿈 문자를 의미하며, 아스키코드에서 10에 해당하기 때문에 4 + 10 으로 14가 출력되는 것이다.

<br/>

```
<ASCII Code>
10진: 10
16진: 0x0A
문자: LF (\n, New Line)
```

LF는 줄 바꿈 (Line Feed) 문자를 나타내는 용어이고 해당 문자에는 숫자 10이 할당되어 있다.