# データベースER図

`docs/database.md` のテーブル定義と、現行の Laravel マイグレーションをもとにした ER 図です。

```mermaid
erDiagram
    STORES {
        string id PK
        string name
        string description
        string ticket_prefix
        int ticket_limit
        boolean is_open
        boolean is_visible
        string exhibition_day
        boolean show_on_monitor
        boolean show_on_call_page
        int current_wait_min
        int current_queue_count
        string wait_display_mode
        string wait_display_text
        string crowd_level
        datetime created_at
        datetime updated_at
    }

    MENU_ITEMS {
        int id PK
        string store_id FK
        string name
        string description
        int price
        boolean is_available
        boolean is_sold_out
        datetime created_at
        datetime updated_at
    }

    ORDERS {
        int id PK
        string store_id FK
        string festival_day
        string ticket_number
        int total_price
        string status
        datetime ordered_at
        datetime called_at
        datetime served_at
        datetime settled_at
        datetime created_at
        datetime updated_at
    }

    ORDER_ITEMS {
        int id PK
        int order_id FK
        int menu_item_id FK
        int quantity
        int unit_price
        int subtotal
    }

    TICKET_COUNTERS {
        string store_id PK
        int last_number
        datetime created_at
        datetime updated_at
    }

    USERS {
        int id PK
        string store_id FK
        string login_id
        string password
        string role
        datetime created_at
        datetime updated_at
    }

    MAP_FACILITIES {
        int id PK
        string store_id FK
        string name
        string type
        int floor
        int x
        int y
        datetime created_at
        datetime updated_at
    }

    SALES_ENTRIES {
        int id PK
        string store_id FK
        int amount
        string memo
        datetime recorded_at
        datetime created_at
        datetime updated_at
    }

    EVENT_NOTICES {
        int id PK
        string title
        string body
        string type
        datetime starts_at
        datetime ends_at
        boolean is_published
        int ticker_lead_minutes
        int ticker_after_minutes
        datetime ticker_started_at
        int map_facility_id FK
        datetime created_at
        datetime updated_at
    }

    FESTIVAL_SETTINGS {
        string key PK
        string value
        datetime created_at
        datetime updated_at
    }

    AUDIT_LOGS {
        int id PK
        string action
        string actor_login_id
        string target_id
        json details
        string ip
        datetime created_at
    }

    STORES ||--o{ MENU_ITEMS : has
    STORES ||--o{ ORDERS : receives
    STORES ||--o| TICKET_COUNTERS : numbers
    STORES ||--o{ USERS : authenticates
    STORES ||--o{ MAP_FACILITIES : displays
    STORES ||--o{ SALES_ENTRIES : records
    ORDERS ||--o{ ORDER_ITEMS : contains
    MENU_ITEMS ||--o{ ORDER_ITEMS : referenced_by
    MAP_FACILITIES ||--o{ EVENT_NOTICES : locates
```

## 補足

- `stores` は店舗・ブースの基本情報、待ち時間、出展日、モニター／呼び出し表示を持つ
- `orders` と `order_items` は店舗レジでの会計・受付番号発行を表す（来場者の事前モバイルオーダーではない）
- `users` は店舗ログイン（`role=store`）と管理者ログイン（`role=admin`）を兼ねる。管理者の `store_id` は null
- `ticket_counters` は店舗ごとの受付番号採番用カウンタ
- `map_facilities` は校内マップ上の施設・ブース位置を表す（`store_id` は null 可）
- `sales_entries` は店舗が手入力する当日売上
- `event_notices` はイベント／お知らせとホーム電光掲示板の元データ
- `festival_settings` は開催日など全体設定（例: `current_day`）
- `audit_logs` は管理操作・ログインなどの監査ログ
