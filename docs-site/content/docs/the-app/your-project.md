+++
title = "あなたのプロジェクト"
description = ""
date = 2021-05-01T18:10:00+00:00
updated = 2024-01-07T21:10:00+00:00
draft = false
weight = 2
sort_by = "weight"
template = "docs/page.html"

[extra]
lead = ""
toc = true
top = false
flair =[]
+++

## `cargo loco`による開発の推進

スターターアプリを作成します：

<!-- <snip id="loco-cli-new-from-template" inject_from="yaml" template="sh"> -->
```sh
❯ loco new
✔ ❯ App name? · myapp
✔ ❯ What would you like to build? · Saas App with client side rendering
✔ ❯ Select a DB Provider · Sqlite
✔ ❯ Select your background worker type · Async (in-process tokio async tasks)

🚂 Loco app generated successfully in:
myapp/

- assets: You've selected `clientside` for your asset serving configuration.

Next step, build your frontend:
  $ cd frontend/
  $ npm install && npm run build
```
<!-- </snip> -->

次に、アプリに移動して、さまざまなコマンドを試してみてください：

<!-- <snip id="help-command" inject_from="yaml" template="sh"> -->
```sh
cargo loco --help
```
<!-- </snip> -->

<!-- <snip id="exec-help-command" inject_from="yaml" action="exec" template="sh"> -->
```sh
The one-person framework for Rust

Usage: demo_app-cli [OPTIONS] <COMMAND>

Commands:
  start       Start an app
  db          Perform DB operations
  routes      Describe all application endpoints
  middleware  Describe all application middlewares
  task        Run a custom task
  jobs        Managing jobs queue
  scheduler   Run the scheduler
  generate    code generation creates a set of files and code templates based on a predefined set of rules
  doctor      Validate and diagnose configurations
  version     Display the app version
  watch       Watch and restart the app
  help        Print this message or the help of the given subcommand(s)

Options:
  -e, --environment <ENVIRONMENT>  Specify the environment [default: development]
  -h, --help                       Print help
  -V, --version                    Print version
```
<!-- </snip> -->

これで、CLIを通じて開発を進めることができます：

```
$ cargo loco generate model posts
$ cargo loco generate controller posts
$ cargo loco db migrate
$ cargo loco start
```

Rustを使用してテストを実行したり作業したりするのは、あなたがすでに知っている通りです：

```
$ cargo build
$ cargo test
```

### アプリの起動

アプリを実行するには、次のコマンドを実行します：

<!-- <snip id="starting-the-server-command" inject_from="yaml" template="sh"> -->
```sh
cargo loco start
```
<!-- </snip> -->

### バックグラウンドワーカー

設定（`config/`内）に基づいて、ワーカーは操作方法を知っています：

```yaml
workers:
  # Redisが必要
  mode: BackgroundQueue

  # 他にも使用可能：
  # ForegroundBlocking - テストに最適
  # BackgroundAsync - 同一プロセスのジョブを使用、tokio非同期を利用
```

そして、さまざまな方法で実際のプロセスを実行できます：

- `rr start --worker` - バックグラウンドジョブを処理するためにワーカーのみを実行します。これはスケールに最適です。1つのサービスアプリを`rr start`で実行し、任意のマシンに分散された多くのプロセスベースのワーカーを`rr start --worker`で実行します。

* `rr start --server-and-worker` - サービスとバックグラウンドワーカープロセッサの両方を同じUnixプロセスで実行します。これは、リソースが制約されている場合や単一のサーバーで実行したい場合に最適です。

### アプリのバージョンの取得

アプリがコンパイルされ、その後本番環境にコピーされるため、Locoは2つの重要な運用情報を提供します：

* このアプリのバージョンと、どのGIT SHAからビルドされたかを表示します。 `cargo loco version`
* このアプリがどのLocoバージョンに対してコンパイルされたかを表示します。 `cargo loco --version`

これらのバージョン文字列は解析可能で安定しているため、統合スクリプトや監視ツールなどで使用できます。

`src/app.rs`ファイル内の`app_version`フックをオーバーライドすることで、独自のカスタムアプリバージョン管理スキームを作成できます。

## スキャフォールドジェネレーターの使用

スキャフォールディングは、アプリケーションの主要コンポーネントを生成するための効率的で迅速な方法です。スキャフォールディングを利用することで、新しいリソースのモデル、ビュー、およびコントローラーを一度に作成できます。

