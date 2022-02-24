---
title: "enable_if 作ってみた"
date: 2022-02-24T21:28:39+09:00
description: "これを読むと enable_if  を定義して使えるようになれます" 
draft: true
tags: [42Tokyo, c++] 
categories: [42Tokyo]
series: [c++98で自作コンテナ]
url: "cpp-enable-if"
---

## 1. はじめに
* * *
c++98 で STL コンテナを自作するときに `enable_if` を実装する必要があったので今回はそのまとめです。あくまで、c++98 ベースで作っていきます。

## 2. SFINAE とは
[cpprefjp より引用](https://cpprefjp.github.io/lang/cpp11/sfinae_expressions.html)
> 「SFINAE (Substitution Failure Is Not An Errorの略称、スフィネェと読む)」は、テンプレートの置き換えに失敗した際に、即時にコンパイルエラーとはせず、置き換えに失敗した関数をオーバーロード解決の候補から除外するという言語機能である。


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
