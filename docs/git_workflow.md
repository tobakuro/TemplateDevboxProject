<!-- Gitのブランチ戦略・コミットメッセージ規約・PR運用ルールを記載 -->

# Git運用ルール

このドキュメントでは、本プロジェクトでのGit運用ルールを記載します。  
プロジェクト規模に応じて、必要部分のみ採用・カスタマイズしてください。

---

## 🌳 ブランチ戦略

### 基本ブランチ

| ブランチ | 役割 |
|---|---|
| `main` | 本番リリース用。常にデプロイ可能な状態を保つ |
| `develop` | 開発用の統合ブランチ。featureブランチをここにマージ |

### 作業ブランチ

| プレフィックス | 用途 | 例 |
|---|---|---|
| `feat/` | 新機能開発 | `feat/login-form` |
| `fix/` | バグ修正 | `fix/login-error` |
| `refactor/` | リファクタリング | `refactor/api-client` |
| `docs/` | ドキュメント変更のみ | `docs/update-readme` |
| `chore/` | ビルド設定・依存更新など | `chore/update-deps` |

### 命名ルール

```
<type>/<issue番号>/<簡潔な説明>
例: feat/12/user-login
```

---

## 📝 コミットメッセージ規約 (Conventional Commits)

```
<type>: <subject>

[optional body]

[optional footer]
```

### type一覧

| type | 用途 |
|---|---|
| `feat` | 新機能追加 |
| `fix` | バグ修正 |
| `docs` | ドキュメントのみの変更 |
| `style` | フォーマット修正 (機能に影響しない) |
| `refactor` | リファクタリング |
| `test` | テスト追加・修正 |
| `chore` | ビルド・依存関係など |
| `perf` | パフォーマンス改善 |

### 例

```
feat: ログイン機能を追加

- メール+パスワード認証
- ログイン後のリダイレクト処理

closes #12
```

```
fix: API呼び出し時のNull参照エラーを修正
```

---

## 🔄 開発フロー

```
1. issueを作成 (or アサインされる)
   ↓
2. develop から feat ブランチを切る
   git checkout develop
   git pull
   git checkout -b feat/12/xxx
   ↓
3. 開発・コミット
   ↓
4. リモートへpush
   git push -u origin feat/12/xxx
   ↓
5. PRを作成 (develop向け)
   ↓
6. レビュー → マージ
   ↓
7. ローカルのfeatブランチを削除
```

---

## ✅ PR作成時のチェック

- [ ] PRテンプレートを埋めている
- [ ] 関連Issueを `closes #<番号>` で紐付けている
- [ ] CIが通っている
- [ ] レビュアーをアサインしている
- [ ] 動作確認のスクショ・動画を添付している (UI変更がある場合)

---

## 🚫 禁止事項

- `main` / `develop` への直接push
- `git push --force` (個人ブランチを除く)
- 巨大なPR (目安: 500行超は分割を検討)
