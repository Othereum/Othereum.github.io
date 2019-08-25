---
title: "[C++] template의 두 가지 의미"
date: 2019-08-25 18:00:00 +0900
categories: C++
---

어디서 많이 본것같은 제목이라고 느꼈다면, 아마 당신은 C++을 열심히 공부한 사람일 것이다. Effective C++ 항목 42의 제목이 "typename의 두 가지 의미를 제대로 파악하자"이기 때문이다. `의존 이름`에 관한 내용인데, 혹시 모른다면 검색해서 알아보기 바란다.

오늘 얘기하고자 하는 것은 `template`에도 두 가지 의미가 있다는 것이다. 자, 다음 코드를 보자.

```cpp
template <class T>
struct A
{
    template <int N>
    static void f(int a) {}
};

template<>
struct A<int>
{
    static constexpr auto f = 2;
};

template <class T>
void f()
{
    A<T>::f<3>(2);
}
```

함수 `f()`에 템플릿 인자로 `int` 외의 타입을 전달하면 `A<T>::f<3>(2)`는 함수 호출로 해석될 것이다.

그러나 `int`가 전달되면 완전히 다르다. `A<T>::f<3>(2)`는 `f`와 `3`을 비교하고 그 결과를 또 `2`랑 비교하는 문장이 된다. 즉, `(A<T>::f < 3 > 2)` → `(2 < 3 > 2)` → `(true > 2)` → `(false)` 가 되는 것이다.

이를 해결하려면 `template` 키워드를 사용하면 된다.

```cpp
A<T>::template f<3>(2);
```

이처럼 중첩 의존 이름이 템플릿 인자를 받도록 하려는 경우, 이름 바로 앞에 `template`을 붙여준다.

이해를 돕기 위해 `typename`과 `template`가 동시에 쓰이는 경우를 보이며 마치겠다.

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
