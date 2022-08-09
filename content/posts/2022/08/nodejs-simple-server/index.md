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
```js
const http = require("http");
```
`http`モジュールを利用します。

```js
const server = http.createServer((request, response) => {
    response.writeHead(200, {
        "Content-Type": "text/html"
    });

    const responseMessage = "<h1>Hello World</h1>";
    response.end(responseMessage);
    console.log(`Sent a response : ${responseMessage}`);
});
```
レスポンスのステータスコード(200)、ヘッダー("Content-Type": "text/html")、ボディ(responseMessage)を設定します。

```js
server.listen(port);
```
3000番ポートを使用します。

```bash
node main.js
```
[http://localhost:3000/](http://localhost:3000/)にアクセスします。
