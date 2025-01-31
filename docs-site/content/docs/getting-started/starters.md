+++
title = "スターター"
date = 2021-12-19T08:00:00+00:00
updated = 2021-12-19T08:00:00+00:00
draft = false
weight = 4
sort_by = "weight"
template = "docs/page.html"

[extra]
toc = true
top = false
flair =[]
+++

Locoの事前定義されたボイラープレートを使用して、プロジェクトのセットアップを簡素化しましょう。これにより、開発の旅がスムーズになります。まずは、CLIをインストールして、ニーズに合ったテンプレートを選択してください。

<!-- <snip id="quick-installation-command" inject_from="yaml" template="sh"> -->
```sh
cargo install loco
cargo install sea-orm-cli # データベースが必要な場合のみ
```
<!-- </snip> -->

スターターを作成します：

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

次のステップは、フロントエンドをビルドすることです：
  $ cd frontend/
  $ npm install && npm run build
```
<!-- </snip> -->

## 利用可能なスターター

### SaaSスターター

SaaSスターターは、UIとREST APIの両方を必要とするプロジェクトのための包括的なセットアップです。このスターターは、クライアントサイドアプリまたはクラシックなサーバーサイドテンプレート（またはその組み合わせ）をサポートしています。

**UI**

- ReactとViteに基づいたフロントエンドスターター（お好みのフレームワークに簡単に置き換え可能）。
- フロントエンドビルドを指し示す静的ミドルウェアと、フォールバックインデックスを含む。あるいは、サーバーサイドテンプレート用の静的アセットに構成できます。
- サーバーサイドテンプレート用に設定されたTeraビューエンジン、i18n構成を含む。テンプレートとi18nアセットは`assets/`にあります。

**REST API**

- サービスの健康状態を確認するための`ping`および`health`エンドポイント。すべてのエンドポイントは次のコマンドで確認できます `cargo loco routes`
- ユーザーテーブルと認証ミドルウェア。
- 認証ロジックとユーザー登録を含むユーザーモデル。
- パスワードを忘れた場合のAPIフロー。
- ウェルカムメールを送信し、パスワードを忘れたリクエストを処理するメール送信機能。

#### サーバーサイドテンプレートのためのアセットの構成

SaaSスターターは、フロントエンドクライアントサイドアセット用に事前構成されています。サーバーサイドテンプレートレンダリングを使用し、画像やスタイルなどのアセットを含めたい場合は、アセットミドルウェアを以下のように構成できます：

`config/development.yaml`でサーバーサイド構成のコメントを外し、クライアントサイド構成のコメントを入れます。

```yaml
    # サーバーサイド静的アセット構成
    # initializers/view_engine.rsのview_engineと一緒に使用
    #
    static:
      enable: true
      must_exist: true
      precompressed: false
      folder:
        uri: "/static"
        path: "assets/static"
      fallback: "assets/static/404.html"
    fallback:
      enable: false
    # client side app static config
    # static:
    #   enable: true
    #   must_exist: true
    #   precompressed: false
    #   folder:
    #     uri: "/"
    #     path: "frontend/dist"
    #   fallback: "frontend/dist/index.html"
    # fallback:
    #   enable: false
```

### REST APIスターター

フロントエンドなしでREST APIのみが必要な場合は、REST APIスターターを選択してください。後でフロントエンドを提供することに決めた場合は、`static`ミドルウェアを有効にし、構成を`frontend`の配布フォルダーにポイントするだけです。

### 軽量サービススターター

コントローラーとビュー（レスポンススキーマ）に焦点を当てた軽量サービススターターは、最小限の構成です。データベース、フロントエンド、ワーカー、またはLocoが提供する他の機能なしでREST APIサービスが必要な場合は、これが理想的な選択です！

