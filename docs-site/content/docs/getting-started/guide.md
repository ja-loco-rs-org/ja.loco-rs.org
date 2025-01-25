+++
title = "The Loco Guide"
date = 2021-05-01T08:00:00+00:00
updated = 2021-05-01T08:00:00+00:00
draft = false
weight = 3
sort_by = "weight"
template = "docs/page.html"

[extra]
toc = true
top = false
flair =[]
+++

## ガイドの前提

これは「遠回り」チュートリアルです。意図的に長く詳細にしています。手動での構築方法とジェネレーターを使った自動化の両方を示し、構築スキルと仕組みを学べるようにしています。

### 名前の由来は？

`Loco`という名前は、Railsへの敬意を込めて**loco**motive（機関車）から来ています。また、`locomotive`よりも`loco`の方がタイピングしやすいです :-)。一部の言語では「狂った」という意味もありますが、それが元々の意図ではありません（RustでRailsを作るのは狂っているのか？それは時間が教えてくれるでしょう！）。

### どれくらいのRustの知識が必要ですか？

初心者から中級者レベルのRustに精通している必要があります。Rustプロジェクトのビルド、テスト、実行方法を知っており、`clap`、`regex`、`tokio`、`axum`などの人気ライブラリを使ったことがある必要があります。Locoには複雑なライフタイムやマクロはありません。

### Locoとは何ですか？

LocoはRailsに強く影響を受けています。RailsとRustの両方を知っているなら、すぐに馴染むでしょう。Railsだけを知っていてRustは初めてなら、Locoは新鮮に感じるでしょう。Railsを知っていることは前提としていません。

<div class="infobox">
Railsが素晴らしいと思っているので、このガイドも<a href="https://guides.rubyonrails.org/getting_started.html">Railsガイド</a>に強く影響を受けています。
</div>

LocoはRust用のWebまたはAPIフレームワークです。また、開発者のための生産性スイートでもあります。趣味や次のスタートアップを構築するために必要なすべてが含まれています。Railsに強く影響を受けています。

- **MVCモデルのバリアントを持っています**。オプションのパラドックスを排除します。アプリの構築に集中でき、抽象化の選択に悩むことはありません。
- **ファットモデル、スリムコントローラー**。モデルにはロジックとビジネス実装の大部分を含め、コントローラーはHTTPを理解し、パラメータを移動する軽量なルーターにします。
- **コマンドライン駆動**でモメンタムとフローを維持します。コピー＆ペーストやゼロからのコーディングよりも生成を優先します。
- **すべてのタスクが「インフラ対応」**です。コードをプラグインしてワイヤリングするだけです：コントローラー、モデル、ビュー、タスク、バックグラウンドジョブ、メーラーなど。
- **設定よりも規約**：決定はすでに行われています。フォルダ構造、設定の形状と値、アプリのワイヤリング方法がアプリの動作に影響し、最も効果的に作業できるようにします。

## 新しいLocoアプリの作成

このガイドに従ってステップバイステップで学ぶこともできますし、[ツアー](@/docs/getting-started/tour/index.md)に飛び込んで、より速い「トップダウン」の紹介を受けることもできます。

### インストール

```sh
cargo install loco
cargo install sea-orm-cli # DBが必要な場合のみ
```

### 新しいLocoアプリの作成

新しいアプリを作成できます（組み込み認証のために「SaaSアプリ」を選択してください）。

```sh
❯ loco new
✔ ❯ アプリ名は？ · myapp
✔ ❯ 何を構築しますか？ · クライアントサイドレンダリングのSaaSアプリ
✔ ❯ DBプロバイダーを選択してください · Sqlite
✔ ❯ バックグラウンドワーカータイプを選択してください · 非同期（インプロセスのtokio非同期タスク）

🚂 Locoアプリが正常に生成されました：
myapp/

- assets: アセットサービング構成として`clientside`を選択しました。

次のステップ、フロントエンドをビルドします：
  $ cd frontend/
  $ npm install && npm run build
```

Locoがデフォルトで作成するものの概要は次のとおりです：

