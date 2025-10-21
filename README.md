# Go Temporal Tutorial

このプロジェクトは、Go言語でTemporalを使用するためのチュートリアルです。Docker Composeを使用してTemporal環境を簡単にセットアップできます。

チュートリアルは、[Go Temporalの公式ページ](https://learn.temporal.io/getting_started/go/dev_environment/)を見てください。

## 必要な環境

- Docker
- Docker Compose
- Go 1.19以上（Goアプリケーション開発用）

## セットアップ

### 1. Dockerコンテナの起動

```bash
docker-compose up -d
```

このコマンドで以下のサービスが起動されます：

#### サービス構成

- **PostgreSQL** (`temporal-tutorial-postgres`)
  - ポート: 15432 (ホスト) → 5432 (コンテナ)
  - データベース: `temporal`
  - ユーザー: `temporal`
  - パスワード: `temporal`
  - Temporalのメタデータストレージとして使用

- **Temporal Server** (`temporal-tutorial`)
  - ポート: 7233
  - PostgreSQLに依存
  - temporalio/auto-setup:latest を使用
  - 自動セットアップ機能付き

- **Temporal Web UI** (`temporal-web-tutorial`)
  - ポート: 8088 (ホスト) → 8080 (コンテナ)
  - ブラウザからアクセス可能: http://localhost:8088
  - ワークフローとアクティビティの監視が可能
  - CORS設定済み

### 2. サービスの確認

全てのサービスが正常に起動しているか確認：

```bash
docker-compose ps
```

ログの確認：

```bash
# 全サービスのログ
docker-compose logs

# 特定サービスのログ
docker-compose logs temporal
docker-compose logs postgresql
```

### 3. Temporal Web UIへのアクセス

ブラウザで http://localhost:8080 にアクセスして、Temporal Web UIを確認できます。

## 開発時の使用方法

### Goアプリケーションの実行

Temporalサーバーが起動している状態で、Goアプリケーションを実行：

```bash
# ワーカーの起動
go run worker/main.go

# クライアントの実行
go run client/main.go
```

### 接続設定

GoアプリケーションからTemporalに接続する際の設定：

```go
import (
    "go.temporal.io/sdk/client"
)

// Temporal クライアントの作成
c, err := client.Dial(client.Options{
    HostPort: "localhost:7233", // docker-compose.ymlで公開されているポート
})
```

## 便利なコマンド

### コンテナの管理

```bash
# サービスの起動
docker-compose up -d

# サービスの停止
docker-compose down

# サービスの再起動
docker-compose restart

# ログの監視
docker-compose logs -f
```

### Admin Toolsの使用

```bash
# Admin toolsコンテナに接続
docker-compose exec temporal-admin-tools bash

# コンテナ内でtctl（Temporal CLI）を実行
tctl namespace list
tctl workflow list
```

### データベースの管理

```bash
# PostgreSQLに直接接続
docker-compose exec postgresql psql -U temporal

# データベースの初期化（必要に応じて）
docker-compose down -v  # ボリュームも削除
docker-compose up -d
```

## トラブルシューティング

### よくある問題

1. **ポートが既に使用されている**
   ```bash
   # 使用中のポートを確認
   lsof -i :7233
   lsof -i :8080
   lsof -i :5432
   ```

2. **サービスが起動しない**
   ```bash
   # 詳細なログを確認
   docker-compose logs temporal
   docker-compose logs postgresql
   ```

3. **データベース接続エラー**
   - PostgreSQLコンテナが完全に起動するまで待つ
   - 環境変数が正しく設定されているか確認

### クリーンアップ

全てのデータを削除して初期状態に戻す：

```bash
docker-compose down -v --remove-orphans
docker-compose up -d
```

## 参考リンク

- [Temporal Documentation](https://docs.temporal.io/)
- [Temporal Go SDK](https://docs.temporal.io/dev-guide/go)
- [Docker Compose Documentation](https://docs.docker.com/compose/)