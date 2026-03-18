# Architecture

## 技術スタック

| カテゴリ | 技術 | バージョン | 選定理由 |
|---------|------|-----------|---------|
| 言語 | TypeScript | 5.x | 型安全性、IDE サポート |
| フレームワーク | React Native (Expo) | SDK 52+ | クロスプラットフォーム、managed workflow で iOS/Android 同時対応 |
| ナビゲーション | Expo Router | v4+ | ファイルベースルーティング、タブ・モーダル対応 |
| データベース | expo-sqlite | - | ローカル完結、SQL ベースで集計クエリが容易 |
| 状態管理 | React hooks (useState/useContext) | - | MVP 規模では十分、外部ライブラリ不要 |
| i18n | expo-localization + カスタム | - | OS 言語検出、軽量な実装 |
| テスト | Jest + React Native Testing Library | - | Expo 標準のテスト環境 |
| リント | ESLint + Prettier | - | コード品質・フォーマット統一 |
| パッケージマネージャ | pnpm | 9.x | 高速、ディスク効率 |
| Node バージョン管理 | fnm | - | 軽量・高速 |

## システム構成図

```
┌──────────────────────────────────────────────────┐
│                    Mobile App                     │
│                  (Expo Managed)                    │
│                                                    │
│  ┌──────────────────────────────────────────────┐ │
│  │  UI Layer                                     │ │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────────┐  │ │
│  │  │HomeScreen│ │DetailScr.│ │SummaryScreen │  │ │
│  │  └────┬─────┘ └────┬─────┘ └──────┬───────┘  │ │
│  │       │             │              │           │ │
│  │  ┌────┴─────┐ ┌─────┴────┐        │           │ │
│  │  │AddModal  │ │EditForm  │        │           │ │
│  │  └──────────┘ └──────────┘        │           │ │
│  └───────────────────┬───────────────┘           │ │
│                      │ ServiceFactory.get()       │ │
│  ┌───────────────────▼──────────────────────────┐ │
│  │  Service Layer                                │ │
│  │  ┌──────────────┐ ┌──────────────┐           │ │
│  │  │FoodItemSvc   │ │SummarySvc    │           │ │
│  │  └──────┬───────┘ └──────┬───────┘           │ │
│  └─────────┼────────────────┼───────────────────┘ │
│            │ interface      │ interface            │
│  ┌─────────▼────────────────▼───────────────────┐ │
│  │  Repository Layer                             │ │
│  │  ┌──────────────────┐ ┌────────────────────┐  │ │
│  │  │FoodItemRepository│ │FoodItemLogRepo.    │  │ │
│  │  └──────┬───────────┘ └──────┬─────────────┘  │ │
│  │  ┌──────┴────────────────────┘                │ │
│  │  │ CategoryRepository                         │ │
│  │  └──────┬─────────────────────────────────────┘ │
│  │         │                                       │
│  │  ┌──────▼─────────────────────────────────────┐ │
│  │  │  Database Layer                             │ │
│  │  │  ┌────────────┐ ┌───────────────────────┐  │ │
│  │  │  │DatabaseSvc │ │MigrationManager       │  │ │
│  │  │  │(expo-sqlite)│ │(auto-migrate on boot) │  │ │
│  │  │  └────────────┘ └───────────────────────┘  │ │
│  │  └─────────────────────────────────────────────┘ │
│  └──────────────────────────────────────────────────┘
│                                                    │
│  ┌──────────────────────────────────────────────┐ │
│  │  Cross-cutting                                │ │
│  │  ┌────────┐ ┌────────┐ ┌──────────────────┐  │ │
│  │  │  i18n  │ │ Theme  │ │  ServiceFactory  │  │ │
│  │  └────────┘ └────────┘ └──────────────────┘  │ │
│  └──────────────────────────────────────────────┘ │
└──────────────────────────────────────────────────┘
```

## レイヤードアーキテクチャ詳細

### UI Layer

- **責務**: ユーザー操作の受付、データの表示
- **依存**: Service Layer のみ（ServiceFactory 経由）
- **制約**: ビジネスロジックを持たない。データの加工・バリデーションは Service に委譲する

### Service Layer

- **責務**: ビジネスロジック、バリデーション、トランザクション管理
- **依存**: Repository Layer のインターフェースのみ
- **制約**: SQLite や具体的な永続化手段を直接参照しない

| サービス | 責務 |
|---------|------|
| `FoodItemService` | 食材の CRUD、残量更新、使い切り/廃棄処理 |
| `SummaryService` | サマリー集計（使い切り率、ネガティブ指標） |

### Repository Layer

- **責務**: データアクセスの抽象化
- **依存**: Database Layer
- **制約**: インターフェース（型定義）と実装（SQLite）を分離する

| リポジトリ | 対応テーブル |
|-----------|------------|
| `ICategoryRepository` / `CategoryRepository` | `category` |
| `IFoodItemRepository` / `FoodItemRepository` | `food_item` |
| `IFoodItemLogRepository` / `FoodItemLogRepository` | `food_item_log` |

### Database Layer

- **責務**: SQLite 接続管理、マイグレーション実行
- **依存**: expo-sqlite

| モジュール | 責務 |
|-----------|------|
| `DatabaseService` | DB 接続の初期化・提供 |
| `MigrationManager` | テーブル作成・シードデータ投入（起動時に自動実行） |

## ServiceFactory パターン

UI Layer から Service Layer へのアクセスは `ServiceFactory` を経由する。
これにより、テスト時にモックの Repository を注入可能にする。

```typescript
// 利用イメージ
const foodItemService = ServiceFactory.getFoodItemService();
const items = await foodItemService.getActiveItems();
```

## マイグレーション戦略

- アプリ起動時に `MigrationManager` がバージョンを確認し、未適用のマイグレーションを順次実行
- マイグレーションはバージョン番号で管理（v1, v2, ...）
- 初回起動時: テーブル作成 + カテゴリシードデータ投入
- MVP ではダウングレードは非対応

```
v1: CREATE TABLE category, food_item, food_item_log
    INSERT カテゴリシードデータ（7件）
```

## 技術的制約

| 制約 | 理由 |
|------|------|
| Expo managed workflow のみ | ネイティブモジュールの管理コストを回避 |
| ローカルストレージのみ（クラウド同期なし） | MVP スコープ、オフライン完結の要件 |
| 外部 API 呼び出しなし | ネットワーク依存を排除 |
| 状態管理ライブラリ不使用 | 4 画面・3 テーブルの規模では React hooks で十分 |

## ビルド・デプロイ

| 環境 | 方法 |
|------|------|
| 開発 | Expo Go / Development Build |
| デモ | iOS シミュレータ / Android エミュレータでの画面録画 |
| 配布（余裕があれば） | EAS Build → TestFlight / Internal Testing |
