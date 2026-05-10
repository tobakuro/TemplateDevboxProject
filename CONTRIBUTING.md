# Contributing Guide

以下のガイドラインを参照してください。

---

### 1. 環境構築

[`docs/1_setup_devbox.md`](./docs/1_setup_devbox.md) を参照してWSL + Devbox環境を整えてください。  
詰まったら [`docs/setup_error_handling.md`](./docs/setup_error_handling.md) へ。

### 2. プロジェクトのセットアップ

```bash
devbox shell
devbox run setup
```

### 3. 動作確認

```bash
devbox run dev
```

---

## 🌳 開発フロー

詳細は [`docs/git_workflow.md`](./docs/git_workflow.md) を参照してください。

```
1. Issueを確認 / 作成
2. develop から feature ブランチを切る
3. 開発・コミット
4. PR作成 (develop向け)
5. レビュー → マージ
```

---

## 📝 コミット前のチェックリスト

PRを作成する前に、以下が通ることを確認してください。

```bash
devbox run lint           # Lint
devbox run format:check   # フォーマット
devbox run typecheck      # 型チェック
devbox run test           # テスト
devbox run build          # ビルド
```

すべて通れば、CIも通る可能性が高いです。

---

## ✅ PR作成時のルール

### タイトル


```
feat: ログイン機能を追加
fix: 検索結果が空のときのエラーを修正
docs: READMEのセットアップ手順を更新
```

### 説明

`.github/PULL_REQUEST_TEMPLATE.md` のテンプレートに沿って記述してください。

### サイズ

- 目安: 500行以内
- 機能ごとに分割するのが理想 (大きすぎるとレビューされない)

### CI

- すべてのCIジョブが通っていること
- レビュー前に最低限 self-review すること

---

## 💬 質問・相談

- バグ報告: [Issues](../../issues/new?template=bug_report.yml)
- 機能要望: [Issues](../../issues/new?template=feature_request.yml)
- 実装相談: [Discussions](../../discussions)

---

## 📚 参考ドキュメント

| ドキュメント | 内容 |
|---|---|
| [docs/architecture.md](./docs/architecture.md) | システム構成 |
| [docs/directory_structure.md](./docs/directory_structure.md) | ディレクトリ構成 |
| [docs/coding_rules.md](./docs/coding_rules.md) | コーディング規約 |
| [docs/git_workflow.md](./docs/git_workflow.md) | Git運用 |
| [docs/env_variables.md](./docs/env_variables.md) | 環境変数 |
| [docs/adr/](./docs/adr/) | 技術選定の意思決定記録 (ADR) |
