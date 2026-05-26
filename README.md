# TemplateDevboxProject

Devboxを用いた開発プロジェクトのテンプレートリポジトリです。
品質ゲート・セキュリティ・CI/CD・運用ドキュメントが揃った状態で開発を始められます。

> リポジトリ: https://github.com/tobakuro/TemplateDevboxProject

---

## 含まれているもの

### 開発環境
- **Devbox** - プロジェクトごとに独立した開発環境 (Nixベース)
- **devbox scripts** - lint/format/typecheck/test/build を統一コマンドで実行 (各自TODOを埋める)
- **`.env.example`** - 環境変数のサンプル

### 品質ゲート (CI)
- **Lint** - コーディング規約チェック
- **Format Check** - フォーマッタ整合性チェック
- **Type Check** - 型チェック
- **Test** - 単体・結合テスト + カバレッジ
- **Build** - ビルドが通ることを確認
- **並列ジョブ実行** + **古い実行の自動キャンセル**

### セキュリティ
- **Gitleaks** - コミットされた機密情報の検出
- **CodeQL** - GitHub標準のコード脆弱性スキャン
- **Dependabot** - 依存パッケージの脆弱性自動PR

### CI/CD強化
- **PR自動ラベリング** (パス/ブランチ/サイズ)
- **Release Drafter** - PRラベルからリリースノート自動生成
- **Stale Bot** - 放置Issue/PR自動クローズ
- **CD ワークフロー** - main/タグでのデプロイ枠

### GitHub設定
- **Issueテンプレ** (bug/feature/task)
- **PRテンプレ**
- **CODEOWNERS** - ディレクトリ別レビュアー自動アサイン

### ドキュメント
- セットアップ手順 (WSL〜Devbox shellまで)
- エラーハンドリング集 (過去経験ベース)
- Git運用ルール (ブランチ戦略・コミット規約)
- コーディング規約・環境変数管理
- CONTRIBUTING

---

## はじめに

### 1. リポジトリの取得

GitHub上で「**Use this template**」ボタンから新規リポジトリを作成。

### 2. 環境構築 (WSL → Devbox)

[docs/1_setup_devbox.md](./docs/1_setup_devbox.md)
詰まったら → [docs/setup_error_handling.md](./docs/setup_error_handling.md)

### 3. プロジェクト固有のセットアップ

[docs/2_setup.md](./docs/2_setup.md)

---

## ドキュメント一覧

### 開発者向け
| ファイル | 内容 |
|---|---|
| [1_setup_devbox.md](./docs/1_setup_devbox.md) | WSL〜Devbox shellまで |
| [2_setup.md](./docs/2_setup.md) | プロジェクト固有のセットアップ |
| [commands.md](./docs/commands.md) | よく使うコマンド集 |
| [setup_error_handling.md](./docs/setup_error_handling.md) | WSL/Devbox/Nixのエラー対処 |

### プロジェクト全体
| ファイル | 内容 |
|---|---|
| [architecture.md](./docs/architecture.md) | システム構成 |
| [directory_structure.md](./docs/directory_structure.md) | ディレクトリ構成 |
| [coding_rules.md](./docs/coding_rules.md) | コーディング規約 |
| [git_workflow.md](./docs/git_workflow.md) | Git運用ルール |
| [env_variables.md](./docs/env_variables.md) | 環境変数管理 |
| [github_labels.md](./docs/github_labels.md) | ラベル定義と一括登録コマンド |

### コミュニティ
| ファイル | 内容 |
|---|---|
| [CONTRIBUTING.md](./CONTRIBUTING.md) | 貢献ガイド |

---

## リポジトリ構成

```
.
├── .github/
│   ├── ISSUE_TEMPLATE/          # bug/feature/task テンプレ
│   ├── workflows/               # GitHub Actions
│   │   ├── ci.yml               # 品質ゲート (lint/format/type/test/build)
│   │   ├── cd.yml               # デプロイ
│   │   ├── codeql.yml           # コード脆弱性スキャン
│   │   ├── secret-scan.yml      # gitleaks
│   │   ├── labeler.yml          # PR自動ラベリング
│   │   ├── release-drafter.yml  # リリースノート自動生成
│   │   └── stale.yml            # 放置Issue/PR自動クローズ
│   ├── CODEOWNERS               # ディレクトリ別レビュアー
│   ├── dependabot.yml           # 依存自動更新
│   ├── labeler.yml              # ラベル付与ルール
│   ├── release-drafter-config.yml
│   └── PULL_REQUEST_TEMPLATE.md
├── docs/
│   ├── 1_setup_devbox.md
│   ├── 2_setup.md
│   ├── commands.md
│   ├── setup_error_handling.md
│   ├── architecture.md
│   ├── directory_structure.md
│   ├── coding_rules.md
│   ├── git_workflow.md
│   ├── env_variables.md
│   └── github_labels.md
├── .env.example
├── .gitattributes
├── .gitignore
├── .gitleaks.toml               # Secret検知の除外設定
├── devbox.json                  # Devbox設定 + scripts
├── README.md
└── CONTRIBUTING.md
```

---

## 使用技術の追加方法

`devbox.json` の `packages` 配列に必要なパッケージを追記:

```json
{
  "packages": [
    "git@latest",
    "gh@latest",
    "nodejs@20",
    "python@3.12"
  ]
}
```

利用可能なパッケージは [Nixhub Search](https://www.nixhub.io/) から検索可能。

## 統一コマンド

`devbox.json` の `scripts` をプロジェクトに合わせて埋めると、CI と同じコマンドでローカル確認できます:

```bash
devbox run setup        # 依存インストール
devbox run dev          # 開発サーバー起動
devbox run lint         # Lint
devbox run lint:fix     # Lint自動修正
devbox run format       # フォーマット
devbox run format:check # フォーマットチェック (CI用)
devbox run typecheck    # 型チェック
devbox run test         # テスト
devbox run build        # ビルド
```
