---
title: "[C++] template 및 typename 키워드의 두 가지 의미"
date: 2019-08-25 18:00:00 +0900
categories: C++
---

보통 `template`과 `typename`은 다음과 같은 형태로 많이 봤을 것이다:

```cpp
template <typename T>
void swap(T t);
```

그러나 사실 이 두 키워드는 다른 의미로도 사용된다. 다음 코드를 보자:

```cpp
template <typename T>
struct A
{
    using type = int;

    template <int N>
    static void f(int a) {}
};

template<>
struct A<int>
{
    static constexpr auto type = 3;
    static constexpr auto f = 2;
};

template <typename T>
void f()
{
    A<T>::type num;
    A<T>::f<3>(2);
}
```

함수 `f()`에 템플릿 인자로 `int` 외의 타입을 전달하면 `A<T>::type num`은 `int`형 변수 선언으로 해석되고, `A<T>::f<3>(2)`는 함수 호출로 해석될 것이다.

그러나 `int`가 전달되면 완전히 다르다. `A<T>::type num`은 `3 num`이 되어 컴파일 에러가 나고, `A<T>::f<3>(2)`는 `f`와 `3`을 비교하고 그 결과를 또 `2`랑 비교하는 문장이 된다. 즉, `(A<T>::f < 3 > 2)` → `(2 < 3 > 2)` → `(true > 2)` → `(false)` 가 되는 것이다.

여기서 `A<T>::type`에서 `type`, 그리고 `A<T>::f`에서 `f`를 `의존 이름`이라고 한다. 이름의 의미가 템플릿 파라미터 `T`에 의존한다 하여 붙여진 이름이다.

위의 경우는 전혀 좋은 상황이 아니다. 함수의 의미와 맥락이 모호해지고 혼란스러워지기 때문이다. C++은 이를 위해서 `template`와 `typename`에 또다른 역할을 추가했다:

```cpp
template <typename T>
void f()
{
    typename A<T>::type num;
    A<T>::template f<3>(2);
}
```

위의 예제처럼 의존 이름을 타입으로 사용하려는 경우, 타입 앞에 `typename`을 붙여줌으로써 이것이 타입을 의미한다고 명시해준다. 반대로 `typename`을 붙이지 않을 경우 값(타입이 아님)을 의미한다.

`template`의 경우는 사실 잘 쓰일 일이 없기 때문에 `typename`은 알고 있더라도 `template`는 모르는 경우가 많다. 의존 이름이 템플릿 인자를 받도록 하려는 경우, 의존 이름 바로 앞에 `template`을 붙여준다.

마지막으로 `typename`과 `template`가 동시에 쓰이는 경우를 보여주면서 마치겠다.

```cpp
template <typename T>
struct A
{
    template <typename T>
    struct B {};
};

template <class T>
void f()
{
    typename A<T>::template B<T> a;
}
```
