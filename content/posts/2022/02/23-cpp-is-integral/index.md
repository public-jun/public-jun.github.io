---
title: "c++98 で is_integral 実装"
date: 2022-02-23T13:37:59+09:00
description: "これを読めば c++98 で is_integral 自作できます"
draft: false
tags: [42Tokyo, c++] 
categories: [42Tokyo]
series: [c++98で自作コンテナ]
url: "cpp-is-integral"
---

## 1. はじめに
* * *
c++98 で STL コンテナを自作するときに `is_integral` を実装する必要があったので今回はそのまとめです。

## 2. is_integral とは
* * *
- [cppreference.com](https://en.cppreference.com/w/cpp/types/is_integral) より
```cpp
//Defined in header <type_traits>
namespace std {
  template <class T>
  struct is_integral;

  template <class T>
  inline constexpr bool is_integral_v = is_integral<T>::value; // C++17
}
```
>Checks whether T is an integral type. Provides the member constant value which is equal to true, if T is the type bool, char, char8_t (since C++20), char16_t, char32_t, wchar_t, short, int, long, long long, or any implementation-defined extended integer types, including any signed, unsigned, and cv-qualified variants. Otherwise, value is equal to false.

つまり型`T`が整数型かを調べます。
T が整数型かどうかで `true_type` か `false_type` になるように `integral_constant` から継承します。

`is_integral<T>::value` で **bool値(true もしくは false)** を取得することができます。
*****

以下のような型が整数型として判定されます。
- bool
- char
- char16_t
- char32_t
- wchar_t
- signed char
- short int
- int
- long int
- long long int
- unsigned char
- unsigned short int
- unsigned int
- unsigned long int
- unsigned long long int

enum (列挙型) は c++ では整数型と **判定されません**。

## 3. 実装
***
実装のためには以下のクラステンプレートを理解する必要があります。
- integral_constatnt
- is_integral_helper
- remove_cv
- is_integral
### integral_constant
*****
```cpp
template <class _Tp, _Tp __v>
struct integral_constant {
    static const _Tp value = __v;
    typedef _Tp value_type;
    typedef integral_constant<_Tp, __v> type;
    // constexpr operator value_type() const noexcept { return value; } // c++11 なので未実装
    const value_type operator()() const { return value; }
};

// integral_constantの特殊化 true_type
typedef integral_constant<bool, true> true_type;

// integral_constantの特殊化 false_type
typedef integral_constant<bool, false> false_type;
```
`integral_constat` は **value** と **value_type** の定義を持ち、実体化すると value に bool値が代入されます。

それぞれの特殊化は `typedef` によって **true_type** もしくは **false_type** に命名されます。

### is_integral_helper
*****

```cpp
template <class _Tp>
struct is_integral_helper : public false_type {};

// int型でのクラステンプレートの完全特殊化
template <>
struct is_integral_helper<int> : public true_type {};

// char型でのクラステンプレートの完全特殊化
template <>
struct is_integral_helper<char> : public true_type {};
```

`is_integral_helper` はデフォルトでは false_type を継承し、int型やchar型などの整数型の場合は完全特殊化を用いて **true_type** を継承します。

同様に他の型での完全特殊化も用意します。ここでは割愛します。

### remove_cv
*****
現時点では int型やchar型は判定できるようになりますが、`const int` や `volatile char` など**const修飾子**や**volatile修飾子**(*コンパイルの最適化を抑制する修飾子*)がついていても整数型が判定できなければいけません。

```cpp
// default
template <class _Tp>
struct remove_cv {
    typedef _Tp type;
};

// remove const
template <class _Tp>
struct remove_cv<const _Tp> {
    typedef _Tp type;
};

// remove volatile
template <class _Tp>
struct remove_cv<volatile _Tp> {
    typedef _Tp type;
};

// remove const volatile
template <class _Tp>
struct remove_cv<const volatile _Tp> {
    typedef _Tp type;
};
```
`remove_cv` は部分特殊化を用いることで const 、 volatile 、 const volatile を取り除いた型を `type` で取得することができます。

以上で is_integral を作る材料が揃いました。

### is_integral
*****
```cpp
template <class _Tp>
struct is_integral : is_integral_helper<typename remove_cv<_Tp>::type>::type {};
```
これが `is_integral` の定義です。
`is_integral<_Tp>::value` で bool値を取得できます。

ぱっと見ではわかりづらいかもしれないので、具体例を示します。
- const int の場合
*****
```cpp
template <class const int>
struct is_integral : is_integral_helper<typename remove_cv<const int>::type>::type {};
```

`typename remove_cv<const int>::type` に注目して
```cpp
typename remove_cv<const int>::type
/*
** const int の const が取り除かれ、
** typedef int type として命名されるから
*/
typename int 
```

`is_integral_helper<typename remove_cv<const int>::type>::type` に注目して
```cpp
// remove_cv により
is_integral_helper<int>::type

/*
** template<>
** struct is_integral_helper<int> : public true_type {};
** より true_type の type メンバが使用できる
*/
true_type::type

/*
** true_type は integral_constant<bool, true> であるから 
** 中身は
*/
struct integral_constant {
    static const bool value = true;
    typedef bool value_type;
    typedef integral_constant<bool, true> type;
    const value_type operator()() const { return value; }
};

/*
** typedef integral_constant<bool, type> type; より
*/
struct is_integral : integral_constant<bool, true> {};

/*
** is_integral は integral_constant<bool, true> のメンバを利用できるから
*/
is_integral::value // true
```
流れとしては以上です。

## 最後に
***
`enable_if` で今回の `is_integral` は用いるのですが、`enable_if` の実装自体はそんなに難しそうじゃなかったので、この記事に追記してもいいかもしれません。
完全特殊化、部分特殊化の理解が深まってよかったです。

次は `enable_if` のこと書きます。

### . 参考URL
- [cppreference.com](https://en.cppreference.com/w/cpp/types/is_integral)
- [cplusplus](https://www.cplusplus.com/reference/type_traits/is_integral/?kw=is_integral)
- [cpprefjp](https://cpprefjp.github.io/reference/type_traits/is_integral.html)
- [gcc のソースコード is_integral 該当ファイル](https://github.com/gcc-mirror/gcc/blob/master/libstdc%2B%2B-v3/include/std/type_traits)
- [llvm のソースコード is_integral 該当ファイル](https://github.com/llvm/llvm-project/blob/main/libcxx/include/type_traits)
