---
title: "c++98で自作 vector (2)"
date: 2022-04-14T21:48:22+09:00
description: 自作vectorイテレータ実装編
draft: false
tags: [42Tokyo, c++] 
categories: [42Tokyo]
series: [c++98で自作コンテナ]
url: "cpp-my-vector2"
---

## 1. はじめに
* * *
[前回の基礎知識編](https://public-jun.github.io/cpp-my-vector1/)の続き、実装編です。
この記事では、vectorのイテレータを作ります。

## 2. 概要を掴む 
* * *
実装のイメージをつけるために、[江添亮のC++入門](https://cpp.rainy.me/)の[std::array](https://cpp.rainy.me/020-array.html)から [vectorの実装 : メモリー確保](https://cpp.rainy.me/034-vector-memory-allocation.html)までを写経しました。

この資料ではイテレータをポインタのエイリアスで実装しています。

[llvm](https://github.com/llvm/llvm-project/blob/main/libcxx/include/vector) や [gcc](https://github.com/gcc-mirror/gcc/blob/master/libstdc%2B%2B-v3/include/bits/stl_vector.h) はイテレータを用意しているので、最終的にはポインタで操作しているところをイテレータに置き換えていきます。

## 3. イテレータ実装
自作 vector は `ft`という名前空間に作ります。
### iterator_traits
* * *
**iterator_traits.hpp**
```cpp
#ifndef ITERATOR_TRAITS_HPP
#define ITERATOR_TRAITS_HPP

#include <cstddef> // ptrdiff_t
#include <iterator>

namespace ft {

/// primary iterator_traits
template <class Iterator>
struct iterator_traits {
    typedef typename Iterator::difference_type   difference_type;
    typedef typename Iterator::value_type        value_type;
    typedef typename Iterator::pointer           pointer;
    typedef typename Iterator::reference         reference;
    typedef typename Iterator::iterator_category iterator_category;
};

/// Partial specialization for pointer types
template <class T>
struct iterator_traits<T*> {
    typedef ptrdiff_t                       difference_type;
    typedef T                               value_type;
    typedef T*                              pointer;
    typedef T&                              reference;
    typedef std::random_access_iterator_tag iterator_category;
};

/// Partial specialization for const pointer types.
template <class T>
struct iterator_traits<const T*> {
    typedef ptrdiff_t                       difference_type;
    typedef T                               value_type;
    typedef const T*                        pointer;
    typedef const T&                        reference;
    typedef std::random_access_iterator_tag iterator_category;
};
} // namespace ft

#endif
```
`iterator_traits` は、イテレータに関する型情報を取得するためのクラスです。
部分特殊化により、ポインタにも対応しています。イテレータは少なくともこれらが定義されなけらばいけません。

`iterator_traits` が保持する情報は以下です。

| 型                | 説明    |
| ----------------- | ------- |
| difference_type   | イテレータの差を示す符号付き整数型    |
| value_type        | イテレータが示す値型                  |
| pointer           | ポインタ型                            |
| reference         | イテレータが示す参照型                |
| iterator_category | イテレータの分類をを表す型            |

### wrap_iter (vector のイテレータ)
`wrap_iter.hpp`
```cpp
#ifndef WRAP_ITER_HPP
#define WRAP_ITER_HPP

#include <iterator_traits.hpp>

namespace ft {
template <class Iter>
class wrap_iter
{
protected:
    Iter                          current_;
    typedef iterator_traits<Iter> traits_type;

public:
    typedef Iter                                    iterator_type;
    typedef typename traits_type::value_type        value_type;
    typedef typename traits_type::difference_type   difference_type;
    typedef typename traits_type::pointer           pointer;
    typedef typename traits_type::reference         reference;
    typedef typename traits_type::iterator_category iterator_category;

    // Member functions
    //// constructor
    wrap_iter() : current_(NULL) {}
    explicit wrap_iter(iterator_type x) : current_(x) {}
    //// copy constructor
    template <class U>
    wrap_iter(const wrap_iter<U>& other) : current_(other.base())
    {}

    //// assignment operator
    template <class U>
    wrap_iter& operator=(const wrap_iter<U>& other)
    {
        // std::cout << "assignment called" << std::endl;
        if (current_ != other.base())
            current_ = other.base();
        return *this;
    }
    //// destructor
    ~wrap_iter() {}

    iterator_type base() const { return current_; }
    reference     operator*() const { return *current_; }
    pointer       operator->() const { return current_; }
    reference     operator[](difference_type n) const { return *(*this + n); }
    wrap_iter&    operator++()
    {
        ++current_;
        return *this;
    }
    wrap_iter& operator--()
    {
        --current_;
        return *this;
    }
    wrap_iter operator++(int)
    {
        wrap_iter tmp = *this;
        ++current_;
        return tmp;
    }
    wrap_iter operator--(int)
    {
        wrap_iter tmp = *this;
        --current_;
        return tmp;
    }
    wrap_iter operator+(difference_type n) const
    {
        return wrap_iter(current_ + n);
    }
    wrap_iter operator-(difference_type n) const
    {
        return wrap_iter(current_ - n);
    }
    wrap_iter& operator+=(difference_type n)
    {
        current_ += n;
        return *this;
    }
    wrap_iter& operator-=(difference_type n)
    {
        current_ -= n;
        return *this;
    }
};
//
// Non member-functions...
//
} // namespace ft

#endif
```

今回 vector のイテレータは `wrap_iter`とします。命名は llvm の実装から取ってきました。イテレータの種類は **ランダムアクセスイテレータ** なので[その要件](https://public-jun.github.io/cpp-my-vector1/#%E3%82%A4%E3%83%86%E3%83%AC%E3%83%BC%E3%82%BF)を満たす必要があります。

vector のイテレータが保持するメンバ変数は上記の `iterator_traits` の型情報と、現在のポインタを表す `current_` です。vector は各要素が連続して配置しているので、ポインタを操作し要素にアクセスすることができます。

非メンバ関数はこの記事には書いてません。別途[リポジトリ](https://github.com/public-jun/42_ft_containers/blob/main/includes/utils/wrap_iter.hpp)を参照してください。

これで vector のイテレータは実装完了です。

## 最後に
次回は、vectorの実装に入っていきます。
### 参考URL
- https://cpprefjp.github.io/reference/iterator/iterator_traits.html
- https://marycore.jp/prog/cpp/implement-iterator-traits/