| ファイル/フォルダ    | 目的                                                                                                                                                           |
| -------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `src/`         | コントローラー、モデル、ビュー、タスクなどを含みます                                                                                                               |
| `app.rs`       | 主要なコンポーネントの登録ポイント。重要な部分をここでワイヤリングします。                                                                                                  |
| `lib.rs`       | コンポーネントのさまざまなRust固有のエクスポート。                                                                                                                 |
| `bin/`         | `main.rs`ファイルが含まれていますが、心配する必要はありません                                                                                                         |
| `controllers/` | コントローラーが含まれ、すべてのコントローラーは`mod.rs`を介してエクスポートされます                                                                                                   |
| `models/`      | モデルが含まれ、`models/_entities`には自動生成されたSeaORMモデルが含まれ、`models/*.rs`にはモデル拡張ロジックが含まれ、`mod.rs`を介してエクスポートされます |
| `views/`       | JSONベースのビューが含まれています。構造体は`serde`を使用してAPIを通じてJSONとして出力できます。                                                                          |
| `workers/`     | バックグラウンドワーカーが含まれています。                                                                                                                                      |
| `mailers/`     | メーラーのロジックとテンプレートが含まれ、メールを送信します。                                                                                                                   |
| `fixtures/`    | データと自動フィクスチャ読み込みロジックが含まれています。                                                                                                                |
| `tasks/`       | メール送信、ビジネスレポートの作成、DBメンテナンスなどの日常的なビジネス指向のタスクが含まれています。                                         |
| `tests/`       | アプリ全体のテスト：モデル、リクエストなど。                                                                                                                       |
| `config/`      | 開発、テスト、本番のステージベースの設定フォルダ。                                                                                                 |

## こんにちは、Loco！

すぐにレスポンスを得るために、サーバーを起動する必要があります。

`myapp`に切り替えます：

```sh
$ cd myapp
```

### サーバーの起動

```sh
cargo loco start
```

そして、動作していることを確認します：

```sh
$ curl localhost:5150/_ping
{"ok":true}
```

組み込みの`_ping`ルートは、ロードバランサーにすべてが稼働していることを伝えます。

必要なすべてのサービスが稼働していることを確認します：

```sh
$ curl localhost:5150/_health
{"ok":true}
```

<div class="infobox">
組み込みの<code>_health</code>ルートは、アプリが正しく構成されていることを確認します：データベースとRedisインスタンスに正常に接続できることを示します。
</div>

### "Hello"と言ってみましょう、Loco

サービスにクイックな_hello_レスポンスを追加しましょう。

```sh
$ cargo loco generate controller guide --api
added: "src/controllers/guide.rs"
injected: "src/controllers/mod.rs"
injected: "src/app.rs"
added: "tests/requests/guide.rs"
injected: "tests/requests/mod.rs"
```

これは生成されたコントローラーの本体です：

```rust
#![allow(clippy::missing_errors_doc)]
#![allow(clippy::unnecessary_struct_initialization)]
#![allow(clippy::unused_async)]
use loco_rs::prelude::*;
use axum::debug_handler;

#[debug_handler]
pub async fn index(State(_ctx): State<AppContext>) -> Result<Response> {
    format::empty()
}

pub fn routes() -> Routes {
    Routes::new()
        .prefix("api/guides/")
        .add("/", get(index))
}
```

`index`ハンドラの本体を変更します：

```rust
// 置き換え
    format::empty()
// これに置き換え
    format::text("hello")
```

サーバーを起動します：

```sh
cargo loco start
```

そして、テストします：

```sh
$ curl localhost:5150/api/guides
hello
```

Locoには強力なジェネレーターがあり、アプリを構築する際に10倍の生産性を発揮し、モメンタムを維持します。

しばらく楽しみたい場合は、「ハードウェイ」を学び、新しいコントローラーを手動で追加してみましょう。

`home.rs`というファイルを追加し、`mod.rs`に`pub mod home;`を追加します：

```
src/
  controllers/
    auth.rs
    home.rs      <--- このファイルを追加
    users.rs
    mod.rs       <--- ここにモジュールを追加
```

次に、_hello_ルートを設定します。これは`home.rs`の内容です：

```rust
// src/controllers/home.rs
use loco_rs::prelude::*;

// _ctxにはデータベース接続や他のアプリリソースが含まれています
async fn hello(State(_ctx): State<AppContext>) -> Result<Response> {
    format::text("ola, mundo")
}

pub fn routes() -> Routes {
    Routes::new().prefix("home").add("/hello", get(hello))
}
```

