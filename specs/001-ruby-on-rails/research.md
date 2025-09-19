# Research Findings: ECサイトプラットフォーム

**Date**: 2025-09-19  
**Context**: Ruby on Rails + React + MySQL + Docker構成での学習用ECサイト開発

## 研究課題と解決策

### 1. テスティングフレームワーク

**Decision**: RSpec (Rails), Jest + React Testing Library (React)  
**Rationale**: 
- RSpec: Rails標準、豊富なmatcher、行動駆動開発に適している
- Jest: React標準、snapshot testing、優れたmocking機能
- React Testing Library: ユーザー行動ベースのテスト、メンテナンス性が高い

**Alternatives considered**: 
- Minitest (Rails): シンプルだが、RSpecほど表現力が高くない
- Mocha/Chai (React): 設定が複雑、Jestの方が統合的

### 2. パフォーマンス目標

**Decision**: ローカル開発での快適な応答性（<1秒 for typical operations）  
**Rationale**: 
- 学習用途のため過度な最適化は不要
- Dockerコンテナでの動作を考慮して現実的な目標設定
- Rails開発サーバー + React開発サーバーでのホットリロード対応

**Alternatives considered**: 
- プロダクション並みの性能目標: 学習用途に過剰
- 性能測定なし: 基本的な指標は必要

### 3. 管理画面技術スタック

**Decision**: ActiveAdmin（独立Railsアプリ、共有MySQL）  
**Rationale**: 
- Rails標準パターン、学習コストが低い
- 管理機能に特化した豊富な機能
- 顧客向けAPIと管理機能を分離、保守性向上
- 権限管理機能が組み込み済み
- 同一DBを参照するため整合性確保

**Alternatives considered**: 
- 同一アプリ内管理機能: API設計の複雑化
- React Admin: 開発コストが高い、学習用途に過剰
- 完全独立DB: データ整合性確保が複雑

### 4. Docker構成パターン

**Decision**: Docker Compose with separate containers (rails-api, react-dev, mysql, admin)  
**Rationale**: 
- 開発時のhot reloadingが可能
- 各サービスの独立性確保
- 本番環境への移行が容易
- ローカル環境の再現性が高い

**Alternatives considered**: 
- Single container: 複雑すぎ、デバッグしにくい
- No Docker: 環境依存性の問題

### 5. API設計パターン

**Decision**: REST API + JSON:API仕様準拠  
**Rationale**: 
- Rails標準、Active Model Serializers対応
- フロントエンド-バックエンド分離に最適
- APIドキュメントの自動生成が可能
- 標準的な慣習でメンテナンス性が高い

**Alternatives considered**: 
- GraphQL: 学習コスト高、過剰な複雑性
- 独自JSON形式: 標準性に欠ける

### 6. UI Library選択

**Decision**: ShadCN UI  
**Rationale**: 
- モダンなコンポーネントライブラリ
- Tailwind CSS ベース、カスタマイズ性が高い
- TypeScriptサポート良好
- 学習用途に適した中程度の複雑性

**Alternatives considered**: 
- Material-UI: 機能豊富だが学習コスト高
- Bootstrap: クラシックだが見た目が古い
- Tailwind CSS単体: コンポーネント作成コスト高

### 7. 認証方式選択

**Decision**: セッションベース認証  
**Rationale**: 
- Rails標準、実装とデバッグが簡単
- サーバーサイドで完全制御可能
- CSRFトークンとの自然な連携
- ゲストカート機能も session_id で統一実装
- 学習用途として理解しやすい

**Alternatives considered**: 
- JWT: ステートレスだがセキュリティ管理複雑
- JWT + Refresh Token: 最もセキュアだが実装複雑度高、学習用途に過剰

### 8. Docker開発環境構成

**Decision**: Hot Reload + MySQL永続化  
**Rationale**: 
- 各サービス個別volume mountで効率的開発
- コード変更時の自動リロード対応
- MySQLデータ永続化で開発継続性確保
- 本番環境に近い構成で学習効果向上

**Alternatives considered**: 
- Shared Volume: 依存関係分離が曖昧
- データ非永続化: 開発中断時のデータ喪失リスク

## 技術スタック確定版

### Backend (Rails API)
- **Ruby**: 3.2.x  
- **Rails**: 7.0.x (API mode)  
- **Database**: MySQL 8.0  
- **Authentication**: セッションベース（Rails標準）  
- **Serialization**: Active Model Serializers (JSON:API)  
- **Testing**: RSpec, FactoryBot, Database Cleaner  
- **Server**: Puma  

### Frontend (React SPA)  
- **Node.js**: 18.x  
- **React**: 18.x  
- **State Management**: React Query + Context API  
- **Routing**: React Router v6  
- **UI Library**: ShadCN UI + Tailwind CSS  
- **HTTP Client**: Axios  
- **Testing**: Jest, React Testing Library  
- **Build Tool**: Vite  

### Admin Panel (Rails Traditional)
- **Rails**: 7.0.x (full-stack mode, 独立アプリ)  
- **Admin Framework**: ActiveAdmin  
- **Authentication**: Devise (管理者専用認証)  
- **UI**: ActiveAdmin default + custom CSS  
- **Port**: 3002（顧客向けAPIと分離）

### Infrastructure
- **Container**: Docker + Docker Compose  
- **Database**: MySQL 8.0 (永続化、全アプリで共有)  
- **Volume Mount**: Hot Reload対応（個別マウント）  
- **Development**: 完全ローカルDocker環境  
- **Asset Storage**: Local files (学習用途)  

## 残課題
なし - すべての NEEDS CLARIFICATION が解決済み