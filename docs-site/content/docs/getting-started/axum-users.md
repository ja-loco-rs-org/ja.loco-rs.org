+++
title = "Axum vs Loco"
description = "Shows how to move from Axum to Loco"
date = 2023-12-01T19:30:00+00:00
updated = 2023-12-01T19:30:00+00:00
draft = false
weight = 5
sort_by = "weight"
template = "docs/page.html"

[extra]
toc = true
top = false
flair =[]
+++

<div class="infobox">
<b>注意: LocoはAxumgがベースになっており、「バッテリー付きのAxum」です</b>。AxumのコードをLocoに移行するのは非常に簡単です。
</div>

私たちは、[realworld-axum-sqlx](https://github.com/launchbadge/realworld-axum-sqlx) を研究します。これは、API、実際のデータベース、実際のシナリオ、および構成やログ記録などの実際の運用要件を使用して、実際のプロジェクトを説明しようとするAxumベースのアプリです。

`realworld-axum-sqlx` を部分ごとに分解して、**AxumからLocoに移行することで、ほとんどのコードがすでに書かれていることを示します**。より良いベストプラクティス、より良い開発体験、統合テスト、コード生成、およびアプリの迅速な構築が可能です。

**この内訳を使用して**、自分のAxumベースのアプリをLocoに移行する方法を理解できます。質問がある場合は、[ディスカッション](https://github.com/loco-rs/loco/discussions) でお問い合わせいただくか、[discordの緑の招待ボタンをクリックして参加](https://github.com/loco-rs/loco) してください。

## `main`

Axumを使用する場合、アプリのすべてのコンポーネントを設定し、ルーターを取得し、ミドルウェアを追加し、コンテキストを設定し、最終的にソケットで `listen` を設定する独自の `main` 関数が必要です。

これは多くの手動でエラーが発生しやすい作業です。

Locoでは次のことを行います：

* 設定で必要なミドルウェアをオン/オフに切り替える
* `cargo loco start` を使用し、`main` ファイルは不要
* 本番環境では、`your_app` という名前のコンパイル済みバイナリを実行する

### Locoへの移行

* Locoの `config/` で必要なミドルウェアを設定する

```yaml
server:
  middlewares:
    limit_payload:
      body_limit: 5mb
  # .. more middleware below ..
```

* Locoの `config/` でサービングポートを設定する

```yaml
server:
  port: 5150
```

### 評価

* **コードを書く必要がない** - `main` 関数を手動で書く必要がない
* **ベストプラクティスがすぐに利用可能** - すべてのLocoアプリで共有されるベストプラクティスの `main` ファイルが提供される
* **変更が簡単** - ミドルウェアを追加/削除してテストする場合、設定でスイッチを切り替えるだけで再ビルドは不要

## 環境

realworld-axumコードベースは、[dotenv](https://github.com/launchbadge/realworld-axum-sqlx/blob/main/.env.sample) を使用しており、`main` で明示的に読み込む必要があります：

```rust
 dotenv::dotenv().ok();
```

そして、`.env` ファイルが利用可能で、維持され、読み込まれる必要があります：

```
DATABASE_URL=postgresql://postgres:{password}@localhost/realworld_axum_sqlx
HMAC_KEY={random-string}
RUST_LOG=realworld_axum_sqlx=debug,tower_http=debug
```

これはプロジェクトと一緒に提供される**サンプル**ファイルであり、手動でコピーして編集する必要があり、非常にエラーが発生しやすいです。

### Locoへの移行

Loco：標準の `config/[stage].yaml` 設定を使用し、`get_env` を使用して環境から特定の値を読み込みます

```yaml
# config/development.yaml

# Web server configuration
server:
  # Port on which the server will listen. the server binding is 0.0.0.0:{PORT}
  port: {{% get_env(name="NODE_PORT", default=5150) %}}
```

この設定は強く型付けされており、データベースURL、ロガーレベルとフィルタリングなどの最も使用される値を含んでいます。推測や再発明の必要はありません。

### 評価

* **コードを書く必要がない** - Locoに移行する際に書くコードが少ない
* **移動部品が少ない** - Axumのみを使用する場合、環境変数に加えて設定が必要ですが、Locoではこれが無料で提供されます

## データベース

Axumのみを使用する場合、通常は接続、プールを設定し、ルートで利用できるように設定する必要があります。これが通常 `main.rs` に記述されるコードです：

```rust
    let db = PgPoolOptions::new()
        .max_connections(50)
        .connect(&config.database_url)
        .await
        .context("could not connect to database_url")?;
```

次に、この接続を手動で配線する必要があります：

```rust
 .layer(AddExtensionLayer::new(ApiContext {
                config: Arc::new(config),
                db,
            }))
```

### Locoへの移行

Locoでは、`config/` フォルダーにプールの値を設定するだけです。すでにベストエフォートのデフォルト値を取得しているため、設定する必要はありませんが、設定したい場合は次のようになります：

```yaml
database:
  enable_logging: false
  connect_timeout: 500
  idle_timeout: 500
  min_connections: 1
  max_connections: 1
```

### 評価

* **コードを書く必要がない** - データベースプールの適切な値を選択したり、誤設定する危険を回避
* **変更が簡単** - 本番環境で異なる負荷の下で異なる値を試したい場合、Axumのみを使用する場合は再コンパイル、再デプロイが必要です。Locoでは設定を変更してプロセスを再起動するだけです。

## ロギング

アプリ全体で手動でロギングストーリーをコーディングする必要があります。どれを選びますか？`tracing` か `slog` か？ロギングかトレースか？どちらが良いですか？

real-world-axumプロジェクトに存在するものは次のとおりです。サービングで：

```rust
  // Enables logging. Use `RUST_LOG=tower_http=debug`
  .layer(TraceLayer::new_for_http()),
```

そして `main` で：

```rust
    // Initialize the logger.
    env_logger::init();
```

そして、さまざまなポイントでのアドホックロギング：

```rust
  log::error!("SQLx error: {:?}", e);
```

### Locoへの移行

Locoでは、これらの難しい質問にすでに答えており、マルチティアのロギングとトレースを提供しています：

* フレームワーク内部で
* ルーターで設定
* 低レベルのDBロギングとトレース
* タスク、バックグラウンドジョブなどのLocoのすべてのコンポーネントが同じ施設を使用

そして、すべてのRustライブラリが一様にログに「ストリーム」できるように `tracing` を選びました。

しかし、デフォルトで知らないライブラリに爆撃されないようにスマートフィルターも作成しました。

ロガーを `config/` で設定できます：

```yaml
logger:
  enable: true
  pretty_backtrace: true
  level: debug
  format: compact
```

### 評価

* **コードを書く必要がない** - 設定コードも決定も不要です。最適な決定を下したので、アプリのコードを書くことに集中できます。
* **迅速な構築** - 必要なものだけのトレースを取得します。エラーバックトレースはカラフルで文脈的でノイズがゼロなので、デバッグが容易です。フォーマットやレベルを変更して本番環境に適用できます。
* **変更が簡単** - 本番環境で異なる負荷の下で異なる値を試したい場合、Axumのみを使用する場合は再コンパイル、再デプロイが必要です。Locoでは設定を変更してプロセスを再起動するだけです。

## ルーティング

AxumからLocoへのルートの移行は実際にはドロップインです。LocoはネイティブのAxumルーターを使用します。

ルートリストや情報などの機能が必要な場合は、ネイティブのLocoルーターを使用できます。これはAxumルーターに変換されるか、独自のAxumルーターを使用できます。

### Locoへの移行

1:1の完全なコピーペースト体験を望む場合は、Axumルートをコピーして、Locoの `after_routes()` フックにプラグインします：

```rust
  async fn after_routes(router: AxumRouter, _ctx: &AppContext) -> Result<AxumRouter> {
      // use AxumRouter to mount your routes and return an AxumRouter
  }
```

Locoがルートに関するメタデータ情報を理解することを望む場合（後で役立つことがあります）、各コントローラーで `routes()` 関数を次のように記述します：

```rust
// this is what people usually do using Axum only
pub fn router() -> Router {
  Router::new()
        .route("/auth/register", post(create_user))
        .route("/auth/login", post(login_user))
}

// this is how it looks like using Loco (notice we use `Routes` and `add`)
pub fn routes() -> Routes {
  Routes::new()
      .add("/auth/register", post(create_user))
