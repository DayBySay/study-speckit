# Quickstart Guide: ECサイトプラットフォーム

**Date**: 2025-09-19  
**Purpose**: 開発環境セットアップと主要機能の動作確認

## Prerequisites

- Docker Desktop installed and running
- Git installed  
- Text editor (VS Code recommended)

## Environment Setup

### 1. Repository Clone and Setup
```bash
# Clone the repository
git clone <repository-url>
cd ecommerce-platform

# Create environment files
cp backend/.env.example backend/.env
cp frontend/.env.example frontend/.env
cp admin/.env.example admin/.env
```

### 2. Docker Environment Start
```bash
# Build and start all services
docker-compose up -d

# Verify all services are running
docker-compose ps

# Expected services:
# - ecommerce-backend (Rails API) - http://localhost:3000  
# - ecommerce-frontend (React) - http://localhost:3001
# - ecommerce-admin (Rails Admin) - http://localhost:3002
# - mysql - internal port 3306
```

### 3. Database Setup
```bash
# Run database migrations
docker-compose exec backend rails db:create db:migrate

# Seed initial data
docker-compose exec backend rails db:seed

# Verify database setup
docker-compose exec backend rails console
> User.count  # Should return 0 (no test users yet)
> Product.count  # Should return sample products from seed
> Category.count  # Should return sample categories
```

## Acceptance Test Scenarios

### Test Scenario 1: ゲストユーザーの商品カート追加

**Given**: 未登録ユーザーがサイトを訪問  
**When**: 商品詳細ページで「カートに追加」を実行  
**Then**: 商品がカートに追加され、カート内容が更新される

**Steps**:
1. ブラウザで http://localhost:3001 を開く
2. 商品一覧ページで任意の商品をクリック
3. 商品詳細ページで「カートに追加」ボタンをクリック
4. カートアイコンに商品数が表示される
5. カートページを開いて商品が表示されることを確認

**Expected API Calls**:
```bash
# Frontend makes these API calls:
GET /api/v1/products/{id}  # 商品詳細取得
POST /api/v1/cart/items    # カート追加
GET /api/v1/cart           # カート内容取得
```

**Verification**:
```bash
# API direct test
curl -X POST http://localhost:3000/api/v1/cart/items \
  -H "Content-Type: application/json" \
  -H "X-Session-ID: test-session-123" \
  -d '{"product_id": 1, "quantity": 1}'
  
# Should return 201 Created with cart item data
```

### Test Scenario 2: カート内商品の数量変更

**Given**: カートに商品が入っている状態  
**When**: カート画面で商品数量を変更  
**Then**: 合計金額が再計算され表示が更新される

**Steps**:
1. カートに商品が1つ以上ある状態から開始
2. カートページで数量変更ボタン（+/-）をクリック
3. 数量とsubtotalが即座に更新される
4. 合計金額も更新される

**API Test**:
```bash
# Update quantity
curl -X PATCH http://localhost:3000/api/v1/cart/items/1 \
  -H "Content-Type: application/json" \
  -H "X-Session-ID: test-session-123" \
  -d '{"quantity": 3}'
```

### Test Scenario 3: ゲストユーザーのアカウント登録とカート引き継ぎ

**Given**: ゲストユーザーがカートに商品を追加済み  
**When**: アカウント登録を実行  
**Then**: カート内容が引き継がれ登録ユーザーとしてログインされる

**Steps**:
1. ゲストとしてカートに商品を追加
2. 「新規登録」リンクをクリック
3. 名前、メールアドレス、パスワードを入力
4. 登録ボタンをクリック
5. ログイン済み状態になり、カート内容が保持されている

**API Test**:
```bash
# User registration with session merge (session-based auth)
curl -X POST http://localhost:3000/api/v1/auth/signup \
  -H "Content-Type: application/json" \
  -H "Cookie: _ecommerce_session=<session-cookie>" \
  -d '{
    "user": {
      "name": "Test User",
      "email": "test@example.com", 
      "password": "password123"
    }
  }'
```

### Test Scenario 4: 管理者アクセスと在庫管理

**Given**: 管理者アカウントでログイン  
**When**: 在庫管理画面にアクセス  
**Then**: 商品一覧と在庫数量が表示され編集可能である

**Steps**:
1. http://localhost:3002/admin にアクセス
2. 管理者認証情報でログイン (admin@example.com / password)
3. 商品管理メニューをクリック
4. 商品一覧で在庫数量を確認
5. 任意の商品の在庫数を変更
6. 保存後、変更が反映されることを確認

**Database Verification**:
```bash
# Check inventory update via Rails console
docker-compose exec backend rails console
> Product.first.inventory.quantity
> # Should reflect the changed quantity
```

### Test Scenario 5: 登録ユーザーのレビュー投稿

**Given**: 登録済みユーザーでログイン  
**When**: 商品詳細ページでレビューを投稿  
**Then**: レビューが商品ページに表示される

**Steps**:
1. 登録済みユーザーでログイン
2. 商品詳細ページのレビュー欄に移動
3. 星評価（1-5）を選択
4. コメントを入力
5. 「レビューを投稿」ボタンをクリック
6. レビューが商品ページに表示される

**API Test**:
```bash
# Post a review (requires session-based authentication)
curl -X POST http://localhost:3000/api/v1/products/1/reviews \
  -H "Content-Type: application/json" \
  -H "Cookie: _ecommerce_session=<logged-in-session>" \
  -d '{"rating": 5, "comment": "Great product!"}'
```

## Development Workflow Verification

### Hot Reload Test
1. **Backend**: Edit `app/controllers/api/v1/products_controller.rb`
2. **Frontend**: Edit `src/components/ProductCard.jsx`  
3. **Admin**: Edit `app/admin/products.rb`
4. Changes should be reflected without restart

### Test Execution
```bash
# Backend tests
docker-compose exec backend rspec

# Frontend tests  
docker-compose exec frontend npm test

# Should see all tests passing (green)
```

### Log Monitoring
```bash
# Monitor all service logs
docker-compose logs -f

# Monitor specific service
docker-compose logs -f backend
docker-compose logs -f frontend
docker-compose logs -f admin
```

## Troubleshooting

### Common Issues

1. **Ports already in use**:
   ```bash
   # Check what's using the ports
   lsof -i :3000
   lsof -i :3001  
   lsof -i :3002
   ```

2. **Database connection error**:
   ```bash
   # Restart MySQL container
   docker-compose restart mysql
   
   # Check database status
   docker-compose exec mysql mysql -u root -p -e "SHOW DATABASES;"
   ```

3. **Frontend build errors**:
   ```bash
   # Clear node modules and reinstall
   docker-compose exec frontend rm -rf node_modules
   docker-compose exec frontend npm install
   ```

## Success Criteria

✅ All Docker services start successfully  
✅ Database migrations run without errors  
✅ All 5 test scenarios complete successfully  
✅ Hot reload works for all services  
✅ API endpoints respond correctly  
✅ Frontend-backend integration works  
✅ Admin panel is accessible and functional  

**Time to Complete**: ~15-20 minutes for full setup and testing

## Next Steps

After quickstart completion:
- Review [data-model.md](./data-model.md) for database schema
- Check [contracts/api-spec.yaml](./contracts/api-spec.yaml) for API documentation
- Proceed with `/tasks` command to generate implementation tasks