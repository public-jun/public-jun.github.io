---
title: "Node.jsでHTTPサーバを構築"
date: 2022-08-09T14:31:42+09:00
description: Node.jsでシンプルなHTTPサーバを構築
draft: true
tags: [ Node.js, JavaScript ] 
categories: [Node.js]
series: [other]
url: "nodejs-http-server"
---

## 1. HTTPサーバ
通常、ユーザーがウェブブラウザからWebページを閲覧するとき、URLをクリックしWebページを表示していることしかわかりません。しかし、内部的にはウェブブラウザがWebページに必要なデータ(HTMLファイルやJavaScriptファイルなど)を取得し表示しています。

ユーザが操作するコンピュータをWebクライアント、Webページに必要なデータが保存されているコンピュータをWebサーバといいます。

WebクライアントとWebサーバの通信の手順やルールのことをHTTPプロトコルといい、HTTPに従って行われる通信のことをHTTP通信といいます。

WebクライアントがWebサーバにデータを要求することをHTTPリクエスト、Webサーバが要求に応じてデータを返すことをHTTPレスポンスといいます。

つまり、HTTPサーバとはHTTPというルールに従ったレスポンスを返すWebサーバのことです。
## 2. Node.jsでアプリケーション作成
```js
const http = require("http");
const port = 3000;

const server = http.createServer((request, response) => {
    response.writeHead(200, {
        "Content-Type": "text/html"
    });

    const responseMessage = "<h1>Hello World</h1>";
    response.end(responseMessage);
    console.log(`Sent a response : ${responseMessage}`);
});

server.listen(port);

```
xxxx

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
