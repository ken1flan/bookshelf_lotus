# lotus.rb の練習メモ

---

## はじめに
[lotus](http://lotusrb.org/)の[getting started](http://lotusrb.org/guides/getting-started/)をひととおりメモしながらやってみて、感想を述べる予定。

---

## 必要条件

* Web 開発の基礎知識
* ターミナルでBundler、Rakeを使えること
* MVCでアプリケーションを作れること
* PostgreSQLが使えること

---

## インストールと新規プロジェクトの作成

---

### インストールと新規プロジェクトの作成

```
$ gem install lotusrb
$ lotus new bookshelf --database=postgres
```

---

### gemのインストールと起動

```
$ bundle install
$ lotus server
```

![lotus server](http://f.st-hatena.com/images/fotolife/k/ken1flan/20151101/20151101192012.png?1446373257)

---

## Lotusのアーキテクチャ

```
$ cd bookshelf
$ tree -L 1
.
├── Gemfile
├── Rakefile
├── apps
├── config
├── config.ru
├── db
├── lib
├── readme.md
└── spec
```

`apps`以下にアプリケーション。Webインターフェイスや管理者ペイン、HTTP APIなどなど。
`lib`以下にビジネスロジック。モデルやプロダクトとして提供される機能など。
lotusは[クリーンアーキテクチャ](http://blog.tai2.net/the_clean_architecture.html)に強く影響を受けている。

---