最後に、この新しいコントローラールートを`app.rs`に登録します：

```rust
src/
  controllers/
  models/
  ..
  app.rs   <---- ここを見てください
```

`routes()`に次の内容を追加します：

```rust
// in src/app.rs
#[async_trait]
impl Hooks for App {
    ..
    fn routes() -> AppRoutes {
        AppRoutes::with_default_routes()
            .add_route(controllers::guide::routes())
            .add_route(controllers::auth::routes())
            .add_route(controllers::home::routes()) // <--- ここに追加
    }
```

これで完了です。サーバーを停止して再起動します：

```sh
cargo loco start
```

そして`/home/hello`にアクセスします：

```sh
$ curl localhost:5150/home/hello
ola, mundo
```

すべてのルートを確認するには次のコマンドを使用します：

```
$ cargo loco routes
  ..
  ..
[POST] /api/auth/login
[POST] /api/auth/register
[POST] /api/auth/reset
[POST] /api/auth/verify
[GET] /home/hello      <---- これは新しいルートです！
  ..
  ..
$
```

<div class="infobox">
<em>SaaSスターター</em>はクライアントサイドルーティングを使用しているため、ルートは<code>/api</code>の下にあります。<br/>
React Routerのようなクライアントサイドルーティングを使用する場合、バックエンドルートとクライアントルートを分離したいです：ブラウザは<code>/home</code>を使用しますが、<code>/api/home</code>はバックエンドルートであり、クライアントから<code>/api/home</code>を呼び出すことができます。それでも、ルート<code>/_health</code>と<code>/_ping</code>は例外で、ルートに残ります。
</div>

## MVCとあなた

**従来のMVC（モデル-ビュー-コントローラー）はデスクトップUIプログラミングのパラダイムから生まれました。** しかし、その適用性がWebサービスに広がり、急速に採用されました。MVCの黄金時代は2010年代初頭で、それ以来、多くの他のパラダイムやアーキテクチャが登場しました。

**MVCはプロジェクトを簡素化するための非常に強力な原則とアーキテクチャです**。これがLocoが従うものです。

WebサービスやAPIにはHTMLやUIレスポンスの概念がないため、**安定した、安全なサービスやAPIにはビューの概念があると主張します**。それはシリアライズされたデータ、その形状、互換性、バージョンです。

```
// 典型的なLocoアプリにはMVCのすべての部分が含まれています

src/
  controllers/
    users.rs
    mod.rs
  models/
    _entities/
      users.rs
      mod.rs
    users.rs
    mod.rs
  views/
    users.rs
    mod.rs
```

**これは重要な認知原則です**。この原則は、APIレスポンスを別個に管理されたものとして扱うことで、安全で互換性のあるレスポンスを作成できると主張します。これがLocoにおけるMVCの'V'です。

<div class="infobox">
LocoのモデルはRailsと同じ意味を持ちます：<b>ファットモデル、スリムコントローラー</b>。つまり、何かを構築したいときは<em>モデルに手を伸ばす</em>ということです。
</div>

### モデルの生成

Locoのモデルはデータ*と*機能を表します。通常、データはデータベースに保存されます。アプリケーションのほとんどのビジネスプロセスは、モデル（アクティブレコードとして）または複数のモデルのオーケストレーションとしてコーディングされます。

`Article`という新しいモデルを作成しましょう：

```sh
$ cargo loco generate model article title:string content:text

added: "migration/src/m20231202_173012_articles.rs"
injected: "migration/src/lib.rs"
injected: "migration/src/lib.rs"
added: "tests/models/articles.rs"
injected: "tests/models/mod.rs"
```

### データベースマイグレーション

**スキーマを正直に保つためにはマイグレーションを行います**。マイグレーションはデータベース構造への単一の変更です：完全なテーブル追加、変更、インデックス作成が含まれます。

