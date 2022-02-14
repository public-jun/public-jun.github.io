---
title: "c++ における typename とは"
date: 2022-02-13T19:19:37+09:00
draft: false
tags: [42Tokyo, c++] 
categories: [42Tokyo]
series: [c++98で自作コンテナ]
url: "cpp-typename"
---

### 1. はじめに
* * *
STL コンテナを自作している時、 `typename` の使い方をしっかり理解していなかったので自分なりにまとめてみました。


### 2. typename とは
* * *
[cppreference.com typename の説明より](https://en.cppreference.com/w/cpp/keyword/typename)
> - In the template parameter list of a template declaration, typename can be used as an alternative to class to declare type template parameters .
> - Inside a declaration or a definition of a template, typename can be used to declare that a dependent qualified name is a type.

- c++ のテンプレート宣言時には、型名を明示する際に `class` の代わりに `typename` を使用してテンプレートパラメータを宣言することができます。

```cpp
template<typename T> void f() {}
template<class T>    void f() {}
```

これはどちらでもコンパイルが通ります。

- テンプレートの宣言や定義の中で， `typename` はテンプレートパレメータ内部にネストされた依存型名が明示的に型である宣言をするために使うことができます．

例えば、以下の例を考えてみます。
```cpp
#include <iostream>

class my_class {
public:
  enum type_ { val1 = 100 };
};

template<typename T>
class test_my_class {
public:
  void print() {
    // T::type_ val1 = T::type_::val1;
    typename T::type_ val1 = T::type_::val1;
    std::cout << val1 << std::endl;
  }
};

int main(int, char**) {
  test_my_class<my_class> tc;
  tc.print();
  return 0;
}
```
`test_my_class` のクラステンプレートのテンプレート引数として `my_class` が与えられた状況です。
```cpp
// T::type_ val1 = T::type_::val1;
typename T::type_ val1 = T::type_::val1;
```
上記 2 行の違いは `typename` がついているかどうかです。
1行目をコメントアウトしなければ、コンパイルが成功しません。
これはコンパイル時には `T::type_` が **クラス名::型** なのか **クラス名::定数** なのか判断できないからです。

この問題を解決するために、**ネストされた依存名(T)** の **型(type_)** を使う場合、明示的に `typename` を書きます。

```cpp
typename Foo<T>::xxx //xxxは型名
```

### 最後に
* * *
今回は `typename` についてまとめてみました。
これで少し STL コンテナのコードを読めそうです。

###  参考URL
- https://en.cppreference.com/w/cpp/keyword/typename
- https://marycore.jp/prog/cpp/template-class-typename/
- https://daily-tech.hatenablog.com/entry/2018/12/01/103658
