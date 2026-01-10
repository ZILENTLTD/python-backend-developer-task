# Backend Developer Evaluation Task

## Overview

Build a mini e-commerce API for a **Product Catalog System** with categories, products, pricing, and order tracking. This task evaluates your ability to design APIs, work with Django REST Framework, implement middleware, signals, and write clean, testable code.

**Time Limit:** 6-8 hours
**Submission:** GitHub repository link with README

---

## Part 1: Models (15 points)

Create the following Django models:

```python
Category
├── id (UUID, primary key)
├── name (CharField, max_length=255)
├── description (TextField, blank=True, null=True)
├── parent_category (ForeignKey to self, null=True, blank=True)
├── image_url (URLField, blank=True, null=True)
├── active (IntegerField, default=1)  # 1=active, 0=inactive
├── delete_status (IntegerField, default=0)  # 0=not deleted, 1=deleted
├── created_at (DateTimeField, auto_now_add=True)
└── updated_at (DateTimeField, auto_now=True)

Product
├── id (UUID, primary key)
├── name (CharField, max_length=255)
├── code (CharField, unique, max_length=50)
├── description (TextField)
├── category (ForeignKey to Category)
├── base_price (DecimalField, max_digits=10, decimal_places=2)
├── discount_percent (IntegerField, default=0, validators=[0-100])
├── stock_quantity (IntegerField, default=0)
├── active (IntegerField, default=1)
├── delete_status (IntegerField, default=0)
├── created_at (DateTimeField, auto_now_add=True)
└── updated_at (DateTimeField, auto_now=True)

Order
├── id (UUID, primary key)
├── order_code (CharField, unique, auto-generated)
├── product (ForeignKey to Product)
├── quantity (IntegerField)
├── unit_price (DecimalField)
├── total_price (DecimalField)
├── status (IntegerField, default=0)
│   # 0=pending, 10=confirmed, 20=processing, 30=shipped, 40=delivered, 50=cancelled
├── status_changed_at (DateTimeField, null=True)
├── created_at (DateTimeField, auto_now_add=True)
└── updated_at (DateTimeField, auto_now=True)

OrderStatusHistory
├── id (UUID, primary key)
├── order (ForeignKey to Order)
├── old_status (IntegerField, null=True)
├── new_status (IntegerField)
├── changed_by (CharField, max_length=100, null=True)  # user identifier
├── change_source (CharField)  # 'api', 'admin', 'system'
├── ip_address (GenericIPAddressField, null=True)
├── created_at (DateTimeField, auto_now_add=True)
└── notes (TextField, blank=True, null=True)
```

**Evaluation Criteria:**
- Proper field types, constraints, and validators
- Soft delete pattern (delete_status field)
- UUID primary keys
- Proper indexes on frequently queried fields

---

## Part 2: API Endpoints (30 points)

### Categories API

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/categories/` | List all parent categories with nested subcategories |
| GET | `/api/categories/{id}/` | Get single category with subcategories |
| POST | `/api/categories/` | Create a new category |
| PATCH | `/api/categories/{id}/` | Update a category |
| DELETE | `/api/categories/{id}/` | Soft delete (set delete_status=1) |

### Products API

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/products/` | List products with filters |
| GET | `/api/products/{id}/` | Get single product with category details |
| POST | `/api/products/` | Create a new product |
| PATCH | `/api/products/{id}/` | Update a product |
| DELETE | `/api/products/{id}/` | Soft delete |

### Orders API

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/orders/` | List orders with filters |
| GET | `/api/orders/{id}/` | Get order with status history |
| POST | `/api/orders/` | Create a new order |
| PATCH | `/api/orders/{id}/` | Update order (especially status) |
| GET | `/api/orders/{id}/history/` | Get status change history |

### Required Features:

1. **Nested Subcategories** - Categories return subcategories recursively
2. **Computed Fields** - `final_price`, `in_stock`, `discount_amount`
3. **Filtering** - Products by `category`, `min_price`, `max_price`, `in_stock`
4. **Filtering** - Orders by `status`, `created_at` date range
5. **Pagination** - Limit/Offset pagination
6. **Search** - Products searchable by name
7. **Soft Delete** - All DELETE endpoints set `delete_status=1`

---

## Part 3: Response Formats (Must Match)

### GET /api/categories/

```json
{
    "count": 2,
    "next": null,
    "previous": null,
    "results": [
        {
            "id": "550e8400-e29b-41d4-a716-446655440000",
            "name": "Electronics",
            "description": "Electronic items and gadgets",
            "image_url": "https://example.com/electronics.jpg",
            "parent_category": null,
            "active": 1,
            "sub_categories": [
                {
                    "id": "550e8400-e29b-41d4-a716-446655440001",
                    "name": "Mobile Phones",
                    "description": "Smartphones and feature phones",
                    "image_url": null,
                    "parent_category": "550e8400-e29b-41d4-a716-446655440000",
                    "active": 1,
                    "sub_categories": []
                },
                {
                    "id": "550e8400-e29b-41d4-a716-446655440002",
                    "name": "Laptops",
                    "description": "Notebooks and laptops",
                    "image_url": null,
                    "parent_category": "550e8400-e29b-41d4-a716-446655440000",
                    "active": 1,
                    "sub_categories": []
                }
            ]
        }
    ]
}
```

### GET /api/products/{id}/

```json
{
    "id": "660e8400-e29b-41d4-a716-446655440000",
    "name": "iPhone 15 Pro",
    "code": "IPHONE15PRO",
    "description": "Latest Apple iPhone with A17 chip",
    "category": {
        "id": "550e8400-e29b-41d4-a716-446655440001",
        "name": "Mobile Phones",
        "parent_category": "550e8400-e29b-41d4-a716-446655440000"
    },
    "base_price": "999.00",
    "discount_percent": 10,
    "final_price": "899.10",
    "discount_amount": "99.90",
    "stock_quantity": 50,
    "in_stock": true,
    "active": 1,
    "created_at": "2024-01-15T10:30:00Z",
    "updated_at": "2024-01-15T10:30:00Z"
}
```

### GET /api/orders/{id}/

```json
{
    "id": "770e8400-e29b-41d4-a716-446655440000",
    "order_code": "ORD-20240115-001",
    "product": {
        "id": "660e8400-e29b-41d4-a716-446655440000",
        "name": "iPhone 15 Pro",
        "code": "IPHONE15PRO"
    },
    "quantity": 2,
    "unit_price": "899.10",
    "total_price": "1798.20",
    "status": 10,
    "status_display": "Confirmed",
    "status_changed_at": "2024-01-15T11:00:00Z",
    "created_at": "2024-01-15T10:30:00Z",
    "updated_at": "2024-01-15T11:00:00Z",
    "timeline": [
        {
            "status": 0,
            "status_display": "Pending",
            "changed_at": "2024-01-15T10:30:00Z",
            "changed_by": null,
            "change_source": "api"
        },
        {
            "status": 10,
            "status_display": "Confirmed",
            "changed_at": "2024-01-15T11:00:00Z",
            "changed_by": "admin@example.com",
            "change_source": "admin"
        }
    ]
}
```

---

## Part 4: Middleware (15 points)

Create a **RequestContextMiddleware** that:

1. **Stores the current request** in thread-local storage
2. **Provides helper functions:**
   - `get_current_request()` - Returns current request or None
   - `get_client_ip(request)` - Extracts client IP (handles X-Forwarded-For)
3. **Cleans up** after request processing

```python
# Expected usage in signals:
from apps.orders.middleware import get_current_request, get_client_ip