```rust
// このコードは`migrations/`に生成されました：
//
// $ cargo loco generate model article title:string content:text
//
// Locoのマイグレーターフレームワークによって自動的に適用されます。
// 手動で適用することもできます：
//
// $ cargo loco db migrate
//
#[async_trait::async_trait]
impl MigrationTrait for Migration {
    async fn up(&self, manager: &SchemaManager) -> Result<(), DbErr> {
        manager
            .create_table(
                table_auto_tz(Articles::Table)
                    .col(pk_auto(Articles::Id))
                    .col(string_null(Articles::Title))
                    .col(text(Articles::Content))
                    .to_owned(),
            )
            .await
    }

    async fn down(&self, manager: &SchemaManager) -> Result<(), DbErr> {
        manager
            .drop_table(Table::drop().table(Articles::Table).to_owned())
            .await
    }
}
```

Locoのマイグレーター（SeaORMから派生）によって、**新しいデータベースにマイグレーションを連続して適用することで完全なデータベースを再作成できます**。

新しいモデルを生成すると、Locoは次のことを行います：

- 新しい「アップ」データベースマイグレーションを生成
- マイグレーションを適用
- データベース構造からエンティティを反映し、`_entities`コードを生成

新しいモデルは、データベース構造から同期されたエンティティとして`models/_entities/`に見つかります：

```
src/models/
├── _entities
│   ├── articles.rs  <-- データベーススキーマから同期、編集しないでください
│   ├── mod.rs
│   ├── prelude.rs
│   └── users.rs
├── articles.rs   <-- あなたのロジックをここに追加
├── mod.rs
└── users.rs
```

### データベースと対話するための`playground`の使用

`examples/`フォルダには次のものが含まれています：

- `playground.rs` - モデルやアプリロジックを試して実験する場所。

`playground.rs`を使用してモデルを使用してデータを取得しましょう：

以下のマークダウンを自然な日本語に翻訳しました。

```rust
// examples/playground.rs にあります
// このファイルを使って実験を行ってください
use loco_rs::{cli::playground, prelude::*};
// articles::ActiveModel を参照するには、インポートは次のようになります：
use myapp::{app::App, models::_entities::articles};

#[tokio::main]
async fn main() -> loco_rs::Result<()> {
    let ctx = playground::<App>().await?;

    // これを追加します：
    let res = articles::Entity::find().all(&ctx.db).await.unwrap();
    println!("{:?}", res);

    Ok(())
}

```

### 投稿のリストを返す

この例では、リストを返すために次のように使用します：

```rust
let res = articles::Entity::find().all(&ctx.db).await.unwrap();
```

