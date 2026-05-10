<!-- プロジェクトで使用するGitHubラベル一覧と一括登録用コマンド -->

# GitHubラベル定義

このドキュメントでは、本プロジェクトで使用するGitHubラベルを定義します。  
PRラベラー (`.github/workflows/labeler.yml`) や Issue テンプレートで参照されます。

---

## 🏷️ ラベル一覧

### タイプ系

| ラベル | 色 | 用途 |
|---|---|---|
| `enhancement` | `#a2eeef` | 新機能・機能改善 |
| `bug` | `#d73a4a` | バグ |
| `documentation` | `#0075ca` | ドキュメントのみの変更 |
| `refactor` | `#fbca04` | リファクタリング |
| `performance` | `#ff7619` | パフォーマンス改善 |
| `test` | `#bfd4f2` | テスト関連 |
| `chore` | `#cccccc` | ビルド・依存関係など |

### 領域系

| ラベル | 色 | 用途 |
|---|---|---|
| `dependencies` | `#0366d6` | 依存パッケージ更新 |
| `ci/cd` | `#1d76db` | CI/CD関連 |
| `config` | `#5319e7` | 設定ファイル変更 |
| `security` | `#ee0701` | セキュリティ関連 |

### サイズ系 (PRサイズで自動付与)

| ラベル | 色 | 行数 |
|---|---|---|
| `size/XS` | `#3CBF00` | 〜10行 |
| `size/S` | `#5D9801` | 11〜100行 |
| `size/M` | `#7F7203` | 101〜300行 |
| `size/L` | `#A14C05` | 301〜500行 |
| `size/XL` | `#C32607` | 500行〜 |

### ステータス系

| ラベル | 色 | 用途 |
|---|---|---|
| `wip` | `#fbca04` | Work In Progress (マージしないで) |
| `stale` | `#cccccc` | 放置されている (Stale Botが付与) |
| `pinned` | `#0e8a16` | 自動クローズ対象外 |
| `good first issue` | `#7057ff` | 初心者向け |
| `help wanted` | `#008672` | 助けが必要 |

---

## ⚡ 一括登録コマンド

リポジトリ作成後、以下のコマンドで全ラベルを一括登録できます。

```bash
# gh CLIにログイン済みであること
# devbox shell 内で実行 (devbox.json の packages に "gh" を追加)

# タイプ系
gh label create "enhancement" --color "a2eeef" --description "新機能・機能改善" --force
gh label create "bug" --color "d73a4a" --description "バグ" --force
gh label create "documentation" --color "0075ca" --description "ドキュメントのみの変更" --force
gh label create "refactor" --color "fbca04" --description "リファクタリング" --force
gh label create "performance" --color "ff7619" --description "パフォーマンス改善" --force
gh label create "test" --color "bfd4f2" --description "テスト関連" --force
gh label create "chore" --color "cccccc" --description "ビルド・依存関係など" --force

# 領域系
gh label create "dependencies" --color "0366d6" --description "依存パッケージ更新" --force
gh label create "ci/cd" --color "1d76db" --description "CI/CD関連" --force
gh label create "config" --color "5319e7" --description "設定ファイル変更" --force
gh label create "security" --color "ee0701" --description "セキュリティ関連" --force

# サイズ系
gh label create "size/XS" --color "3CBF00" --description "〜10行" --force
gh label create "size/S" --color "5D9801" --description "11〜100行" --force
gh label create "size/M" --color "7F7203" --description "101〜300行" --force
gh label create "size/L" --color "A14C05" --description "301〜500行" --force
gh label create "size/XL" --color "C32607" --description "500行〜" --force

# ステータス系
gh label create "wip" --color "fbca04" --description "Work In Progress" --force
gh label create "stale" --color "cccccc" --description "放置されている" --force
gh label create "pinned" --color "0e8a16" --description "自動クローズ対象外" --force
gh label create "good first issue" --color "7057ff" --description "初心者向け" --force
gh label create "help wanted" --color "008672" --description "助けが必要" --force
```
