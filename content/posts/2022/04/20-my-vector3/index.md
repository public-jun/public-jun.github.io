---
title: "c++98で自作 vector (3)"
date: 2022-04-20T16:58:49+09:00
description: 自作vector実装編
draft: true
tags: [42Tokyo, c++] 
categories: [42Tokyo]
series: [c++98で自作コンテナ]
url: "cpp-my-vector3"
---

## 1. はじめに
* * *
[前回のイテレータ実装編](https://public-jun.github.io/cpp-my-vector2/)の続き、実装編(3)です。
この記事では、`vector` の `insert` を作ります。

## 2. vector 概要
* * *
**vector_hpp** [(GitHubより)](https://github.com/public-jun/42_ft_containers/blob/main/includes/containers/vector.hpp)
```cpp
namespace ft {

template <typename T, typename Allocator = std::allocator<T> >
class vector
{
public:
    // Member type
    typedef T                                    value_type;
    typedef Allocator                            allocator_type;
    typedef std::size_t                          size_type;
    typedef std::ptrdiff_t                       difference_type;
    typedef value_type&                          reference;
    typedef const value_type&                    const_reference;
    typedef typename Allocator::pointer          pointer;
    typedef typename Allocator::const_pointer    const_pointer;
    typedef wrap_iter<pointer>                   iterator;
    typedef wrap_iter<const_pointer>             const_iterator;
    typedef ft::reverse_iterator<iterator>       reverse_iterator;
    typedef ft::reverse_iterator<const_iterator> const_reverse_iterator;

private:
    pointer        first_;
    pointer        last_;
    pointer        capacity_last_;
    allocator_type alloc_;

//
// Member-functions...
//
};

} // namespace ft
```
### 定義
***
**template <typename T, typename Allocator = std::allocator<T> >**

- `T` は `vector` に格納する値型を入れます。

- `std::allocator` (c++17で非推奨、c++20で削除)は標準ライブラリ内で使用されるメモリアロケータクラスです。
主にメモリの確保や解放、初期化を行います。
デフォルト引数なので、独自で定義したメモリアロケータクラスを代わりに使用することも可能です。

### メンバ型
***
**typedef typename Allocator::pointer pointer;**

- `std::allocator<T>` はメンバ型に `pointer` を持っています(以下の allocator.h 参照)。 `vector` はそれを利用してメンバ型を定義しています。

**allocator.h(llvm)**
```cpp
template <class _Tp>
class _LIBCPP_TEMPLATE_VIS allocator
    : private __non_trivial_if<!is_void<_Tp>::value, allocator<_Tp> >
{
    // ...
    _LIBCPP_DEPRECATED_IN_CXX17 typedef _Tp*       pointer;
    // ...
}
```
その他の **メンバ型(Member type)** に関してはこのあたりを参照してください。
- [c++日本語リファレンス](https://cpprefjp.github.io/reference/vector/vector.html)
- [cppreference.com](https://en.cppreference.com/w/cpp/container/vector)
- [cppreference.com(日本語)](https://ja.cppreference.com/w/cpp/container/vector)

### メンバ変数
***
```cpp
private:
    pointer        first_;
    pointer        last_;
    pointer        capacity_last_;
    allocator_type alloc_;
```
- `first_` は先頭要素のアドレス
- `last_` は最後の要素のアドレス
- `capacity_last_` は確保したメモリ領域の最後の次のアドレス[(参照URL)](https://public-jun.github.io/cpp-my-vector1/#3-vector-%E3%81%A8%E3%81%AF)
- `alloc_` はメモリアロケータ。

## 2. insert 実装
* * *

### 道具紹介

## 3. その他の関数

## 最後に

### 参考URL


### x. Markdown CheetSheet

#### Text Format

_Italic（斜体）_
*Italic（斜体）*

__Emphasis（強調）__
**Emphasis（強調）**

~~Strikethrough（取り消し線）~~

<details><summary>これは詳細表示の例です。</summary>詳細をこっちに書きます。</details>

This is `inline`.

### List
* text
    * test
    * test

- text
    - test
    - test

1. text
1. test
    1. test

#### Horizontal rules
* * *
***
*****
- - -
---------------------------------------

#### Blockquotes（引用）
> This is Blockquotes

#### Links（参照）
[]()

#### Images（画像）
![]()

#### Tables（表）
| id     | name    | date       |
| ------ | ------- | ---------- |
| 1      | test    | 2019-01-01 |
| 2      | test    | 2019-01-02 |
| 3      | test    | 2019-01-03 |
