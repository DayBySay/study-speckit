# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 言語設定
このリポジトリで作業する際は、**日本語でコミュニケーション**を行ってください。

## プロジェクト概要
このリポジトリは仕様策定とタスクフローを管理するためのフレームワークです。機能開発のための構造化されたワークフローを提供します。

## 主要なディレクトリ構造
- `memory/` - プロジェクトの憲法やガイドラインを含む
- `scripts/` - 開発フローを支援するBashスクリプト群
- `templates/` - 仕様書やプランのテンプレート
- `specs/` - 各機能ブランチごとの仕様書（ブランチベースで管理）

## よく使用されるコマンド

### Docker開発環境
```bash
# 全サービス起動
docker-compose up -d

# サービス状態確認
docker-compose ps

# ログ監視
docker-compose logs -f [backend|frontend|admin|mysql]

# サービス再起動
docker-compose restart [service-name]
```

### Rails (Backend API)
```bash
# Rails console
docker-compose exec backend rails console

# データベース操作
docker-compose exec backend rails db:migrate
docker-compose exec backend rails db:seed

# テスト実行
docker-compose exec backend rspec

# Rails server (通常はdocker-composeで自動起動)
docker-compose exec backend rails server
```

### React (Frontend)
```bash
# 開発サーバー (通常は自動起動)
docker-compose exec frontend npm start

# テスト実行
docker-compose exec frontend npm test

# ビルド
docker-compose exec frontend npm run build

# 依存関係インストール
docker-compose exec frontend npm install
```

### Admin Panel
```bash
# Rails console (admin)
docker-compose exec admin rails console

# Admin特有の設定確認
docker-compose exec admin rails routes | grep admin
```

## ブランチ戦略
- 機能ブランチは `001-feature-name` 形式で命名
- 各機能ブランチには `specs/[ブランチ名]/` ディレクトリに以下の標準ファイルが作成される：
  - `spec.md` - 機能仕様書
  - `plan.md` - 実装プラン
  - `tasks.md` - タスク一覧
  - `research.md` - 調査内容
  - `data-model.md` - データモデル
  - `quickstart.md` - クイックスタート
  - `contracts/` - 契約ファイル

## 重要な原則
- テンプレートベースの構造化された開発プロセス
- 仕様策定を重視したアプローチ
- ブランチごとの独立した文書管理

## 現在の技術スタック (001-ruby-on-rails ブランチ)

### Backend (Rails API) - Port: 3000
- **Ruby**: 3.2.x
- **Rails**: 7.0.x (API mode)
- **Database**: MySQL 8.0 (永続化)
- **Authentication**: セッションベース（Rails標準）
- **Testing**: RSpec, FactoryBot, Database Cleaner
- **Server**: Puma

### Frontend (React SPA) - Port: 3001
- **Node.js**: 18.x
- **React**: 18.x
- **UI Library**: ShadCN UI + Tailwind CSS
- **State Management**: React Query + Context API
- **Testing**: Jest, React Testing Library
- **Build Tool**: Vite

### Admin Panel (Rails Traditional) - Port: 3002
- **Rails**: 7.0.x (独立アプリ、full-stack mode)
- **Admin Framework**: ActiveAdmin
- **Authentication**: Devise (管理者専用認証)
- **Database**: MySQL 8.0 (Backend APIと共有)

### Infrastructure
- **Container**: Docker + Docker Compose
- **Volume Mount**: Hot Reload対応（個別マウント）
- **Database**: MySQL 8.0 (永続化、全サービス共有)
- **Development**: 完全ローカルDocker環境