スキャフォールドコマンドを参照してください：
<!-- <snip id="scaffold-help-command" inject_from="yaml" action="exec" template="sh"> -->
```sh
CRUDスキャフォールド、モデルおよびコントローラーを生成します

使用法: demo_app-cli generate scaffold [OPTIONS] <NAME> [FIELDS]...

引数:
  <NAME>       生成するものの名前
  [FIELDS]...  モデルフィールド、例： title:string hits:int

オプション:
  -k, --kind <KIND>                生成するスキャフォールドの種類 [可能な値: api, html, htmx]
      --htmx                       HTMXスキャフォールドを使用
      --html                       HTMLスキャフォールドを使用
      --api                        APIスキャフォールドを使用
  -e, --environment <ENVIRONMENT>  環境を指定 [デフォルト: development]
  -h, --help                       ヘルプを表示
  -V, --version                    バージョンを表示
```
<!-- </snip> -->

Postリソースのスキャフォールドを生成して、単一のブログ投稿を表すことができます。これを実行するには、ターミナルを開いて次のコマンドを入力します：
<!-- <snip id="scaffold-post-command" inject_from="yaml" template="sh"> -->
```sh
cargo loco generate scaffold posts name:string title:string content:text --api
```
<!-- </snip> -->

スキャフォールド生成コマンドは、`--template`フラグを追加することでAPI、HTML、またはHTMXをサポートします。

### スキャフォールドファイルのレイアウト

スキャフォールドジェネレーターは、アプリケーション内にいくつかのファイルを構築します：

| ファイル    | 目的                                                                                                                                    |
| ------------------------------------------ | ------------------------------------------------------------------------------------------------------- |
| `migration/src/lib.rs`                     | Postマイグレーションを含める。                                                                                |
| `migration/src/m20240606_102031_posts.rs`  | Postsマイグレーション。                                                                                        |
| `src/app.rs`                               | アプリケーションルーターにPostsを追加。                                                                     |
| `src/controllers/mod.rs`                   | Postsコントローラーを含める。                                                                           |
| `src/controllers/posts.rs`                 | Postsコントローラー。                                                                                   |
| `tests/requests/posts.rs`                  | 機能テスト。                                                                                     |
| `src/models/mod.rs`                        | Postsモデルを含める。                                                                                  |
| `src/models/posts.rs`                      | Postsモデル。                                                                                            |
| `src/models/_entities/mod.rs`              | Posts Sea-ormエンティティモデルを含める。                                                                    |
| `src/models/_entities/posts.rs`            | Sea-ormエンティティモデル。                                                                                   |
| `src/views/mod.rs`                         | Postsビューを含める。HTMLおよびHTMXテンプレート専用。                                                |
| `src/views/posts.rs`                       | Postsテンプレートジェネレーター。HTMLおよびHTMXテンプレート専用。                                             |
| `assets/views/posts/create.html`           | 投稿作成テンプレート。HTMLおよびHTMXテンプレート専用。                                                 |
| `assets/views/posts/edit.html`             | 投稿編集テンプレート。HTMLおよびHTMXテンプレート専用。                                                   |
| `assets/views/posts/list.html`             | 投稿リストテンプレート。HTMLおよびHTMXテンプレート専用。                                                   |
| `assets/views/posts/show.html`             | 投稿表示テンプレート。HTMLおよびHTMXテンプレート専用。                                                   |

## アプリの設定
デフォルトでは、locoはその設定ファイルを`config/`ディレクトリに保存します。3つの環境に対して事前定義された設定を提供します：

```
config/
  development.yaml
  production.yaml
  test.yaml
```

環境は自動的に次の方法で選択されます：

- コマンドラインフラグ： `cargo loco start --environment production`、指定されていない場合は
- `LOCO_ENV`または`RAILS_ENV`または`NODE_ENV`

何も指定されていない場合、デフォルト値は`development`です。

`Loco`フレームワークは、デフォルトの環境に加えてカスタム環境のサポートを提供します。カスタム環境を追加するには、前述の例で使用される環境識別子に一致する名前の設定ファイルを作成します。

### デフォルト設定パスのオーバーライド
カスタム設定ディレクトリを使用するには、`LOCO_CONFIG_FOLDER`環境変数を希望のフォルダパスに設定します。これにより、locoはデフォルトの`config/`フォルダではなく、指定されたフォルダから設定ファイルを読み込みます。

### 設定ファイル内のプレースホルダー/変数

