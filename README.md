# タイヤ管理システム データベースER図

## ER図（Entity-Relationship Diagram）

```mermaid
erDiagram
    stores ||--o{ customers : "has many"
    customers ||--o{ tire_sets : "has many"
    tire_sets ||--o{ tire_storage : "has many"
    tire_sets ||--o{ tire_images : "has many"
    locations ||--o{ tire_storage : "has many"

    stores {
        bigint id PK
        string store_code UK "店舗コード"
        string name "店舗名"
        string phone "電話番号"
        string email "メール"
        string postal_code "郵便番号"
        string address "住所"
        boolean is_active "有効フラグ"
        text notes "備考"
        timestamp created_at
        timestamp updated_at
    }

    customers {
        bigint id PK
        bigint store_id FK "店舗ID"
        string customer_code UK "顧客コード"
        string name "顧客名"
        string name_kana "フリガナ"
        string phone "電話番号"
        string email "メール"
        string postal_code "郵便番号"
        string address "住所"
        string car_number "車両番号"
        string car_model "車種"
        text notes "備考"
        timestamp created_at
        timestamp updated_at
    }

    tire_sets {
        bigint id PK
        string qr_code UK "QRコード"
        bigint customer_id FK "顧客ID"
        enum tire_type "タイヤタイプ(summer/winter/all_season)"
        int tire_count "本数"
        string manufacturer "メーカー"
        string brand "ブランド"
        string size "サイズ"
        string production_year "製造年"
        boolean rim_included "リム付き"
        int condition_score "状態スコア"
        text notes "備考"
        timestamp created_at
        timestamp updated_at
    }

    tire_storage {
        bigint id PK
        bigint tire_set_id FK "タイヤセットID"
        bigint location_id FK "保管場所ID"
        date storage_date "保管日"
        date scheduled_return_date "返却予定日"
        date actual_return_date "実際の返却日"
        enum status "ステータス(stored/returned/reserved)"
        decimal storage_fee "保管料"
        boolean paid "支払済み"
        string checked_in_by "入庫担当者"
        string checked_out_by "出庫担当者"
        text notes "備考"
        timestamp created_at
        timestamp updated_at
    }

    locations {
        bigint id PK
        string location_code UK "ロケーションコード"
        string rack_name "ラック名"
        string section "セクション"
        string rack_row "ラック行"
        string rack_column "ラック列"
        int capacity "収容数"
        boolean is_available "利用可能"
        text notes "備考"
        timestamp created_at
        timestamp updated_at
    }

    tire_images {
        bigint id PK
        bigint tire_set_id FK "タイヤセットID"
        string file_path "ファイルパス"
        string file_name "ファイル名"
        bigint file_size "ファイルサイズ"
        string mime_type "MIMEタイプ"
        int display_order "表示順"
        text description "説明"
        timestamp created_at
        timestamp updated_at
    }

    users {
        bigint id PK
        string name "ユーザー名"
        string email UK "メール"
        timestamp email_verified_at
        string password "パスワード"
        string role "役割(admin/staff)"
        string remember_token
        timestamp created_at
        timestamp updated_at
    }
```

## データベース構造の概要

### 主要エンティティ

#### 1. Store（店舗）
- 複数の店舗を管理するためのマスターテーブル
- 各店舗に固有の店舗コード（store_code）を持つ
- 店舗ごとに顧客を管理

#### 2. Customer（顧客）
- 店舗に紐づく顧客情報を管理
- 顧客コード、連絡先、住所、車両情報を保持
- 各顧客は複数のタイヤセットを所有可能

#### 3. TireSet（タイヤセット）
- 顧客が所有するタイヤの情報
- QRコードで一意に識別
- タイヤのタイプ（夏/冬/オールシーズン）、メーカー、サイズ、状態などを記録

#### 4. TireStorage（タイヤ保管履歴）
- タイヤセットの保管・返却履歴を管理
- 保管場所（Location）との紐付け
- 入出庫日、料金、支払状況、担当者を記録
- ステータス管理（保管中/返却済み/予約済み）

#### 5. Location（保管場所）
- タイヤの物理的な保管位置を管理
- ラック名、セクション、行、列で位置を特定
- 収容数と利用可否を管理

#### 6. TireImage（タイヤ画像）
- タイヤセットに関連する画像ファイルを管理
- タイヤの状態確認用の写真を保存
- 表示順序を管理可能

#### 7. User（ユーザー）
- システムの認証・認可を管理
- 役割（admin/staff）によるアクセス制御
- Laravel SanctumトークンによるAPI認証

## リレーションシップ

### 1対多の関係

1. **Store → Customer**
   - 1つの店舗は複数の顧客を持つ
   - 外部キー: `customers.store_id`
   - 削除時: SET NULL

2. **Customer → TireSet**
   - 1人の顧客は複数のタイヤセットを所有
   - 外部キー: `tire_sets.customer_id`
   - 削除時: CASCADE（顧客削除時にタイヤセットも削除）

3. **TireSet → TireStorage**
   - 1つのタイヤセットは複数の保管履歴を持つ
   - 外部キー: `tire_storage.tire_set_id`
   - 削除時: CASCADE

4. **TireSet → TireImage**
   - 1つのタイヤセットは複数の画像を持つ
   - 外部キー: `tire_images.tire_set_id`
   - 削除時: CASCADE

5. **Location → TireStorage**
   - 1つの保管場所は複数の保管履歴を持つ
   - 外部キー: `tire_storage.location_id`
   - 削除時: SET NULL

## インデックス戦略

### ユニークインデックス
- `stores.store_code` - 店舗コード
- `customers.customer_code` - 顧客コード
- `tire_sets.qr_code` - QRコード
- `locations.location_code` - ロケーションコード
- `users.email` - メールアドレス

### 検索用インデックス
- `customers.phone` - 電話番号検索
- `customers.name` - 顧客名検索
- `tire_sets.tire_type` - タイヤタイプフィルタ
- `tire_storage.status` - ステータス検索
- `tire_storage.storage_date` - 保管日検索
- `locations.rack_name` - ラック名検索

## データ整合性

### 外部キー制約
- すべての関連テーブルに外部キー制約を設定
- CASCADE削除: 親データ削除時に関連データも自動削除
- SET NULL: 親データ削除時に外部キーをNULLに設定

### タイムスタンプ
- すべてのテーブルに `created_at` と `updated_at` を設定
- データの作成・更新履歴を自動記録

## 技術仕様

- **ORM**: Laravel Eloquent
- **データベース**: MySQL / SQLite（開発環境）
- **文字コード**: UTF8MB4（多言語対応）
- **認証**: Laravel Sanctum（APIトークン認証）
- **バージョン管理**: Laravel Migrations

## 使用方法

このER図はMermaid記法で記述されています。以下のツールで表示可能です：

1. **GitHub** - このファイルをGitHubにプッシュすると自動的にレンダリングされます
2. **VS Code** - Mermaid Preview拡張機能をインストール
3. **オンラインエディタ** - https://mermaid.live/ で直接編集・表示
4. **ドキュメントツール** - GitBook、Docusaurus、MkDocsなどで使用可能
