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

### コンテナとアプリケーション

`Web`という定数を不思議に思っただろうか？
Lotusはコンテナアーキテクチャをデフォルト採用している。
それによって、ひとつのプロジェクトで複数のアプリケーションを持つことができる。
すべてのアプリケーションは `apps/`以下にあり、
デフォルトのアプリケーションは `Web`と名付けられています。

Lotusのコアフレームワークは、コンテナの起動時に複製され、
それぞれ別のアプリケーション向けにされた設定を邪魔することがありません。

---

## 新しいアクションを作る

---

### テスト

```
# spec/web/features/list_books_spec.rb
require 'features_helper'

describe 'List books' do
  it 'displays each book on the page' do
    visit '/books'

    within '#books' do
      assert page.has_css?('.book', count: 2), "Expected to find 2 books"
    end
  end
end
```

---

### ジェネレータ

```
$ lotus generate action web books#index
      insert  apps/web/config/routes.rb
      create  spec/web/controllers/books/index_spec.rb
      create  apps/web/controllers/books/index.rb
      create  apps/web/views/books/index.rb
      create  apps/web/templates/books/index.html.erb
      create  spec/web/views/books/index_spec.rb
$
```

---

### books/index.html.erbの編集

`apps/web/templates/books/index.html.erb`

```
<h1>Bookshelf</h1>
<h2>All books</h2>

<div id="books">
  <div class="book">
    <h3>Patterns of Enterprise Appilcation Architecture</h3>
    <p>by <strong>Martin Fowler</strong></p>
  </div>

  <div class="book">
    <h3>Test Driven Development</h3>
    <p>by <strong>Kent Bech</strong></p>
  </div>
</div>
```

---

### テストグリーン

```
$ rake test
Run options: --seed 6084

# Running:

....

Finished in 0.039437s, 101.4278 runs/s, 126.7847 assertions/s.

4 runs, 5 assertions, 0 failures, 0 errors, 0 skips
$
```

---

### レイアウト

`apps/web/templates/application.html.erb`

```
<!DOCTYPE html>
<html>
  <head>
    <title>Bookshelf</title>
  </head>
  <body>
    <h1>Bookshelf</h1>
    <%= yield %>
  </body>
</html>
```

---

### ブラウザで確認

