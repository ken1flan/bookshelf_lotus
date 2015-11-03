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

## テストを書きながらappsをめぐる

---

### ホームを訪れた場合のテスト

```
# spec/web/features/visit_home_spec.rb

require 'features_helper'

describe 'Visit home' do
  it 'is successfull' do
    visit '/'

    page.body.must_include 'Bookshelf'
  end
end
```

---

### ルーティング

```
# apps/web/config/routes.rb
get '/', to: 'home#index'
```

* アプリケーションの下にルーティング設定がありますね。

---

### コントローラ

```
# apps/web/controllers/home/index.rb
module Web::Controllers::Home
  class Index
    include Web::Action

    def call(params)
    end
  end
end
```

* ページごとにコントローラがいるカンジかしら。

---

### ビュー

```
# apps/web/views/home/index.rb
module Web::Views::Home
  class Index
    include Web::Views
  end
end
```

 * テンプレートとビューが分かれています。

---

### テンプレート

```
# apps/web/templates/home/index.html.erb
<h1>Bookshelf</h1>
```

---

### テストグリーン

```
$ rake test
Run options: --seed 29710

# Running:

.

Finished in 0.018290s, 54.6734 runs/s, 109.3469 assertions/s.

1 runs, 2 assertions, 0 failures, 0 errors, 0 skips
✔ ~/src/bookshelf_lotus [master ↑·5|✔]
$
```

---
