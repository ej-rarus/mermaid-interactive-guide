```mermaid
---
title: MISTO POS - Data Model ER Diagram
---
erDiagram
    SHOP ||--o{ USER : "has"
    SHOP ||--o{ DAY_OPERATION : "operates"
    SHOP ||--o{ ORDER : "processes"

    USER ||--o{ ORDER : "creates"
    USER ||--o{ DAY_OPERATION : "manages"

    MEMBER ||--o{ ORDER : "purchases"

    ORDER ||--|{ ORDER_ITEM : "contains"
    ORDER ||--o{ PAYMENT : "pays_with"
    ORDER ||--o{ REFUND : "may_have"
    ORDER ||--o{ ORDER_PROMOTION : "applies"

    PRODUCT ||--o{ ORDER_ITEM : "ordered_as"

    PROMOTION ||--o{ ORDER_PROMOTION : "used_in"

    PAYMENT_METHOD ||--o{ PAYMENT : "uses"
    PAYMENT_METHOD ||--o{ DAY_OPERATION_DETAIL : "tracked_in"

    DAY_OPERATION ||--|{ DAY_OPERATION_DETAIL : "details"

    REFUND ||--o{ REFUND_ITEM : "includes"

    SHOP {
        int shop_id PK
        string shop_name
        string address
        string phone
        string timezone
        string currency
        datetime created_at
        datetime updated_at
    }

    USER {
        int user_id PK
        int shop_id FK
        string user_code
        string user_name
        string role
        string password_hash
        boolean is_active
        datetime created_at
        datetime updated_at
    }

    MEMBER {
        int member_id PK
        string member_number
        string member_name
        string phone
        string email
        string member_grade
        boolean is_employee
        decimal employee_purchase_limit
        decimal employee_limit_used
        datetime created_at
        datetime updated_at
    }

    PRODUCT {
        int product_id PK
        string product_code
        string barcode
        string product_name
        string style_code
        string category
        string size
        string color
        decimal original_price
        decimal selling_price
        int stock_quantity
        string description
        datetime created_at
        datetime updated_at
    }

    ORDER {
        int order_id PK
        int shop_id FK
        int user_id FK
        int member_id FK
        string order_number
        string receipt_number
        datetime order_date
        decimal sub_total
        decimal shipping_fee
        decimal tax_amount
        decimal promotion_discount
        decimal manual_discount
        decimal grand_total
        string status
        datetime created_at
        datetime updated_at
    }

    ORDER_ITEM {
        int order_item_id PK
        int order_id FK
        int product_id FK
        int quantity
        decimal original_price
        decimal selling_price
        decimal line_total
        string size
        string color
        datetime created_at
    }

    PAYMENT {
        int payment_id PK
        int order_id FK
        int payment_method_id FK
        decimal amount_received
        decimal payment_amount
        decimal change_amount
        datetime payment_date
        string transaction_id
        string status
        datetime created_at
    }

    PAYMENT_METHOD {
        int payment_method_id PK
        string method_name
        string method_code
        boolean is_active
        datetime created_at
    }

    PROMOTION {
        int promotion_id PK
        string promotion_name
        string promotion_type
        string promotion_code
        decimal discount_value
        string discount_type
        datetime start_date
        datetime end_date
        string conditions
        boolean is_active
        datetime created_at
        datetime updated_at
    }

    ORDER_PROMOTION {
        int order_promotion_id PK
        int order_id FK
        int promotion_id FK
        decimal discount_amount
        datetime applied_at
    }

    DAY_OPERATION {
        int day_operation_id PK
        int shop_id FK
        int user_id FK
        date operation_date
        datetime day_open_time
        datetime day_close_time
        string status
        decimal total_over_short
        string notes
        datetime created_at
        datetime updated_at
    }

    DAY_OPERATION_DETAIL {
        int day_operation_detail_id PK
        int day_operation_id FK
        int payment_method_id FK
        decimal opening_balance
        decimal amount_received
        decimal total_amount
        decimal closing_balance
        decimal over_short
        boolean is_confirmed
        datetime created_at
    }

    REFUND {
        int refund_id PK
        int order_id FK
        int user_id FK
        string refund_type
        decimal refund_amount
        string refund_reason
        datetime refund_date
        string status
        boolean is_shopify_synced
        datetime created_at
        datetime updated_at
    }

    REFUND_ITEM {
        int refund_item_id PK
        int refund_id FK
        int order_item_id FK
        int refund_quantity
        decimal refund_amount
        datetime created_at
    }
```

## Database Schema Relationships

### Core Business Flow
1. **Shop Management**: SHOP → USER → DAY_OPERATION
2. **Sales Process**: USER → ORDER → ORDER_ITEM → PRODUCT
3. **Member Management**: MEMBER → ORDER
4. **Payment Processing**: ORDER → PAYMENT → PAYMENT_METHOD
5. **Promotion System**: PROMOTION → ORDER_PROMOTION → ORDER
6. **Refund Handling**: ORDER → REFUND → REFUND_ITEM

### Key Constraints
- One ORDER can have multiple PAYMENTS (split payment)
- One ORDER can have multiple PROMOTIONS applied
- One DAY_OPERATION has multiple DAY_OPERATION_DETAIL (one per payment method)
- One REFUND can have multiple REFUND_ITEMS (partial refund)
- MEMBER is optional for ORDER (non-member purchases allowed)
