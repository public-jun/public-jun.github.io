---
title: "c++98で自作 vector (1)"
date: 2022-03-26T15:28:14+09:00
description: Text about this post
draft: true
tags: [42Tokyo, c++] 
categories: [42Tokyo]
series: [c++98で自作コンテナ]
url: "cpp-my-vector"
---

## 1. はじめに
* * *

c++98 で STL コンテナの vector(c++98 ver) を自作していきます。

完成形は [cppreferece](https://en.cppreference.com/w/cpp/container/vector) 、 [C++ Reference](https://www.cplusplus.com/reference/vector/vector/?kw=vector) の vector を目指します。

## 2. そもそも vector とは？？
* * *
`std::vector` は c++ で標準で使用することができる **動的配列(可変長配列)** クラスです。

通常の配列はサイズを予め指定する必要があり、実行時に動的にサイズを変更することができません。

一方、動的配列はサイズを **自由に増減** することができます。
保持する要素数に合わせて、サイズを変更できるのでメモリ領域を無駄に確保せずに済みます。

通常の配列と同じように各要素は連続して配置しているので、イテレータだけでなく、[]演算子で添字でアクセスすることが可能です。

## 
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
