---
title: "enable_if 作ってみた"
date: 2022-02-24T21:28:39+09:00
description: "これを読むと enable_if の定義が分かります" 
draft: false
tags: [42Tokyo, c++] 
categories: [42Tokyo]
series: [c++98で自作コンテナ]
url: "cpp-enable-if"
---

## 1. はじめに
* * *
c++98 で STL コンテナを自作するときに `enable_if` を実装する必要があったので今回はそのまとめです。あくまで、c++98 ベースで作っていきます。

前回の [is_integral](https://public-jun.github.io/cpp-is-integral/) を使うので、場合によっては参照してください。

## 2. SFINAE とは
* * *
[cpprefjp より引用](https://cpprefjp.github.io/lang/cpp11/sfinae_expressions.html)
> 「SFINAE (Substitution Failure Is Not An Errorの略称、スフィネェと読む)」は、テンプレートの置き換えに失敗した際に、即時にコンパイルエラーとはせず、置き換えに失敗した関数をオーバーロード解決の候補から除外するという言語機能である。

テンプレート引数の展開に不正があっても、エラーにはならず候補から外され、コンパイルは続行される。

残った候補の中から置き換えに成功したものを実行する。
```cpp
struct Test {
    typedef void type;
};

template <typename T> 
void f(typename T::type) {} // 定義#1

template <typename T> 
void f(T) {}               // 定義#2

int main() {
    f<Test>(10); // 定義#1 の呼び出し
    f<int>(10);  // 定義#2 の呼び出し
}
```
- `f<Test>(10)` について、`Test::type` が存在するので **定義1** が実体化する。
- `f<int>(10)` について、int型は `int::type` が存在しないので **定義1** が候補から除外され、 **定義2** が実体化される。

このようにテンプレート実引数の型を調べて、実体化するテンプレート定義を決定することができる特徴がある。 

## enable_if
* * *
**定義**
```cpp
// プライマリー (primary template)
template<bool, class _Tp = void>
struct enable_if {};

// 部分特殊化 (Partial specialization for true)
template <class _Tp>
struct enable_if <true, _Tp> {
    typedef _Tp type;
};
```
プライマリーの最後のテンプレート仮引数に `class _Tp = void` を用意し、呼び出し時に `enable_if<bool値, 型>` で使用することができます。

_Tp は void型が指定されており省略可能なので、実際には `enable_if<bool値>::type` で使用することが多いです。

### 整数型かどうか
* * *
整数型かどうか判定する`is_integral` は `is_integral<T>::value` で bool値を取り出すことができました。
```cpp
#include <iostream>
#include <type_traits>

template <class T>
void f(T, typename std::enable_if<std::is_integral<T>::value>::type* = NULL)
{
    cout << "Tは整数型" << endl;
}

template <class T>
void f(T, typename std::enable_if<!std::is_integral<T>::value>::type* = NULL)
{
    cout << "Tは整数型以外" << endl;
}

int main()
{
    f(3); // Tは整数型
    f("hello"); // Tは整数型以外
}
```
このように `enable_if` と組み合わせて書くことができます。

`enable_if<true>` のときは `type` が定義されているので `enable_if<true>::type` を取得できますが、`false` のときは定義されていないので取得できません。

## 最後に
* * *
今日は `enable_if` についてまとめてみました。SFINAE を通して改めてメタプログラミング難しいなと思いました。もう少しcppになれないといけないです。

### 参考URL
- [cpprefjp](https://cpprefjp.github.io/lang/cpp11/sfinae_expressions.html)
- [wiki](https://ja.wikipedia.org/wiki/SFINAE)
- [Theoride Technology 部分的特殊化で良く使う部品とSFINAEの利用方法](https://theolizer.com/cpp-school2/cpp-school2-6/)
- [Theoride Technology 部分的特殊化で便利な標準ライブラリの仕組みとその使用例](https://theolizer.com/cpp-school2/cpp-school2-7/)
- https://izadori.net/cpp-templ-sfinae/#
- https://programming-place.net/ppp/contents/cpp/language/023.html#partial_specialization