request = get_current_request()
if request:
    ip = get_client_ip(request)
    user = request.user if request.user.is_authenticated else None
```

**Evaluation Criteria:**
- Proper thread-local storage implementation
- Handles proxy headers (X-Forwarded-For)
- Cleans up request after processing
- Registered correctly in settings.py MIDDLEWARE

---

## Part 5: Signals (20 points)

Implement the following signals for Order model:

### 1. Pre-save Signal: `track_order_status_change`
- Capture the old status before saving
- Store in instance attribute for post_save access

### 2. Post-save Signal: `create_order_status_history`
- Create `OrderStatusHistory` entry when status changes
- Capture: old_status, new_status, changed_by, change_source, ip_address
- Use middleware to get request context

### 3. Pre-save Signal: `update_status_timestamp`
- Auto-set `status_changed_at` when status changes

### 4. Post-save Signal: `update_stock_on_order`
- Decrease product stock when order is created
- Restore stock if order is cancelled (status=50)

```python
# Example signal flow:
# 1. Order created with status=0 → stock decreases, history created
# 2. Order status updated to 10 → status_changed_at set, history created
# 3. Order cancelled (status=50) → stock restored, history created
```

**Evaluation Criteria:**
- Correct use of pre_save and post_save
- Proper signal registration in apps.py or signals.py
- Uses middleware for request context
- Handles edge cases (new vs update, stock validation)

---

## Part 6: Testing (15 points)

Write tests covering:

### Model Tests
- Category creation with parent relationship
- Product with valid/invalid discount_percent
- Order creation decreases stock

### API Tests
- List categories with nested subcategories
- Create product with category
- Filter products by price range and stock
- Update order status creates history entry
- Soft delete behavior

### Signal Tests
- Status change creates OrderStatusHistory
- Stock updates on order create/cancel
- Timestamp auto-updates on status change

### Middleware Tests
- Request context available in signals
- IP extraction from headers

**Minimum:** 12-15 test cases

---

## Part 7: Code Quality (5 points)

- Clean, readable code
- DRY principles
- Proper error handling
- README with setup instructions
- Requirements.txt

---

## Bonus Points (Optional)

| Bonus Task | Points |
|------------|--------|
| Docker setup (Dockerfile + docker-compose) | +5 |
| API documentation with drf-spectacular | +5 |
| JWT Authentication on write endpoints | +5 |
| Custom FilterSet with complex lookups | +3 |
| Management command to seed test data | +2 |

---

## Submission Checklist

- [ ] GitHub repository with clean commit history
- [ ] README with setup and run instructions
- [ ] All migrations included
- [ ] Tests passing (pytest or Django TestCase)
- [ ] Middleware registered in settings
- [ ] Signals properly connected
- [ ] Sample data or fixtures
- [ ] Postman collection (optional)

---

## Evaluation Scoring

| Section | Points |
|---------|--------|
| Models | 15 |
| API Endpoints | 30 |
| Middleware | 15 |
| Signals | 20 |
| Testing | 15 |
| Code Quality | 5 |
| **Total** | **100** |
| Bonus | Up to +20 |

**Pass Threshold:** 75 points

---

## Status Code Reference

```python
ORDER_STATUS_CHOICES = [
    (0, 'Pending'),
    (10, 'Confirmed'),
    (20, 'Processing'),
    (30, 'Shipped'),
    (40, 'Delivered'),
    (50, 'Cancelled'),
]
```

---

## Tech Stack

- Python 3.10+
- Django 4.x or 5.x
- Django REST Framework
- PostgreSQL (or SQLite for simplicity)
- pytest or Django TestCase

---

*Good luck! Focus on clean, working code that follows the patterns shown above.*
