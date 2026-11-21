| データ名 | 目的 | 主な項目 | 関連データ |
|---|---|---|---|
| Store | 店舗・ブース情報を管理する | 店舗ID、店舗名、説明、営業状態、待ち時間、待ち人数、待ち時間表示、出展日、モニター／呼び出し表示 | MenuItem, Order, MapFacility, SalesEntry, User |
| MenuItem | 店舗ごとの商品情報を管理する | 商品ID、店舗ID、商品名、説明、価格、販売状態、売り切れ | Store, OrderItem |
| Order | 店舗会計と電子呼び出し番号を管理する | 注文ID、店舗ID、開催日、受付番号、合計金額、注文状態 | Store, OrderItem |
| OrderItem | 注文された商品の明細を管理する | 注文明細ID、注文ID、商品ID、数量、単価 | Order, MenuItem |
| TicketCounter | 店舗ごとの受付番号採番を管理する | 店舗ID、最終番号 | Store, Order |
| User | 店舗・管理者のログイン情報を管理する | ユーザーID、店舗ID（管理者は null）、ログインID、パスワード、role | Store |
| MapFacility | 校内マップに表示する施設・ブース情報を管理する | 施設ID、名称、種別、階数、x座標、y座標 | Store, EventNotice |
| SalesEntry | 店舗が当日入力する売上を管理する | 店舗ID、金額、メモ、入力日時 | Store |
| EventNotice | イベント・お知らせを管理する | タイトル、本文、表示期間、種別、電光掲示板設定、場所 | MapFacility |
| FestivalSetting | 学園祭全体設定（開催日など）を管理する | key、value | なし |
| AuditLog | 管理操作・ログインなどの監査ログを管理する | action、操作者、対象、詳細、IP | なし |

#### テーブル名：
stores

目的：
学園祭の店舗・ブース情報を管理する

主なカラム：
- id
- name
- description
- ticket_prefix（未使用。互換のため残置。新規発番では使わない）
- ticket_limit（受付番号の上限。既定 20）
- disabled_ticket_numbers（使わない番号の配列）
- is_open
- is_visible
- exhibition_day（`day1` / `day2` / `both`。同じ部屋の日替わりブース用）
- show_on_monitor（校内モニターに出すか）
- show_on_call_page（来場者の呼び出しページに出すか）
- current_wait_min
- current_queue_count
- wait_display_mode（`minutes` / `text`）
- wait_display_text（時間開始型ブース向け表示文言）
- crowd_level（店内混雑。`empty` / `moderate` / `crowded`）
- time_slots（開始時刻ありブース向けの枠。JSON）
- created_at
- updated_at

主キー：
id

外部キー：
なし

初期データの有無：
あり

読み取り権限：
来場者、店舗、管理者

書き込み権限：
店舗、管理者

#### テーブル名：
menu_items

目的：
各店舗の商品情報を管理する

主なカラム：
- id
- store_id
- name
- description
- price
- is_available
- is_sold_out
- created_at
- updated_at

主キー：
id

外部キー：
store_id → stores.id

初期データの有無：
あり

読み取り権限：
来場者、店舗、管理者

書き込み権限：
店舗、管理者

#### テーブル名：
orders

目的：
店舗会計と電子呼び出し番号を管理する。モバイルオーダー・予約は行わず、店舗側の会計・呼び出し番号発行を主用途にする。

主なカラム：
- id
- store_id
- festival_day（会計時点の開催日。`day1` / `day2`）
- ticket_number
- total_price
- status
- ordered_at
- called_at
- served_at
- settled_at
- created_at
- updated_at

主キー：
id

外部キー：
store_id → stores.id

初期データの有無：
なし

読み取り権限：
来場者、店舗、管理者

書き込み権限：
店舗、管理者

#### テーブル名：
ticket_counters

目的：
各店舗ごとの受付番号（ticket_number）を原子的に採番するためのカウンタを管理します。高並列でのオーダー作成時にもユニークな受付番号を保証する用途です。

主なカラム：
- store_id (PK) — stores.id を参照
- last_number (unsigned big integer) — 最後に割り当てた数値部分（例: 101）
- created_at
- updated_at

運用メモ：
- 初回利用時は `1` から採番されます（数字だけ）。
- カウンタは各店舗単位で管理され、DB トランザクションと行ロックで安全に更新されます。
- 1〜枚数を使い回し、提供前の注文や非表示番号は飛ばします。

#### テーブル名：
order_items

目的：
注文に含まれる商品明細を管理する

主なカラム：
- id
- order_id
- menu_item_id
- quantity
- unit_price
- subtotal

主キー：
id

外部キー：
order_id → orders.id
menu_item_id → menu_items.id

初期データの有無：
なし

読み取り権限：
店舗、管理者

書き込み権限：
店舗、管理者

#### テーブル名：
users

目的：
店舗向け画面と `/admin` のログイン情報を管理する。`role` で店舗ユーザーと管理者を区別する。

主なカラム：
- id
- store_id（店舗ユーザーは必須。管理者は null）
- login_id
- password
- role（`store` / `admin`）
- created_at
- updated_at

主キー：
id

外部キー：
store_id → stores.id（null 可）

初期データの有無：
あり（UserSeeder）

読み取り権限：
管理者（自店舗ユーザーは自分の情報のみ）

書き込み権限：
管理者

#### テーブル名：
sales_entries

目的：
店舗が当日アプリ内で売上を入力し、紙媒体の決済合計と突き合わせるための売上記録を管理する。

主なカラム：
- id
- store_id
- amount
- memo
- recorded_at
- created_at
- updated_at

主キー：
id

外部キー：
store_id → stores.id

読み取り権限：
店舗、管理者

書き込み権限：
店舗、管理者

#### テーブル名：
event_notices

目的：
イベント画面に表示するお知らせ・タイムテーブル・運営連絡を管理する。ホームの電光掲示板にも利用する。

主なカラム：
- id
- title
- body
- type（event / notice）
- starts_at
- ends_at
- is_published
- ticker_lead_minutes
- ticker_after_minutes
- ticker_started_at
- map_facility_id（開催場所。map_facilities.id）
- created_at
- updated_at

主キー：
id

外部キー：
map_facility_id → map_facilities.id（任意）

読み取り権限：
来場者、店舗、管理者

書き込み権限：
管理者

#### テーブル名：
audit_logs

目的：
管理操作やログインなどの監査ログを残す。

主なカラム：
- id
- action
- actor_login_id
- target_id
- details（JSON。パスワードは入れない）
- ip
- created_at

主キー：
id

読み取り権限：
管理者

書き込み権限：
システム（API 側で自動記録）

#### テーブル名：
map_facilities

目的：
校内マップに表示する施設・ブース情報を管理する

主なカラム：
- id
- store_id（店舗に紐づかない施設は null 可）
- name
- type
- floor
- x
- y
- created_at
- updated_at

主キー：
id

外部キー：
store_id → stores.id（null 可）

初期データの有無：
あり

読み取り権限：
来場者、店舗、管理者

書き込み権限：
管理者

#### テーブル名：
festival_settings

目的：
学園祭の「今日が何日目か」など、全体設定を持つ

主なカラム：
- key（PK。例: `current_day`）
- value（例: `day1` / `day2`）
- created_at
- updated_at

読み取り権限：
来場者（API経由）、管理者

書き込み権限：
管理者
