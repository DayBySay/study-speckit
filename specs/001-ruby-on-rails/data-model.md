# Data Model: ECサイトプラットフォーム

**Date**: 2025-09-19  
**Source**: Extracted from [spec.md](./spec.md) functional requirements

## Core Entities

### User (顧客アカウント)
```ruby
# users table
class User < ApplicationRecord
  # Authentication
  email: string, null: false, unique: true
  password_digest: string, null: false
  
  # Profile
  name: string, null: false
  
  # Timestamps
  created_at: datetime
  updated_at: datetime
  
  # Relationships
  has_many :reviews, dependent: :destroy
  has_one :wishlist, dependent: :destroy
  has_many :cart_items, dependent: :destroy
end
```

**Validation Rules**:
- Email format validation (RFC compliant)
- Password minimum 8 characters
- Name presence and length (2-50 chars)

### Product (商品)
```ruby
# products table  
class Product < ApplicationRecord
  # Basic Info
  name: string, null: false
  description: text
  price: decimal(10,2), null: false
  image_url: string
  
  # Organization
  category_id: integer, null: false, foreign_key: true
  
  # Status
  active: boolean, default: true
  
  # Timestamps
  created_at: datetime
  updated_at: datetime
  
  # Relationships
  belongs_to :category
  has_one :inventory, dependent: :destroy
  has_many :reviews, dependent: :destroy
  has_many :cart_items, dependent: :destroy
  has_many :wishlist_items, dependent: :destroy
end
```

**Validation Rules**:
- Name presence and uniqueness per category
- Price positive number
- Description optional but max 1000 chars
- Image URL format validation

### Category (カテゴリ)
```ruby
# categories table
class Category < ApplicationRecord
  name: string, null: false, unique: true
  description: text
  active: boolean, default: true
  
  # Timestamps  
  created_at: datetime
  updated_at: datetime
  
  # Relationships
  has_many :products, dependent: :restrict_with_exception
end
```

### Inventory (在庫)
```ruby
# inventories table
class Inventory < ApplicationRecord
  product_id: integer, null: false, foreign_key: true, unique: true
  quantity: integer, null: false, default: 0
  
  # Timestamps
  created_at: datetime  
  updated_at: datetime
  
  # Relationships
  belongs_to :product
end
```

**Validation Rules**:
- Quantity non-negative integer
- One inventory record per product

### CartItem (カートアイテム)
```ruby
# cart_items table
class CartItem < ApplicationRecord  
  # For registered users
  user_id: integer, null: true, foreign_key: true
  
  # For guest sessions  
  session_id: string, null: true
  
  # Product reference
  product_id: integer, null: false, foreign_key: true
  quantity: integer, null: false, default: 1
  
  # Timestamps
  created_at: datetime
  updated_at: datetime
  
  # Relationships  
  belongs_to :user, optional: true
  belongs_to :product
end
```

**Validation Rules**:
- Either user_id OR session_id must be present
- Quantity positive integer
- Unique combination of (user_id/session_id, product_id)

### Review (レビュー)
```ruby
# reviews table
class Review < ApplicationRecord
  user_id: integer, null: false, foreign_key: true
  product_id: integer, null: false, foreign_key: true
  
  # Content
  rating: integer, null: false
  comment: text
  
  # Status
  approved: boolean, default: true
  
  # Timestamps
  created_at: datetime
  updated_at: datetime
  
  # Relationships
  belongs_to :user  
  belongs_to :product
end
```

**Validation Rules**:
- Rating between 1-5 stars
- Comment max 500 chars
- One review per user per product

### Wishlist (ウィッシュリスト)
```ruby  
# wishlists table
class Wishlist < ApplicationRecord
  user_id: integer, null: false, foreign_key: true, unique: true
  
  # Timestamps
  created_at: datetime
  updated_at: datetime
  
  # Relationships
  belongs_to :user
  has_many :wishlist_items, dependent: :destroy
  has_many :products, through: :wishlist_items
end
```

### WishlistItem (ウィッシュリストアイテム)  
```ruby
# wishlist_items table
class WishlistItem < ApplicationRecord
  wishlist_id: integer, null: false, foreign_key: true
  product_id: integer, null: false, foreign_key: true
  
  # Timestamps
  created_at: datetime
  updated_at: datetime
  
  # Relationships
  belongs_to :wishlist
  belongs_to :product
end
```

**Validation Rules**:
- Unique combination of (wishlist_id, product_id)

### Admin (管理者)
```ruby
# admins table (separate from users)
class Admin < ApplicationRecord
  email: string, null: false, unique: true
  password_digest: string, null: false
  name: string, null: false
  
  # Permissions
  role: string, default: 'editor' # editor, admin
  active: boolean, default: true
  
  # Timestamps
  created_at: datetime
  updated_at: datetime
end
```

**Validation Rules**:
- Email format validation, separate namespace from users
- Role in ['editor', 'admin']
- Name presence

## Relationships Summary

```
User (1) --> (0..*) Review --> (1) Product
User (1) --> (0..1) Wishlist --> (0..*) WishlistItem --> (1) Product  
User (1) --> (0..*) CartItem --> (1) Product

Product (1) --> (0..1) Inventory
Product (1) --> (0..*) Review
Product (1) --> (0..*) CartItem
Product (1) --> (0..*) WishlistItem
Product (*) --> (1) Category

Admin (independent entity - no relationships to User domain)
```

## State Transitions

### CartItem Lifecycle
1. **Created** (guest or user adds product)
2. **Updated** (quantity changes)  
3. **Merged** (guest session → user cart on registration)
4. **Deleted** (removed from cart)

### Product Lifecycle  
1. **Draft** (created by admin, active: false)
2. **Active** (published, active: true)
3. **Inactive** (archived, active: false)

### Inventory Updates
- **Stock Check** before adding to cart
- **Stock Reserve** on cart add (optional)
- **Stock Release** on cart remove/timeout

## Database Indexes

**Performance-Critical Indexes**:
```sql
-- User lookups
CREATE INDEX idx_users_email ON users(email);

-- Product queries  
CREATE INDEX idx_products_category_active ON products(category_id, active);
CREATE INDEX idx_products_active_created ON products(active, created_at);

-- Cart operations
CREATE INDEX idx_cart_items_user ON cart_items(user_id);
CREATE INDEX idx_cart_items_session ON cart_items(session_id);  
CREATE INDEX idx_cart_items_product ON cart_items(product_id);

-- Review queries
CREATE INDEX idx_reviews_product ON reviews(product_id, approved);
CREATE INDEX idx_reviews_user ON reviews(user_id);

-- Wishlist operations
CREATE INDEX idx_wishlist_items_wishlist ON wishlist_items(wishlist_id);
```