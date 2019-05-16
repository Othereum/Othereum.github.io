---
title: "C++ numeric_limits, min? lowest? 뭔 차이?"
date: 2019-05-16 19:11:00 +0900
categories: C++
---
`numeric_limits`. 산술 타입의 다향한 속성을 표준화된 방법으로 조회할 수 있는 특성 정보 (traits) 클래스다. 예를 들어 int의 최대값은 `numeric_limits<int>::max()`, 최소값은 `numeric_limits<int>::min()`이다. 너무나 직관적이고 당연하며, 의문점도 설명할 것도 없다. 하지만 한 가지 특이점이 있다.

C++ 레퍼런스의 [`numeric_limits`](https://en.cppreference.com/w/cpp/types/numeric_limits) 문서를 보면 [`min`](https://en.cppreference.com/w/cpp/types/numeric_limits/min) 외에도 [`lowest`](https://en.cppreference.com/w/cpp/types/numeric_limits/lowest)라는 함수가 있다. 처음 본 순간 대체 뭔가 싶었다. 최소와 최저. 뭔 차이지? 말장난인가 싶었다. 레퍼런스에는 `lowest`의 설명이 다음과 같이 서술되어 있다.

> 산술 타입 T로 나타낼 수 있는 **최저** 유한 값, 즉 x > y (단, x, y는 모두 유한한 값) 일 때 y가 존재하지 않는 x를 반환합니다.

대체 이것이 `min`과 다를 게 무엇인지 모르겠다. `min`에 대한 설명을 보자.

> 산술 타입 T로 나타낼 수 있는 **최소** 유한 값을 반환합니다.
>
> **denormalization이 있는 부동 소수점 타입의 경우, `min`은 최소 양의 normalized 값을 반환합니다.** 이러한 동작은 특히 정수 타입의 최소값과 비교할 때 예기치 않은 동작이 발생할 수 있다는 점에 유의하십시오. 더 작은 값이 존재하지 않는 값을 찾으려면 `numeric_limits::lowest`를 사용하십시오.

일단 denormalization이 무엇인지 알아보자. 위키백과에선 [비정상 값](https://ko.wikipedia.org/wiki/%EB%B9%84%EC%A0%95%EC%83%81_%EA%B0%92)이라고 하는데, IEEE 부동 소수점에서 일반적으로 표현할 수 있는 가장 작은 숫자보다 더 작은, 0이 아닌 값을 **비정상 값**이라고 한다. 다시 말해, 0 주위의 언더플로 차이를 채워주기 위한 값이다.

여기서 주목해야 할 것은 denormalization이 있는 부동 소수점 타입의 경우 normalized 값 중에서 최소 **양의 값을 반환**한다는 점이다. 이러한 이유로 `numeric_limits<float>::min`은 `1.17549e-38`을 반환한다. 이는 float에서 정상적으로 표현할 수 있는, 0이 아닌 가장 작은 **양수**다. 즉, 우리가 기대했던 **최소값이 아니라는 것**이다.

최소값이 최소값이 아니라니, 뭐 이런 아이러니한 상황이 다 있나 싶다. 때문에 부동 소수점 타입의 경우 일반적으로 원하는 최소값을 얻으려면 `numeric_limits::lowest`를 사용해야 한다.

사실 부동 소수점 타입에 대해서만 `lowest`를 사용할 것이 아니라 대부분의 경우 항상 `lowest`를 사용하는 것이 좋다. **템플릿**을 사용할 경우 `numeric_limits<T>::min`과 같은 코드를 작성할 수 있는데, `T`가 항상 정수 타입이라는 보장이 없기 때문이다.