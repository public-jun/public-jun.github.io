---

title: "HugoのサイトにTag機能を追加"
date: 2022-02-04T15:39:20+09:00
draft: false
tags: [42Tokyo, blog, Hugo, TIL]
categories: [Tech]
url: "custom-taxonomies"
---

### きっかけ
ブログにタグ機能が欲しかったから。

### Taxonomyって何？？
> Hugo includes support for user-defined groupings of content called taxonomies. Taxonomies are classifications of logical relationships between content.

タグやカテゴリなどのユーザーが定義した分類のことをタクソノミーという。

- タクソノミー(Taxonomy)

	コンテンツをグルーピングするための分類法

- ターム(Term)
	
	実際の具体的なタグやカテゴリの値

## Taxonomyを使う
---
Hugoはデフォルトでタグとカテゴリをサポートしている。

### confingの設定
`config.yml`
```yaml
taxonomies:
  category: categories
  tag: tags
```

https://[サイトURL]/tags/

https://[サイトURL]/categories/

上記のURLにアクセスするとタグ一覧、カテゴリ一覧ページへアクセスできる。

ちなみにこのサイトでは

- https://public-jun.github.io/tags/

- https://public-jun.github.io/categories/

となる。

### 記事の Front matter内でタグ、カテゴリ追加
`contents/posts/../記事.md`

```md
---
title: "HugoのサイトにTag機能を追加"
date: 2022-02-04T15:39:20+09:00
draft: false
tags: [42Tokyo, blog, Hugo, TIL]  //追加
categories: [Tech] //追加
.
.

---
```
これで「42Tokyo」、「blog」、「Hugo」、「TIL」タグが追加され、

記事自身は「Tech」というカテゴリに分類された。

### 画面上部にMenu表示(Papermod)
---
[PaperMod](https://github.com/adityatelange/hugo-PaperMod)テーマはホームページ右上にMenu(ページへのショートカット)を作成することができる。

`config.yml`
```yml
menu:
  main:
    - identifier: tags
      name: Tags
      url: /tags/
      weight: 1 
    - identifier: categories
      name: Categories
      url: /categories/
      weight: 2
```
このように設定すると画面右上に`Tags`と`categories`へのMenuを表示することができる。

## 終わり
---
次はコメント機能の追加やgoogle analyticsの導入にチャレンジしよかな。

42Tokyoの課題も学んだことアウトプットしていきます:raised_hands:

### 参考URL

- [HUGO document](https://gohugo.io/content-management/taxonomies/)

- https://hugo-de-blog.com/hugo-taxonomy/

- https://maku77.github.io/hugo/taxonomy/basic.html