![http://localhost:2300/books](http://f.st-hatena.com/images/fotolife/k/ken1flan/20151104/20151104073423.png?1446590082)

---

## エンティティでデータをモデリングする

---

### モデルジェネレータ

```
$ lotus generate model book
      create  lib/bookshelf/entities/book.rb
      create  lib/bookshelf/repositories/book_repository.rb
      create  spec/bookshelf/entities/book_spec.rb
      create  spec/bookshelf/repositories/book_repository_spec.rb
```

* bookshelfとなっているところが、親ディレクトリ名になってるっぽい

---

### エンティティの役割

エンティティはRubyのプレーンなオブジェクトで、データベース構造については何も知りません。振る舞いに注力してほしいからです。

```
# lib/bookshelf/entities/book.rb
class Book
  include Lotus::Entity
  attributes :title, :author
end
```

```
# spec/bookshelf/entities/book_spec.rb
require 'spec_helper'

describe Book do
  it 'can be initialised with attributes' do
    book = Book.new(title: 'Refactoring')
    book.title.must_equal 'Refactoring'
  end
end
```

---

### リポジトリを使う

---

#### .env でDB接続情報の管理

今っぽく `.env` でDBの接続情報を管理しています。
開発環境は`.env.development`、テスト環境は`.env.test`で管理されています。

```
# .env.development
BOOKSHELF_DATABASE_URL="postgres://localhost/bookshelf_development"
WEB_SESSIONS_SECRET="d019503e1799a381fd83511921ccba0cedc472954de5279e2058ca2a638ae64d"
```

---

#### データベースの作成

```
$ lotus db create
WARN: Unresolved specs during Gem::Specification.reset:
      mime-types (< 3, >= 1.16)
WARN: Clearing out unresolved specs.
Please report a bug if this causes problems.
$ psql -l
                                          List of databases
            Name            |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges   
----------------------------+----------+----------+-------------+-------------+-----------------------
 bookshelf_development      | ken1flan | UTF8     | ja_JP.UTF-8 | ja_JP.UTF-8 |
   :
   :
```

---

#### DBスキーマの変更のためのマイグレーション

```
$ lotus generate migration create_books
WARN: Unresolved specs during Gem::Specification.reset:
      mime-types (< 3, >= 1.16)
WARN: Clearing out unresolved specs.
Please report a bug if this causes problems.
      create  db/migrations/20151108095957_create_books.rb
$
```

---

```
# db/migrations/20151108095957_create_books.rb
Lotus::Model.migration do
  change do
    create_table :books do
      primary_key :id
      column :title, String, null: false
      column :author, String, null: false
    end
  end
end
```

---

```
$ lotus db migrate
WARN: Unresolved specs during Gem::Specification.reset:
      mime-types (< 3, >= 1.16)
WARN: Clearing out unresolved specs.
Please report a bug if this causes problems.
✔ ~/src/bookshelf [master ↑·1|✚ 1…1]
$
```

---

(memo)なんかわかんないけど、一回失敗した…

```
$ lotus db migrate
WARN: Unresolved specs during Gem::Specification.reset:
      mime-types (< 3, >= 1.16)
WARN: Clearing out unresolved specs.
Please report a bug if this causes problems.
ERROR:  relation "schema_migrations" does not exist at character 27
STATEMENT:  SELECT NULL AS "nil" FROM "schema_migrations" LIMIT 1
ERROR:  relation "schema_info" does not exist at character 27
STATEMENT:  SELECT NULL AS "nil" FROM "schema_info" LIMIT 1
$
```

---

#### リポジトリで遊んでみる

```
$ lotus console
irb(main):002:0> BookRepository.all
=> []
irb(main):003:0> book = Book.new(title: "TDD", author: "Kent Beck")
=> #<Book:0x007fb5f5c49520 @title="TDD", @author="Kent Beck">
irb(main):004:0> BookRepository.create(book)
=> #<Book:0x007fb5f523b9c0 @title="TDD", @author="Kent Beck", @id=1>
irb(main):005:0> BookRepository.find(1)
=> #<Book:0x007fb5f522b318 @title="TDD", @author="Kent Beck", @id=1>
irb(main):006:0>
```

---

### 動的データを表示してみる

---

#### テスト

Bookを動的に作成するようにします。

```
describe 'List books' do
  before do
    BookRepository.clear

    BookRepository.create(Book.new(title: 'PoEEA', author: 'Martin Fowler'))
    BookRepository.create(Book.new(title: 'TDD', author: 'Kent Beck'))
  end
  it 'displays each book on the page' do
    visit '/books'

    within '#books' do
      assert page.has_css?('.book', count: 2), "Expected to find 2 books"
    end
  end
end
```

---

#### テストDBのセットアップ

```
# Define ENV variables for test environment
BOOKSHELF_DATABASE_URL="postgres://localhost/bookshelf_test"
WEB_SESSIONS_SECRET="2f9352a64d07323b4491bff390a190ae1bb44b58deb67afa327d2c92ad5a0c1d"
```

```
LOTUS_ENV=test lotus db prepare
WARN: Unresolved specs during Gem::Specification.reset:
      mime-types (< 3, >= 1.16)
WARN: Clearing out unresolved specs.
Please report a bug if this causes problems.
ERROR:  database "bookshelf_test" does not exist
STATEMENT:  DROP DATABASE bookshelf_test;
ERROR:  relation "schema_migrations" does not exist at character 27
STATEMENT:  SELECT NULL AS "nil" FROM "schema_migrations" LIMIT 1
ERROR:  relation "schema_info" does not exist at character 27
STATEMENT:  SELECT NULL AS "nil" FROM "schema_info" LIMIT 1
✔ ~/src/bookshelf [master|✚ 1]
$
```

…なんかエラーが。何度かやってるうちにできてた。ふーむ？

#### viewのテストの追加

```
# spec/web/views/books/index_spec.rb
require 'spec_helper'
require_relative '../../../../apps/web/views/books/index'

describe Web::Views::Books::Index do
  let(:exposures) { Hash[books: []] }
  let(:template)  { Lotus::View::Template.new('apps/web/templates/books/index.html.erb') }
  let(:view)      { Web::Views::Books::Index.new(template, exposures) }
  let(:rendered)  { view.render }

  it "exposes #index" do
    view.books.must_equal exposures.fetch(:books)
  end

  describe "when there are no books" do
    it "shows a placeholder message" do
      rendered.must_include('<p class="placeholder">There are no books yet</p>')
    end
  end

  describe "when there are books" do
    let(:book1) { Book.new(title: 'Refactoring', author: 'Martin Fowler') }
    let(:book2) { Book.new(title: 'Domain Driven Design', author: 'Kent Bech') }

    let(:exposures) { Hash[books: [book1, book2]] }

    it 'lists them all' do
      rendered.scan(/class="book").count.must_equal 2
      rendered.must_include('Refactoring')
      rendered.must_include('Domain Driven Design')
    end

    it 'hides the placeholder message' do
      rendered.wont_include('<p class="placeholder">There are no books yet</p>')
    end
  end
end
```

---

#### template の準備

```
# apps/web/templates/books/index.html.erb
<h2>All books</h2>

<% if books.any? %>
  <% books.each do |book| %>
    <div class="book">
      <h2><%= book.title %></h2>
      <p><%= book.author %></p>
    </div>
  <% end %>
<% else %>
  <p class="placeholder">There are no books yet.</p>
<% end %>
```

---

#### controllerのテストの修正

```
# spec/web/controllers/books/index_spec.rb
require 'spec_helper'
require_relative '../../../../apps/web/controllers/books/index'

describe Web::Controllers::Books::Index do
  let(:action) { Web::Controllers::Books::Index.new }
  let(:params) { Hash[] }

  before do
    BookRepository.clear

    @book = BookRepository.create(Book.new(title: "TDD", author: "Kent Beck"))
  end

  it "is successful" do
    response = action.call(params)
    response[0].must_equal 200
  end

  it 'expose all books' do
    action.call(params)
    action.exposures[:books].must_equal [@book]
  end
end
```

---

#### controllerの修正

`books`をさらす(expose)でtemplateのほうで使えるようにしてるのかー。
あと、`book`は`BookRepository`から受け取る、と。
なるほどねぇ。

```
# apps/web/controllers/books/index.rb
module Web::Controllers::Books
  class Index
    include Web::Action

    expose :books

    def call(params)
      @books = BookRepository.all
    end
  end
end
```

---

#### ブラウザで見てみる

![ブラウザで](http://f.st-hatena.com/images/fotolife/k/ken1flan/20151114/20151114214201.png?1447504935)

---

### レコードを作るフォーム

---

#### feature テスト

```
# spec/web/features/add_book_spec.rb
require 'features_helper'

describe 'Books' do
  after do
    BookRepository.clear
  end

  it 'can create a new book' do
    visit '/books/new'

    within 'form#book-form' do
      fill_in 'Title', with: 'New book'
      fill_in 'Author', with: 'Some author'

      click_button 'Create'
    end
  end

  current_path.must_equal('/books')
  assert page.has_content?('New book')
end
```

---

#### フォームの基礎を作る

```
$ lotus generate action web books#new --url=/books/new
      insert  apps/web/config/routes.rb
      create  spec/web/controllers/books/new_spec.rb
      create  apps/web/controllers/books/new.rb
      create  apps/web/views/books/new.rb
      create  apps/web/templates/books/new.html.erb
      create  spec/web/views/books/new_spec.rb
✔ ~/src/bookshelf [master ↑·1|✚ 1…5]
$
```

`apps/web/config/routes.rb`にも一行追記されています。

---

#### フォームヘルパーを使う

`apps/web/templates/books/new.html.erb`

```
<h2>Add book</h2>

<%=
  form_for *book, '/books' do
    div class: 'input' do
      label :text
      text_field :title

      div class: 'input' do
        label :author
        text_field :author
      end

      div class: 'controlls' do
        submit 'Create book'
      end
    end
  end
%>
```

---

### フォームをサブミットする

```
$ lotus generate action web books#create
      insert  apps/web/config/routes.rb
      create  spec/web/controllers/books/create_spec.rb
      create  apps/web/controllers/books/create.rb
      create  apps/web/views/books/create.rb
      create  apps/web/templates/books/create.html.erb
      create  spec/web/views/books/create_spec.rb
```

```
$ git diff apps/web/config/routes.rb
diff --git a/apps/web/config/routes.rb b/apps/web/config/routes.rb
index e2ee699..b525441 100644
--- a/apps/web/config/routes.rb
+++ b/apps/web/config/routes.rb
@@ -1,3 +1,4 @@
+get '/books', to: 'books#create'
 get '/books/new', to: 'books#new'
 get '/books', to: 'books#index'
 get '/', to: 'home#index'
$
```

---

#### create アクションの実装

```
require 'spec_helper'
require_relative '../../../../apps/web/controllers/books/create'

describe Web::Controllers::Books::Create do
  let(:action) { Web::Controllers::Books::Create.new }
  let(:params) { Hash[] }

  after do
    BookRepository.clear
  end

  it 'creates a new book' do
    action.call(params)
    action.book.id.wont_be_nil
  end

  it 'redirects the user to the books listing' do
    response = action.call(params)

    response[0].must_equal 302
    response[1]['Location'].must_equal '/books'
  end
end
```

---

#### controller

```
module Web::Controllers::Books
  class Create
    include Web::Action

    expose :book

    def call(params)
      @book = BookRepository.create(Book.new(params[:book]))

      redirect_to '/books'
    end
  end
end
```

---

### バリデーション

---

#### コントローラのテスト


```
# spec/web/controllers/books/create_spec.rb
require 'spec_helper'
require_relative '../../../../apps/web/controllers/books/create'

describe Web::Controllers::Books::Create do
  let(:action) { Web::Controllers::Books::Create.new }
  let(:params) { Hash[] }

  after do
    BookRepository.clear
  end

  describe 'with valid params' do
    it 'creates a new book' do
      action.call(params)
      action.book.id.wont_be_nil
    end

    it 'redirects the user to the books listing' do
      response = action.call(params)

      response[0].must_equal 302
      response[1]['Location'].must_equal '/books'
    end
  end

  describe 'with invalid params' do
    let(:params) { Hash[book: {}] }

    it 're-renders the books#new view' do
      reponse = action.call(params)
      response[0].must_equal 200
    end
  end
end
```

---

#### コントローラ

validationがコントローラにある！

```
# apps/web/controllers/books/create.rb
module Web::Controllers::Books
  class Create
    include Web::Action

    expose :book

    params do
      param :book do
        param :title, presence: true
        param :author, presence: true
      end
    end
    def call(params)
      @book = BookRepository.create(Book.new(params[:book]))

      redirect_to '/books'
    end
  end
end
```

---

#### 
