# Deployment Guide

Vibebaseを自分のCloudflareアカウントにデプロイする方法です。

## 📋 前提条件

1. **Cloudflareアカウント** - [cloudflare.com](https://cloudflare.com)で無料アカウントを作成
2. **Node.js** - v18以上をインストール
3. **Git** - このリポジトリをクローンするため

## 🚀 ワンクリックデプロイ

### 方法1: 自動セットアップスクリプト（推奨）

```bash
# 1. リポジトリをクローン
git clone https://github.com/meso/vibebase.git
cd vibebase

# 2. Wrangler CLIをインストール（まだの場合）
npm install -g wrangler

# 3. Cloudflareにログイン
wrangler login

# 4. 自動セットアップを実行
chmod +x deploy/setup.sh
./deploy/setup.sh
```

これで完了！🎉

### 方法2: 手動セットアップ

詳細な制御が必要な場合は手動でセットアップできます：

```bash
# 1. 依存関係をインストール
cd packages/core
npm install

# 2. D1データベースを作成
wrangler d1 create vibebase-db

# 3. wrangler.tomlのdatabase_idを更新（上記コマンドの出力から）

# 4. データベーススキーマを作成
wrangler d1 execute vibebase-db --file=migrations/0001_initial.sql --remote

# 5. ダッシュボードをビルド
cd ../dashboard
npm install
npm run build

# 6. デプロイ
cd ../core
wrangler deploy
```

## 🔧 カスタマイズ

デプロイ後、以下をカスタマイズできます：

### 1. Worker名の変更
`packages/core/wrangler.toml`の`name`を変更

### 2. カスタムドメインの設定
Cloudflare Dashboardで「Workers & Pages」→ 「Custom domains」

### 3. 環境変数の設定

#### JWT_SECRET（必須：本番環境）
ユーザー認証用のJWTトークン署名に使用される秘密鍵です。

```bash
# 本番環境用のセキュアなJWT秘密鍵を設定
wrangler secret put JWT_SECRET
```

**重要：**
- 本番環境では必ずこの設定が必要です
- 開発環境では自動的に安全な一時キーが生成されます
- 32文字以上のランダムな文字列を使用してください
- 以下のコマンドで安全な秘密鍵を生成できます：

```bash
# 安全なJWT秘密鍵を生成（Linux/macOS）
openssl rand -base64 32

# または、Nodeで生成
node -e "console.log(require('crypto').randomBytes(32).toString('base64'))"
```

#### その他のシークレット
```bash
# 必要に応じて他のシークレットも設定
wrangler secret put SECRET_NAME
```

### 4. R2ストレージの有効化
```bash
# R2バケットを作成
wrangler r2 bucket create vibebase-storage

# wrangler.tomlのR2セクションをアンコメント
```

### 5. KVストレージの有効化（セッション用）
```bash
# KV namespaceを作成
wrangler kv:namespace create "SESSIONS"

# wrangler.tomlのKVセクションをアンコメント
```

## 📖 API使用方法

デプロイ完了後、以下のAPIが利用可能になります：

### ヘルスチェック
```bash
curl https://your-worker.your-subdomain.workers.dev/api/health
```

### アイテム管理
```bash
# アイテム作成
curl -X POST https://your-worker.your-subdomain.workers.dev/api/items \
  -H "Content-Type: application/json" \
  -d '{"name": "Test Item", "description": "A test item"}'

# アイテム一覧
curl https://your-worker.your-subdomain.workers.dev/api/items

# アイテム取得
curl https://your-worker.your-subdomain.workers.dev/api/items/[ID]

# アイテム更新
curl -X PUT https://your-worker.your-subdomain.workers.dev/api/items/[ID] \
  -H "Content-Type: application/json" \
  -d '{"name": "Updated Item"}'

# アイテム削除
curl -X DELETE https://your-worker.your-subdomain.workers.dev/api/items/[ID]
```

### プロジェクト管理
```bash
# プロジェクト作成
curl -X POST https://your-worker.your-subdomain.workers.dev/api/projects \
  -H "Content-Type: application/json" \
  -d '{"name": "My Project", "description": "A test project"}'

# プロジェクト一覧
curl https://your-worker.your-subdomain.workers.dev/api/projects
```

## 🛠️ ローカル開発

```bash
# 開発サーバーを起動
cd packages/core
wrangler dev

# 別のターミナルでダッシュボード開発サーバー
cd packages/dashboard
npm run dev
```

## 🔍 トラブルシューティング

### Wranglerログインエラー
```bash
wrangler logout
wrangler login
```

### データベース接続エラー
- D1データベースが作成されているか確認
- wrangler.tomlのdatabase_idが正しいか確認

### デプロイエラー
```bash
# ログを確認
wrangler tail

# 詳細なエラー情報を表示
wrangler deploy --verbose
```

## 📚 次のステップ

1. [認証機能の設定](./docs/authentication.md)
2. [R2ストレージの設定](./docs/storage.md)
3. [リアルタイム機能の実装](./docs/realtime.md)
4. [カスタムドメインの設定](./docs/custom-domain.md)

## 💬 サポート

問題が発生した場合は、[Issues](https://github.com/meso/vibebase/issues)でお知らせください。