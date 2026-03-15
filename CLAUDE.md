# CLAUDE.md

## プロジェクト概要

食材在庫管理アプリ「mottainai」。家庭の食材残量を可視化し、食品ロス削減を促す React Native (Expo) アプリ。EcoHack ハッカソン出展作品。

## コマンド

```bash
# 開発
pnpm start              # Expo 開発サーバー起動
pnpm ios                # iOS シミュレータで起動
pnpm android            # Android エミュレータで起動

# ビルド
pnpm build              # プロダクションビルド（EAS Build）

# テスト
pnpm test               # テスト全体実行
pnpm test -- path/to/test  # 単一テスト実行

# リント・フォーマット
pnpm lint               # ESLint 実行
pnpm format             # Prettier 実行

# 型チェック
pnpm typecheck          # TypeScript 型チェック実行
```

## 技術スタック

- 言語: TypeScript
- フレームワーク: React Native (Expo) 最新版
- データベース: expo-sqlite（ローカル完結）
- テスト: Jest + React Native Testing Library
- パッケージマネージャ: pnpm
- Node バージョン管理: fnm

## アーキテクチャ

レイヤードアーキテクチャを採用。各層はインターフェース経由で依存する。

```
┌─────────────────────────────┐
│   UI Layer                  │  screens/, components/
│   (Screens + Components)    │
└──────────────┬──────────────┘
               │ use services via ServiceFactory
┌──────────────▼──────────────┐
│   Service Layer             │  services/
│   (Business Logic)          │  - バリデーション後に操作
└──────────────┬──────────────┘  - インターフェース経由のみ
               │ use repository interfaces
┌──────────────▼──────────────┐
│   Repository Layer          │  repositories/
│   (Data Access)             │  - インターフェース定義
└──────────────┬──────────────┘  - SQLite 実装
               │
┌──────────────▼──────────────┐
│   Database Layer            │  services/database/
│   (SQLite / expo-sqlite)    │  - 起動時に自動マイグレーション
└─────────────────────────────┘
```

## コーディング規約

- コード・コミットメッセージ・PR・ドキュメントは日本語で記述する
- アプリの UI テキストはデフォルト英語、日本語も対応（i18n）
- 既存コードのスタイルに従う
- コード変更後は必ずリント・型チェックを通す
- セキュリティを考慮する（入力バリデーションなど）
- メソッド追加時は必ず対応するテストを追加する
- テストが成功する状態で PR を作成する
- Service 層は Repository インターフェース経由でのみデータにアクセスする

## ドキュメント構造

### 永続的ドキュメント（`docs/`）

プロジェクト全体の設計を定義する恒久的なドキュメント。基本設計が変わらない限り更新しない。

| ファイル | 内容 |
|---------|------|
| product-requirements.md | プロダクト要求定義（ビジョン、ユーザーストーリー、機能要件、非機能要件） |
| functional-design.md | 機能設計（データモデル、コンポーネント設計、画面遷移、ER図） |
| architecture.md | 技術仕様（技術スタック、システム構成、技術的制約） |
| repository-structure.md | リポジトリ構造定義（ディレクトリ構成、配置ルール） |
| development-guidelines.md | 開発ガイドライン（コーディング規約、命名規則、テスト規約、Git規約） |
| glossary.md | 用語定義（ドメイン用語、英語・日本語対応表、コード上の命名） |

### 作業単位のドキュメント（`.steering/`）

個別の開発作業ごとに作成する一時的なドキュメント。

```
.steering/[YYYYMMDD]-[開発タイトル]/
  ├── requirements.md   # 要求内容・受け入れ条件
  ├── design.md         # 実装設計・影響範囲
  └── tasklist.md       # タスクリスト・進捗管理
```

## 開発プロセス

### 初回セットアップ

1. `docs/` 配下の永続的ドキュメントを順に作成する
2. **1ファイルごとに確認・承認を得てから次へ進む**
3. `.steering/[YYYYMMDD]-initial-implementation/` を作成し、初回実装のドキュメントを配置する
4. 環境セットアップ後、`tasklist.md` に基づいて実装する

### 機能追加・修正

1. `docs/` への影響を確認し、必要なら更新する
2. `.steering/[YYYYMMDD]-[タイトル]/` を作成する
3. requirements.md → design.md → tasklist.md の順に作成する（**各ファイル作成後に承認を得る**）
4. `tasklist.md` に基づいて実装する

### 図表ルール

- 図表は関連する `docs/` 内のドキュメントに直接記載する（独立フォルダは作らない）
- Mermaid 記法を推奨、シンプルな図は ASCII アートも可
- 画像が必要な場合は `docs/images/` に PNG/SVG で配置

## サブエージェント連携

### レポート駆動ルール

サブエージェント（test-analyzer, code-review 等）を呼び出す際は、レポート駆動で連携する。

- サブエージェントの実行結果は**マークダウンファイルとして書き出す**
- デフォルトの出力先: `.claude/reports/`
- 作業中の steering ディレクトリがある場合は `{steering_dir}/reports/` に出力させる
- ファイル名: `{エージェント名}-{YYYYMMDD-HHmmss}.md`
