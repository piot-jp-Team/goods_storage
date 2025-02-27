# SSGS
Service station goods strage system
お預かり品管理システムの設計（案）

誰の何を、何処に収納しているのかを管理する。

夏用タイヤ冬用タイヤの交換など一人の利用者が物品を交互に預けられるように
物品ごとにタグ付けする。

```mermaid
sequenceDiagram
    participant 所有者
    participant 管理者
    participant DB
    所有者-->>管理者: 電話予約
    管理者-->>DB: 登録　or オンラインIncoming Webhooks
    管理者-->>所有者: 予約データがない場合、伝票に記入依頼
    所有者->>管理者: 保管物品
    管理者-->>DB: 登録(GOODSタグ)
    管理者->>保管場所: 保管物品
    管理者-->>DB: 登録(LOCATONタグスキャン)
    Note right of 管理者: 保管ロケーションへ <br/>写真撮影し <br/>情報登録
    管理者-->>所有者: 保管伝票を手渡し
    loop 保管状態管理
        管理者<<->>保管場所: 
    end
    所有者-->>管理者: 返却依頼
    Note right of 管理者: 保管ロケーションへ
    管理者-->>DB: 登録(GOODSタグ)
    保管場所->>管理者: 保管物品
    管理者-->>DB: 登録(LOCATONタグスキャン)
    管理者->>所有者: 返却
    Note left of 管理者: 返却情報登録
```

# データ構成

1.所有者　OWNER
* 所有者ID
* 所有者属性（名前、他）
* 登録日時
* 変更日時
* ステータス
  
2.品物　GOODS
* 品物ID
* 品物属性（品名、物品区分、他）
* 物品下げタグ（QRコードやRFIDタグなど）
* 所有者ID
* 登録日時
* 変更日時
* ステータス
  
3.場所　LOCATION
* ロケーションID
* ロケーション番号
* ロケーションタグ（QRコードやRFIDタグなど）
* 場所属性（場所名、他）
* 備考
* 登録日時
* 変更日時
* ステータス
  
4.予約伝票 RESERVATIONS
* 伝票ID
* 伝票番号
* 区分
* 所有者ID
* 予約日時
* 備考
* 登録日時
* 変更日時
* ステータス
  
5.保管伝票 SLIPS
* 伝票ID
* 伝票番号
* 所有者ID
* 備考
* 登録日時
* 変更日時
* ステータス

6.伝票明細 SLIP_DETAILS
* 伝票明細ID
* 保管伝票ID
* 明細行NO
* 品物ID
* 場所ID
* 保管開始日時
* 保管終了日時
* 備考
* 登録日時
* 変更日時
* ステータス

# ER図
```mermaid
erDiagram
    LOCATION ||--o{ SLIP_DETAILS : allows
    ADMIN ||--o{ OWNER : makes
    OWNER ||--o{ GOODS : allows
    GOODS ||--o{ SLIP_DETAILS : allows
    SLIPS ||--o{ OWNER : allows
    SLIPS ||--o{ SLIP_DETAILS : allows

  
    OWNER {
        integer id PK
        string firstName "Only 99 characters are allowed"
        string lastName
        string address
        string phone
        string email
        string status
    }

    GOODS {
        integer id PK
        string Name "Only 50 characters are allowed"
        string type
        string status
    }
  
    LOCATION {
        integer id PK
        string code "Only 13 characters are allowed"
        string tagtext
        string Name "Only 50 characters are allowed"
        string status
    }

    SLIPS {
        integer id PK
        string number "slip number exp. 2025020199999"
        integer owner_id
        datetime startdate
        datetime enddate
        datetime createdate
        datetime updatedate
        string status
    }
    SLIP_DETAILS {
        integer id PK
        integer slips_id
        integer record_num
        string tagtext
        integer goods_id
        integer location_id
        datetime startdate
        datetime enddate
        datetime createdate
        datetime updatedate
        string status
    }

    RESERVATIONS {
        integer id PK
        string number "reserve number exp. 2025020199999"
        string type "DriveOn, Telephone"
        integer owner_id
        datetime startdate
        datetime enddate
        datetime createdate
        datetime updatedate
        string status
    }

    RESERVATIONS_LOG {
        integer id PK
        string number
        string type
        integer owner_id
        datetime startdate
        datetime enddate
        datetime createdate
        datetime updatedate
        string status
    }
```