より多くのクエリを実行する方法については、[SeaORM ドキュメント](https://www.sea-ql.org/SeaORM/docs/next/basic-crud/select/)を参照してください。

プレイグラウンドを実行するには、次のコマンドを実行します：

```rust
$ cargo playground
[]
```

次に、アイテムを1つ挿入してみましょう：

```rust
async fn main() -> loco_rs::Result<()> {
    let ctx = playground::<App>().await?;

    // これを追加します：
    let active_model: articles::ActiveModel = articles::ActiveModel {
        title: Set(Some("3ステップでアプリを構築する方法".to_string())),
        content: Set(Some("Locoを使う: https://loco.rs".to_string())),
        ..Default::default()
    };
    active_model.insert(&ctx.db).await.unwrap();

    let res = articles::Entity::find().all(&ctx.db).await.unwrap();
    println!("{:?}", res);

    Ok(())
}
```

再度プレイグラウンドを実行します：

```sh
$ cargo playground
[Model { created_at: ..., updated_at: ..., id: 1, title: Some("3ステップでアプリを構築する方法"), content: Some("Locoを使う: https://loco.rs") }]
```

これで、`articles` コントローラーに接続する準備が整いました。まず、新しいコントローラーを生成します：

```sh
$ cargo loco generate controller articles --api
added: "src/controllers/articles.rs"
injected: "src/controllers/mod.rs"
injected: "src/app.rs"
added: "tests/requests/articles.rs"
injected: "tests/requests/mod.rs"
```

`src/controllers/articles.rs` を編集します：

```rust
#![allow(clippy::unused_async)]
use loco_rs::prelude::*;

use crate::models::_entities::articles;

pub async fn list(State(ctx): State<AppContext>) -> Result<Response> {
    let res = articles::Entity::find().all(&ctx.db).await?;
    format::json(res)
}

pub fn routes() -> Routes {
    Routes::new().prefix("api/articles").add("/", get(list))
}
```

アプリを起動します：

```sh
cargo loco start
```

リクエストを送信します：

```sh
$ curl localhost:5150/api/articles
[{"created_at":"...","updated_at":"...","id":1,"title":"3ステップでアプリを構築する方法","content":"Locoを使う: https://loco.rs"}]
```

## CRUD API の構築

次に、単一の記事を取得し、削除し、編集する方法を見ていきます。IDによる記事の取得は、`axum` の `Path` エクストラクタを使用して行います。

`articles.rs` の内容を次のように置き換えます：

```rust
// src/controllers/articles.rs

#![allow(clippy::unused_async)]
use loco_rs::prelude::*;
use serde::{Deserialize, Serialize};

use crate::models::_entities::articles::{ActiveModel, Entity, Model};

#[derive(Clone, Debug, Serialize, Deserialize)]
pub struct Params {
    pub title: Option<String>,
    pub content: Option<String>,
}

impl Params {
    fn update(&self, item: &mut ActiveModel) {
        item.title = Set(self.title.clone());
        item.content = Set(self.content.clone());
    }
}

async fn load_item(ctx: &AppContext, id: i32) -> Result<Model> {
    let item = Entity::find_by_id(id).one(&ctx.db).await?;
    item.ok_or_else(|| Error::NotFound)
}

pub async fn list(State(ctx): State<AppContext>) -> Result<Response> {
    format::json(Entity::find().all(&ctx.db).await?)
}

pub async fn add(State(ctx): State<AppContext>, Json(params): Json<Params>) -> Result<Response> {
    let mut item: ActiveModel = Default::default();
    params.update(&mut item);
    let item = item.insert(&ctx.db).await?;
    format::json(item)
}

pub async fn update(
    Path(id): Path<i32>,
    State(ctx): State<AppContext>,
    Json(params): Json<Params>,
) -> Result<Response> {
    let item = load_item(&ctx, id).await?;
    let mut item = item.into_active_model();
    params.update(&mut item);
    let item = item.update(&ctx.db).await?;
    format::json(item)
}

pub async fn remove(Path(id): Path<i32>, State(ctx): State<AppContext>) -> Result<Response> {
    load_item(&ctx, id).await?.delete(&ctx.db).await?;
    format::empty()
}

pub async fn get_one(Path(id): Path<i32>, State(ctx): State<AppContext>) -> Result<Response> {
    format::json(load_item(&ctx, id).await?)
}

pub fn routes() -> Routes {
    Routes::new()
        .prefix("api/articles")
        .add("/", get(list))
        .add("/", post(add))
        .add("/{id}", get(get_one))
        .add("/{id}", delete(remove))
        .add("/{id}", patch(update))
}
```

いくつかのポイントに注意してください：

- `Params` は強い型を持つ必須のパラメータデータホルダーで、Rails の _strongparams_ に似ていますが、安全性が向上しています。
- `Path(id): Path<i32>` は URL から `:id` コンポーネントを抽出します。
- エクストラクタの順序は重要で、`axum` のドキュメントに従っています（パラメータ、状態、ボディ）。
- 単一アイテムのルートでは、`load_item` ヘルパー関数を作成し、それを使用するのが常に良いです。
- `use loco_rs::prelude::*` はコントローラーを構築するために必要なものをすべて持ち込みますが、`crate::models::_entities::articles::{ActiveModel, Entity, Model}` と `Serialize, Deserialize` をインポートすることに注意してください。

<div class="infobox">
エクストラクタの順序は重要で、順序を変更するとコンパイルエラーが発生する可能性があります。ハンドラーに <code>#[debug_handler]</code> マクロを追加すると、より良いエラーメッセージを表示するのに役立ちます。エクストラクタの詳細については、<a href="https://docs.rs/axum/latest/axum/extract/index.html#the-order-of-extractors">axum ドキュメント</a>を参照してください。
</div>

これで、アプリを起動して動作するか確認できます：

```sh
cargo loco start
```

新しい記事を追加します：

```sh
$ curl -X POST -H "Content-Type: application/json" -d '{
  "title": "あなたのタイトル",
  "content": "あなたのコンテンツ xxx"
}' localhost:5150/api/articles
{"created_at":"...","updated_at":"...","id":2,"title":"あなたのタイトル","content":"あなたのコンテンツ xxx"}
```

リストを取得します：

```sh
$ curl localhost:5150/api/articles
[{"created_at":"...","updated_at":"...","id":1,"title":"3ステップでアプリを構築する方法","content":"Locoを使う: https://loco.rs"},{"created_at":"...","updated_at":"...","id":2,"title":"あなたのタイトル","content":"あなたのコンテンツ xxx"}]
```

### 2つ目のモデルを追加する

次に、別のモデル「Comment」を追加します。コメントは投稿に属し、各投稿は複数のコメントを持つことができます。

モデルとコントローラーを手動でコーディングする代わりに、完全に動作するCRUD APIコメントを生成する**コメントスキャフォールド**を作成します。また、特別な `references` タイプも使用します：

```sh
$ cargo loco generate scaffold comment content:text article:references --api
```

<div class="infobox">
特別な <code>references:&lt;table&gt;</code> も使用可能です。異なる列名を持たせたい場合に便利です。
</div>

新しいマイグレーションを確認すると、記事テーブルに新しいデータベースリレーションが追加されていることがわかります：

```rust
      ..
      ..
  .col(integer(Comments::ArticleId))
  .foreign_key(
      ForeignKey::create()
          .name("fk-comments-articles")
          .from(Comments::Table, Comments::ArticleId)
          .to(Articles::Table, Articles::Id)
          .on_delete(ForeignKeyAction::Cascade)
          .on_update(ForeignKeyAction::Cascade),
  )
      ..
      ..
```

次に、APIを次のように変更します：

1. コメントは浅いルートを通じて追加できます： `POST comments/`
2. コメントはネストされたルートでのみ取得できます（投稿が存在することを強制）： `GET posts/1/comments`
3. コメントは更新、単体取得、削除できません。

`src/controllers/comments.rs` から不要なルートと関数を削除します：

```rust
pub fn routes() -> Routes {
    Routes::new()
        .prefix("api/comments")
        .add("/", post(add))
        // .add("/", get(list))
        // .add("/{id}", get(get_one))
        // .add("/{id}", delete(remove))
        // .add("/{id}", patch(update))
}
```

また、`src/controllers/comments.rs` の Params と update 関数を次のように調整します：

```rust
pub struct Params {
    pub content: Option<String>,
    pub article_id: i32, // <- これを追加
}

impl Params {
    fn update(&self, item: &mut ActiveModel) {
        item.content = Set(self.content.clone());
        item.article_id = Set(self.article_id.clone()); // <- これを追加
    }
}
```

次に、`src/controllers/articles.rs` でリレーションを取得する必要があります。次のルートを追加します：

```rust
pub fn routes() -> Routes {
  // ..
  // ..
  .add("/{id}/comments", get(comments))
}
```

リレーション取得を実装します：

```rust
// comments::Entity を参照するには、インポートは次のようになります：
use crate::models::_entities::{
    articles::{ActiveModel, Entity, Model},
    comments,
};

pub async fn comments(
    Path(id): Path<i32>,
    State(ctx): State<AppContext>,
) -> Result<Response> {
    let item = load_item(&ctx, id).await?;
    let comments = item.find_related(comments::Entity).all(&ctx.db).await?;
    format::json(comments)
}
```

<div class="infobox">
これは「遅延読み込み」と呼ばれ、最初にアイテムを取得し、その後に関連するリレーションを取得します。心配しないでください - 記事と一緒にコメントを事前に読み込む方法もあります。
</div>

再度アプリを起動します：

```sh
cargo loco start
```

記事 `1` にコメントを追加します：

```sh
$ curl -X POST -H "Content-Type: application/json" -d '{
  "content": "これは素晴らしい",
  "article_id": 1
}' localhost:5150/api/comments
{"created_at":"...","updated_at":"...","id":4,"content":"これは素晴らしい","article_id":1}
```

リレーションを取得します：

```sh
$ curl localhost:5150/api/articles/1/comments
[{"created_at":"...","updated_at":"...","id":4,"content":"これは素晴らしい","article_id":1}]
```

これで、包括的な _Loco ガイド_ が終了します。ここまで来たなら、おめでとうございます！

## タスク：データレポートのエクスポート

現実のアプリケーションでは、現実の状況を処理する必要があります。ユーザーや顧客の中には、何らかのレポートを要求する人もいるかもしれません。

次のようなことができます：

- 本番データベースに接続し、アドホックSQLクエリを発行します。または、何らかのDBツールを使用します。_これは安全ではなく、不正確で、エラーが発生しやすく、自動化できません_。
- データをRedshiftやGoogleなどにエクスポートし、そこでクエリを発行します。_これはリソースの無駄で、安全ではなく、正しくテストできず、遅いです_。
- 管理者を構築します。_これは時間がかかり、無駄です_。
- **または、Rustでアドホックタスクを構築します。これは迅速に記述でき、型安全で、コンパイラによって保護され、高速で、環境に依存せず、テスト可能で、安全です。**

これが `cargo loco task` の出番です。

まず、`cargo loco task` を実行して現在のタスクを確認します：

```sh
$ cargo loco task
seed_data		[データをシードするためのタスク]
```

新しいタスク `user_report` を生成します：

```sh
$ cargo loco generate task user_report

added: "src/tasks/user_report.rs"
injected: "src/tasks/mod.rs"
injected: "src/app.rs"
added: "tests/tasks/user_report.rs"
injected: "tests/tasks/mod.rs"
```

`src/tasks/user_report.rs` には生成されたタスクが表示されます。次のように置き換えます：

```rust
// `src/tasks/user_report.rs` で見つけます

use loco_rs::prelude::*;
use loco_rs::task::Vars;

use crate::models::users;

pub struct UserReport;

#[async_trait]
impl Task for UserReport {
    fn task(&self) -> TaskInfo {
      // CLI に表示される説明
        TaskInfo {
            name: "user_report".to_string(),
            detail: "ユーザーレポートを出力".to_string(),
        }
    }

    // CLI を通じての変数：
    // `$ cargo loco task name:foobar count:2`
    // `vars` で {"name":"foobar", "count":2} と表示されます
    async fn run(&self, app_context: &AppContext, vars: &Vars) -> Result<()> {
        let users = users::Entity::find().all(&app_context.db).await?;
        println!("args: {vars:?}");
        println!("!!! user_report: ユーザーのリストを表示 !!!");
        println!("------------------------");
        for user in &users {
            println!("ユーザー: {}", user.email);
        }
        println!("完了: {} ユーザー", users.len());
        Ok(())
    }
}
```

このタスクは自由に修正できます。`app_context` や他の環境リソースにアクセスし、CLI から渡された変数を `vars` で取得します。

このタスクを実行するには、次のコマンドを使用します：

```rust
$ cargo loco task user_report var1:val1 var2:val2 ...

args: Vars { cli: {"var1": "val1", "var2": "val2"} }
!!! user_report: ユーザーのリストを表示 !!!
------------------------
完了: 0 ユーザー
```
もし以前にユーザーを追加していなければ、レポートは空になります。

ユーザーを追加するには、[新しいユーザーの登録](/docs/getting-started/tour/#registering-a-new-user)をチェックしてください。

これは環境に依存するため、タスクを一度記述すれば、開発や本番環境で自由に実行できます。タスクはメインアプリのバイナリにコンパイルされます。

## 認証：リクエストの認証

`SaaS App` スターターを選択した場合、アプリには完全に構成された認証モジュールが組み込まれているはずです。
**コメントを追加する際に認証を要求する**方法を見てみましょう。

`src/controllers/comments.rs` に戻り、`add` 関数を確認します：

```rust
pub async fn add(State(ctx): State<AppContext>, Json(params): Json<Params>) -> Result<Response> {
    let mut item: ActiveModel = Default::default();
    params.update(&mut item);
    let item = item.insert(&ctx.db).await?;
    format::json(item)
}
```
認証を要求するには、関数のシグネチャを次のように修正する必要があります：

```rust
async fn add(
    auth: auth::JWT,
    State(ctx): State<AppContext>,
    Json(params): Json<Params>,
) -> Result<Response> {
    // 存在を確認するだけで十分です
    let _current_user = crate::models::users::Model::find_by_pid(&ctx.db, &auth.claims.pid).await?;

    // 次に、更新します
    // 課題/ボーナス: コメントを実際にユーザーに属させる（user_id）
    let mut item: ActiveModel = Default::default();
    params.update(&mut item);
    // item.user_id = Set(Some(_current_user.id)); // ここでユーザーIDを設定します
    let item = item.insert(&ctx.db).await?;
    format::json(item)
}
```
