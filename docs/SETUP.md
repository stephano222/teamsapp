# 開発環境のセットアップ

## 動作環境

- Ruby 3.4.10（`.ruby-version` で固定）
- Rails 8.1
- PostgreSQL 16（Docker で起動）

## 必要なもの

- [rbenv](https://github.com/rbenv/rbenv)
- [Docker Desktop](https://www.docker.com/products/docker-desktop/)
- libpq（`pg` gem のビルドに必要）

## セットアップ

```bash
# Ruby
rbenv install 3.4.10

# pg gem のビルド用に libpq を入れる（macOS）
brew install libpq
bundle config build.pg --with-pg-config=$(brew --prefix libpq)/bin/pg_config

# DB（PostgreSQL）を起動
docker compose up -d

# gem のインストール・DB 作成・サーバー起動
bin/setup
```

http://localhost:3000 で確認できます。

## よく使うコマンド

| やること | コマンド |
| --- | --- |
| サーバー起動 | `bin/dev` |
| DB 起動 / 停止 | `docker compose up -d` / `docker compose down` |
| マイグレーション | `bin/rails db:migrate` |
| テスト | `bin/rails test` |
| Lint | `bin/rubocop` |

## DB 接続設定

開発・テスト環境は `config/database.yml` で以下の環境変数を参照します（未設定ならデフォルト値）。
5432 番ポートが他で使われている場合は `DB_PORT` を変えてください。

| 変数 | デフォルト |
| --- | --- |
| `DB_HOST` | `localhost` |
| `DB_PORT` | `5432` |
| `DB_USERNAME` | `postgres` |
| `DB_PASSWORD` | `password` |

## ブランチ運用

- `main`: リリース用
- `develop`: 開発の統合ブランチ
- 作業は `develop` から `feature/xxx` などを切り、`develop` へ PR を出す
