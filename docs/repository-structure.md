# Repository Structure

## ディレクトリ構成

```
mottainai/
├── app/                          # Expo Router ページ（ファイルベースルーティング）
│   ├── _layout.tsx               # ルートレイアウト
│   ├── (tabs)/                   # タブナビゲーション
│   │   ├── _layout.tsx           # タブレイアウト
│   │   ├── index.tsx             # ホーム（在庫一覧）
│   │   └── summary.tsx           # サマリー
│   ├── items/
│   │   └── [id].tsx              # 食材詳細 / 更新
│   └── add.tsx                   # 食材追加（モーダル）
│
├── components/                   # 再利用可能な UI コンポーネント
│   ├── CategoryFilter.tsx
│   ├── CategoryIcon.tsx
│   ├── ConfirmDialog.tsx
│   ├── EmptyState.tsx
│   ├── FoodItemCard.tsx
│   ├── QuantityBar.tsx
│   ├── QuantitySlider.tsx
│   ├── StatsCard.tsx
│   ├── Toast.tsx
│   └── UseUpRateCard.tsx
│
├── services/                     # Service Layer（ビジネスロジック）
│   ├── FoodItemService.ts
│   ├── SummaryService.ts
│   ├── ServiceFactory.ts         # サービスの生成・DI
│   └── database/                 # Database Layer
│       ├── DatabaseService.ts    # DB 接続管理
│       └── MigrationManager.ts   # マイグレーション管理
│
├── repositories/                 # Repository Layer（データアクセス）
│   ├── interfaces/               # リポジトリインターフェース
│   │   ├── ICategoryRepository.ts
│   │   ├── IFoodItemRepository.ts
│   │   └── IFoodItemLogRepository.ts
│   ├── CategoryRepository.ts     # SQLite 実装
│   ├── FoodItemRepository.ts
│   └── FoodItemLogRepository.ts
│
├── types/                        # 型定義
│   └── index.ts                  # エンティティ型、enum 等
│
├── i18n/                         # 多言語対応
│   ├── index.ts                  # i18n 初期化・言語切り替え
│   ├── en.ts                     # 英語（デフォルト）
│   └── ja.ts                     # 日本語
│
├── hooks/                        # カスタムフック（必要に応じて追加）
│   └── (例: useFoodItems.ts, useDatabase.ts)
│
├── constants/                    # 定数定義
│   └── index.ts                  # カラーコード等（※カテゴリ実データは MigrationManager で管理）
│
├── __tests__/                    # テスト
│   ├── services/
│   │   ├── FoodItemService.test.ts
│   │   └── SummaryService.test.ts
│   ├── repositories/
│   │   ├── CategoryRepository.test.ts
│   │   ├── FoodItemRepository.test.ts
│   │   └── FoodItemLogRepository.test.ts
│   └── components/
│       ├── FoodItemCard.test.tsx
│       ├── QuantityBar.test.tsx
│       └── ...
│
├── docs/                         # 永続的ドキュメント
│   ├── product-requirements.md
│   ├── functional-design.md
│   ├── architecture.md
│   ├── repository-structure.md
│   ├── development-guidelines.md
│   └── glossary.md
│
├── .steering/                    # 作業単位ドキュメント（.gitignore 対象）
│
├── CLAUDE.md                     # Claude Code 用プロジェクト指示
├── .gitignore
├── app.json                      # Expo 設定
├── tsconfig.json                 # TypeScript 設定
├── package.json
├── pnpm-lock.yaml
├── .node-version                 # fnm 用 Node バージョン指定
├── jest.config.js                # Jest 設定
├── .eslintrc.js                  # ESLint 設定
└── .prettierrc                   # Prettier 設定
```

## 配置ルール

### `app/`

- Expo Router のファイルベースルーティングに従う
- 画面コンポーネントのみ配置（ビジネスロジックは含めない）
- レイアウトファイル（`_layout.tsx`）でナビゲーション構造を定義

### `components/`

- 画面に依存しない再利用可能な UI コンポーネントを配置
- 1 ファイル = 1 コンポーネント（`PascalCase.tsx`）
- ビジネスロジックを持たない（表示とユーザー操作のコールバックのみ）

### `services/`

- ビジネスロジックを担当するサービスクラスを配置
- Repository インターフェース経由でのみデータにアクセス
- `ServiceFactory.ts` でサービスの生成と依存注入を管理
- `database/` サブディレクトリに DB 関連のサービスを配置

### `repositories/`

- `interfaces/` にインターフェース（型定義）を配置
- ルート直下に SQLite 実装を配置
- インターフェースと実装を分離し、テスト時のモック差し替えを可能にする

### `types/`

- エンティティ型（`FoodItem`, `Category`, `FoodItemLog` 等）を定義
- アプリ全体で共有する型を集約

### `i18n/`

- 言語ごとに翻訳ファイルを分離（`en.ts`, `ja.ts`）
- キーは英語ベースのドット記法（例: `home.title`, `item.usedUp`）

### `hooks/`

- 画面間で共有するカスタムフックを配置（例: `useFoodItems`, `useDatabase`）
- MVP 初期では不要な場合もあるため、必要に応じて追加する
- 1 ファイル = 1 フック（`use{Name}.ts`）

### `constants/`

- UI 用の定数（カラーコード、スタイル値等）を配置
- カテゴリの実データ（シードデータ）は `MigrationManager` で管理し、ここには置かない
- カテゴリの型定義は `types/` に配置

### `__tests__/`

- ソースコードのディレクトリ構造をミラーリング
- テストファイル名: `{対象ファイル名}.test.ts(x)`
- Service / Repository は単体テスト、Components は描画テスト

### `docs/`

- Git 管理下の永続的ドキュメント
- 基本設計が変わらない限り更新しない