設定ファイルに値を注入することが可能です。この例では、`NODE_PORT`環境変数からポート値を取得します：

```yaml
# config/development.yaml
# すべての設定ファイルは有効なTeraテンプレートです
server:
  # サーバーがリッスンするポート。サーバーバインディングは0.0.0.0:{PORT}
  port:  {{/* get_env(name="NODE_PORT", default=5150) */}}
  # メーラーが指すUIホスト名またはIPアドレス。
  host: http://localhost
  # デフォルトのミドルウェア構成。ミドルウェアを無効にするには、`enable`フィールドを`false`に変更するか、ミドルウェアブロックをコメントアウトします。
```

[get_env](https://keats.github.io/tera/docs/#get-env)関数はTeraテンプレートエンジンの一部です。詳細については、[Tera](https://keats.github.io/tera/docs)のドキュメントを参照してください。

### 例

'qa'環境を追加したいとします。`config`フォルダに`qa.yaml`ファイルを作成します：

```
config/
  development.yaml
  production.yaml
  test.yaml
  qa.yaml
```

アプリケーションを'qa'環境で実行するには、次のコマンドを実行します：
<!-- <snip id="starting-the-server-command-with-environment-env-var" inject_from="yaml" template="sh"> -->
```sh
LOCO_ENV=qa cargo loco start
```
<!-- </snip> -->

### 設定

設定ファイルには、Locoアプリを設定するためのノブが含まれています。また、`settings:`セクションを使用して独自の設定を持つこともできます。`config/development.yaml`に`settings:`セクションを追加します：
<!-- <snip id="configuration-settings" inject_from="code" template="yaml"> -->
```yaml
settings:
  allow_list:
    - google.com
    - apple.com
```
<!-- </snip> -->

これらの設定は、`ctx.config.settings`内の`serde_json::Value`として表示されます。構造体を追加することで、強く型付けされた設定を作成できます：

```rust
// これをsrc/common/settings.rsに置く
#[derive(Serialize, Deserialize, Default, Debug)]
pub struct Settings {
    pub allow_list: Option<Vec<String>>,
}

impl Settings {
    pub fn from_json(value: &serde_json::Value) -> Result<Self> {
        Ok(serde_json::from_value(value.clone())?)
    }
}
```

次に、次のようにしてどこからでも設定にアクセスできます：

```rust
// コントローラー、ワーカー、タスク、または他の場所で、
// AppContext（ここでは`ctx`）にアクセスできる限り

if let Some(settings) = &ctx.config.settings {
    let settings = common::settings::Settings::from_json(settings)?;
    println!("許可リスト: {:?}", settings.allow_list);
}
```

### サーバー

ここでは、`server:`の下にあるインターフェース（リッスンなど）のパラメーターについての詳細な説明があります：

* `port:` 名前が示す通り、ポートを変更するためのものです。主にロードバランサーの背後にいる場合などです。

* `binding:` IPインターフェースが「バインド」する場所を変更するためのものです。主に、`nginx`のようなロードバランサーの背後にいる場合は、ローカルアドレスにバインドします（LBもそこにある場合）。ただし、「ワールド」（`0.0.0.0`）にバインドすることもできます。設定は、CLI（`-b`フラグを使用）経由で行うことができます。これはRailsが行っていることです。

* `host:` - 「可視性」使用ケースやアウトオブバンド使用ケースのためです。たとえば、現在のサーバーホスト（ドメイン名など）の表示に使用したい場合や、メールのように、サーバーアドレスが「アウトオブバンド」である場合があります。これは、Gmailアカウントを開いたときに、あなたのメールをクリックすると、外部アドレスや可視アドレス（公式ドメイン名など）を指す必要があるためです。

### ロガー

YAMLファイルの`logger:`セクションのコメント付きフィールド以外に、以下のようなコンテキストがあります：

* `logger.pretty_backtrace` - ノイズのないカラフルなバックトレースを表示し、素晴らしい開発体験を提供します。これは、特定のエラーでバックトレースのキャプチャを有効にするためにプロセスの環境に`RUST_BACKTRACE=1`を強制的に設定します。開発中に有効にし、本番環境では無効にします。本番環境で必要な場合は、コマンドラインで`RUST_BACKTRACE=1`を使用して、特定のエラーのバックトレースを表示します。

すべての利用可能な設定オプションについては、[こちらをクリック](https://docs.rs/loco-rs/latest/loco_rs/config/struct.Config.html)してください。
