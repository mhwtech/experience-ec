# システム構成（アーキテクチャ）

## 概要

Shopify を基盤に、**イベントの作成・申込・支払**と**イベント後の参加者限定商品オファー・購入**を扱う構成です。

**推奨方針（クライアント提案用）**: ユーザーに「2つのサイト」と感じさせないため、**ストアフロントは一つの統一ストア**（テーマアプリ拡張またはカスタムセクションでイベント一覧と「あなた向け商品」を同一レイアウトで表示）とし、**管理はプラグイン中心**（イベントアプリ・Locksmith・メール/SMS アプリ）とする一案を推奨します。

---

## 1. 全体構成図

```mermaid
flowchart TB
  subgraph admin [管理画面]
    A1[イベント作成]
    A2[日時・場所・料金・定員設定]
    A3[イベント後オファー作成]
    A4[商品・対象選択・通知送信]
  end
  subgraph store [Shopify ストア]
    P[商品]
    C[チェックアウト]
    Cust[顧客]
    Ord[注文]
  end
  subgraph user [ユーザー]
    U1[イベント一覧・詳細閲覧]
    U2[申込・支払]
    U3[通知受信]
    U4[限定商品購入]
  end
  A1 --> A2
  A2 --> P
  U1 --> U2
  U2 --> C
  C --> Ord
  Ord --> Cust
  A3 --> A4
  A4 --> U3
  U3 --> U4
  U4 --> C
  U4 --> P
```

- **管理**: イベント作成 → ストアにチケット商品が紐づく。イベント後はオファー（商品・対象・通知）を設定。
- **ユーザー**: イベント閲覧 → 申込・支払（同一チェックアウト）→ 後日通知 → 限定商品購入（同一チェックアウト）。

---

## 2. データの流れ（全体）

```mermaid
flowchart LR
  subgraph input [入力]
    EventDef[イベント定義]
    OfferDef[オファー定義]
  end
  subgraph shopify [Shopify]
    Products[商品]
    Checkout[チェックアウト]
    Customers[顧客]
    Orders[注文]
  end
  subgraph app [カスタムアプリ]
    EventDB[イベントDB]
    Attendees[参加者一覧]
    Tags[顧客タグ]
  end
  EventDef --> EventDB
  EventDB --> Products
  Orders --> Attendees
  Attendees --> Tags
  OfferDef --> Products
  OfferDef --> Attendees
  Tags --> Checkout
  Products --> Checkout
  Customers --> Checkout
```

- イベント定義はアプリで保持し、チケット商品を Shopify と連携。
- 注文（orders）から参加者を抽出し、タグで限定表示・通知対象を制御。
- オファーは「どの商品を」「誰に」を定義し、同一チェックアウトで購入可能にする。

---

## 3. コンポーネント関係（推奨: 統一ストアフロント ＋ プラグイン中心の管理）

```mermaid
flowchart TB
  subgraph storefront [ストアフロント]
    Theme[Shopify テーマ]
    Ext[テーマアプリ拡張]
    EventList[イベント一覧]
    EventDetail[イベント詳細]
    OfferProduct[限定商品ページ]
  end
  subgraph checkout [決済]
    Cart[カート]
    ShopifyCheckout[Shopify チェックアウト]
  end
  subgraph customApp [カスタムアプリ]
    AdminUI[管理UI]
    API[Admin GraphQL API]
    Webhooks[Webhook]
    Notify[通知送信]
  end
  subgraph shopifyCore [Shopify 基盤]
    Products[商品]
    Customers[顧客]
    Orders[注文]
  end
  Theme --> Ext
  Ext --> EventList
  Ext --> EventDetail
  EventList --> Cart
  EventDetail --> Cart
  OfferProduct --> Cart
  Cart --> ShopifyCheckout
  ShopifyCheckout --> Orders
  AdminUI --> API
  API --> Products
  API --> Customers
  API --> Orders
  Orders --> Webhooks
  Webhooks --> customApp
  customApp --> Notify
```

- ストアフロント: テーマ + アプリ拡張（またはカスタムセクション）でイベント一覧・詳細・「あなた向け商品」を同一レイアウトで表示。申込・購入は同一チェックアウト。データはイベントアプリ・Locksmith 等のプラグインから利用。
- 管理: プラグイン（イベントアプリ・Locksmith・メール/SMS アプリ）中心。必要に応じ小規模な管理専用アプリでオファー作成・通知を一括。

---

## 4. ストアフロントの見え方（参加前・参加後）

同じストアフロントで、**参加前**と**参加後**で表示が変わります（レイアウトは同じで、条件付きブロックが増えるイメージ）。

| 状態 | ユーザーが見るもの |
|------|---------------------|
| **参加前** | イベント一覧（と詳細・申込）のみ |
| **参加後** | イベント一覧 ＋ そのユーザー向けに提案／限定表示される商品（あなたへのおすすめ・参加者限定） |

```mermaid
flowchart LR
  subgraph before [参加前]
    B1[イベント一覧のみ]
  end
  subgraph after [参加後]
    A1[イベント一覧]
    A2["あなたへのおすすめ / 参加者限定商品"]
  end
  before --> after
```

- ストアそのものは別ではなく、**参加後**のユーザーには「イベント一覧」に加えて「あなた向け商品」のブロックが表示される想定です。

---

## 5. 図の一覧（他ドキュメント）

| ドキュメント | 内容 |
|--------------|------|
| [02-tech-details.md](02-tech-details.md) | プラグイン vs 開発量・技術スタック図・プラグイン推奨 |
| [03-diagrams.md](03-diagrams.md) | ストアフロント表示の違い・管理／ユーザーフロー・画面遷移・参加者と限定商品の関係図 |
