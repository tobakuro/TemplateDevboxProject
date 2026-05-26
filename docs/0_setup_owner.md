<!-- テンプレートを使って新しいリポジトリを立ち上げるオーナー向けのセットアップ手順 -->

# 0. オーナー向けセットアップ手順

このドキュメントでは、テンプレートをダウンロードしてから、チームメンバーが開発を始められる状態にするまでの手順を記載します。

> チームメンバー向けのセットアップ手順は [`1_setup_devbox.md`](./1_setup_devbox.md) を参照してください。

---

## 全体の流れ

```
STEP 1. テンプレートの取得
STEP 2. GitHubリポジトリの作成・接続
STEP 3. テンプレートのカスタマイズ
STEP 4. GitHubリポジトリの初期設定
STEP 5. チームメンバーへの共有
```

---

## STEP 1. テンプレートの取得

### 1-1. ZIPをWSL上に直接ダウンロード

> ⚠️ **CドライブへのダウンロードをWSLにコピーする方法は避けてください。**
> Cドライブを経由すると改行コードがCRLFに変換され、git差分が大量に発生します。
> また展開時に `Zone.Identifier` ファイルが生成されるトラブルもあります。

WSL (Ubuntu) のターミナルで以下を実行します。

```bash
# ホームディレクトリ配下にgithubフォルダを作成（既にある場合は不要）
mkdir -p ~/github
cd ~/github

# ZIPをWSL上に直接ダウンロードして展開
wget https://github.com/tobakuro/TemplateDevboxProject/archive/refs/heads/main.zip
unzip main.zip

# フォルダ名を自分のプロジェクト名に変更
mv TemplateDevboxProject-main <your-project-name>
cd <your-project-name>

# ダウンロードしたZIPを削除
rm ../main.zip
```

### 1-2. gitリポジトリとして初期化

ZIPにはgit情報が含まれていないため、新規に初期化します。

```bash
git init
git add .
git commit -m "feat: 初回コミット"
```

---

## STEP 2. GitHubリポジトリの作成・接続

### 2-1. GitHubにリポジトリを作成

GitHub上で新しいリポジトリを作成します。

- Repository name: `<your-project-name>`
- Visibility: PublicまたはPrivate
- **Initialize this repository with:** は何もチェックしない（READMEやgitignoreを自動生成しない）

### 2-2. リモートリポジトリと接続

```bash
git remote add origin https://github.com/<your-name>/<your-project-name>.git
git branch -M main
git push -u origin main
```

---

## STEP 3. テンプレートのカスタマイズ

以下のファイルをプロジェクトに合わせて編集してください。

### devbox.json — 使用するパッケージとスクリプトを埋める

```json
{
  "packages": [
    "git@latest",
    "gh@latest",
    "nodejs@20"  // ← プロジェクトで使う言語・ツールを追加
  ],
  "shell": {
    "scripts": {
      "setup": ["pnpm install"],      // ← 各TODO行を実際のコマンドに書き換える
      "dev":   ["pnpm dev"],
      "lint":  ["pnpm lint"],
      ...
    }
  }
}
```

利用可能なパッケージは [Nixhub Search](https://www.nixhub.io/) から検索可能。

### .github/CODEOWNERS — レビュアーを自分のGitHubアカウントに変更

```
*       @<your-github-username>
/docs/  @<your-github-username>
/.github/ @<your-github-username>
```

### .github/dependabot.yml — 使わないエコシステムをコメントアウト

Node.jsを使わない場合は `npm` セクションをコメントアウトまたは削除してください。

### .env.example — プロジェクトで必要な環境変数を記載

### docs/ 配下のドキュメント — プロジェクトの内容に合わせて更新

| ファイル | 更新内容 |
|---|---|
| [2_setup.md](./2_setup.md) | 依存インストール・サーバー起動手順を記載 |
| [architecture.md](./architecture.md) | システム構成を記載 |
| [directory_structure.md](./directory_structure.md) | ディレクトリ構成を記載 |
| [coding_rules.md](./coding_rules.md) | コーディング規約を記載 |
| [env_variables.md](./env_variables.md) | 環境変数の説明を記載 |
| [commands.md](./commands.md) | よく使うコマンドを記載 |

カスタマイズが完了したらコミット・pushしてください。

```bash
git add .
git commit -m "chore: テンプレートをプロジェクト用にカスタマイズ"
git push
```

---

## STEP 4. GitHubリポジトリの初期設定

### 4-1. ラベルの一括登録

gh CLIでGitHubにログインし、ラベルを一括登録します。

```bash
# devbox shellに入る
devbox shell

# GitHubにログイン（まだの場合）
gh auth login

# ラベルを一括登録（コマンドはgithub_labels.mdを参照）
```

> 詳細は [`github_labels.md`](./github_labels.md) を参照してください。

### 4-2. Branch Protection の設定

GitHubリポジトリの **Settings → Branches → Add branch ruleset** から以下を設定します。

| 設定項目 | 推奨値 |
|---|---|
| 対象ブランチ | `main` |
| Require a pull request before merging | ON |
| Require approvals | 1人以上 |
| Require status checks to pass | ON（CIジョブを指定） |
| Do not allow bypassing the above settings | ON |

### 4-3. Dependabotの有効化

**Settings → Security → Dependabot alerts** をONにしてください。

---

## STEP 5. チームメンバーへの共有

以下を共有すればメンバーがすぐに開発を始められます。

- [ ] GitHubリポジトリへの招待（Settings → Collaborators）
- [ ] リポジトリURLの共有
- [ ] [`1_setup_devbox.md`](./1_setup_devbox.md) を参照してもらう

---

## チェックリスト

- [ ] ZIPをWSL上で直接ダウンロード・展開した
- [ ] `git init` → 初回コミット → GitHubへpush済み
- [ ] `devbox.json` のpackagesとscriptsを埋めた
- [ ] `CODEOWNERS` を自分のアカウントに変更した
- [ ] `dependabot.yml` を使用するエコシステムに合わせた
- [ ] `.env.example` を整備した
- [ ] GitHubラベルを一括登録した
- [ ] Branch Protectionを設定した
- [ ] メンバーをリポジトリに招待した
