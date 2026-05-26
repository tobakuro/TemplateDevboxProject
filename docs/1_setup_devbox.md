<!-- WSLのインストールからDevbox shellに入るまでの環境構築手順を記載 -->

# 1. Devbox 環境セットアップ手順

このドキュメントでは、Windows上で **WSL2 (Ubuntu)** をセットアップし、**Devbox** をインストールして `devbox shell` に入るまでの流れを記載します。

> ⚠️ 途中でエラーが出たら → [`setup_error_handling.md`](./setup_error_handling.md) を参照してください。  
> 過去に踏んだエラー (systemd未有効、nix-daemon未起動、`/mnt/c/`配下による遅延 等) と解決策をまとめてあります。

---

## 📋 前提条件

- Windows 10 (バージョン2004以降, ビルド19041以降) または Windows 11
- 管理者権限のあるユーザーアカウント
- インターネット接続

---

## STEP 1. WSL2 のインストール

### 1-1. PowerShellを管理者権限で起動

スタートメニューで「PowerShell」を検索 → 右クリック → 「**管理者として実行**」を選択。

### 1-2. WSLのインストール

以下のコマンドを実行します。

```powershell
wsl --install
```

このコマンドで以下が自動的に行われます。

- WSL2の有効化
- 仮想マシンプラットフォームの有効化
- Linuxカーネルの最新化
- Ubuntuディストリビューションのインストール

### 1-3. PCの再起動

インストール完了後、**PCを再起動**してください。

### 1-4. Ubuntuの初期設定

再起動後、自動的にUbuntuが起動します(起動しない場合はスタートメニューから「Ubuntu」を起動)。  
初回起動時にユーザー名とパスワードを設定してください。

```
Enter new UNIX username: <好きなユーザー名>
New password: <パスワード>
Retype new password: <パスワード再入力>
```

### 1-5. WSLが動作していることを確認

PowerShellで以下を実行し、Ubuntuがインストールされていることを確認します。

```powershell
wsl -l -v
```

出力例:

```
  NAME      STATE           VERSION
* Ubuntu    Running         2
```

### 1-6. systemd の有効化 ⭐ **重要**

DevboxはNixを内部で利用しますが、Nixは `nix-daemon` というsystemdサービスを必要とします。  
WSLではsystemdがデフォルトで無効なので、**ここで有効化しないと後でハマります**。

WSL (Ubuntu) のターミナルで `/etc/wsl.conf` を編集:

```bash
sudo nano /etc/wsl.conf
```

以下を追記 (既存の `[boot]` セクションがあれば `systemd=true` のみ追記):

```ini
[boot]
systemd=true
```

`Ctrl+O` → `Enter` で保存、`Ctrl+X` で終了。

**PowerShellに戻って** WSLを完全シャットダウン:

```powershell
wsl --shutdown
```

> ⚠️ Docker Desktopを起動している場合は **先に終了** してから実行してください。  
> Dockerが掴んでいると `--shutdown` が中途半端に終わります。

WSLを再度開いて確認:

```bash
ps -p 1 -o comm=
```

`systemd` と表示されれば成功。`init` と出る場合は [`setup_error_handling.md` E-WSL-02](./setup_error_handling.md#e-wsl-02-wsl---shutdown-してもsystemdが起動しない) を参照。

---

## STEP 2. Ubuntu の初期更新

WSL内のUbuntuターミナルを開き、パッケージを最新化します。

```bash
sudo apt update && sudo apt upgrade -y
```

必要に応じて、開発でよく使うツールも入れておきます。

```bash
sudo apt install -y curl git build-essential
```

---

## STEP 3. Devbox のインストール

### 3-1. Devboxインストールコマンドの実行

WSL (Ubuntu) のターミナルで以下を実行します。

```bash
curl -fsSL https://get.jetify.com/devbox | bash
```

途中で `sudo` のパスワードを求められた場合は、Ubuntuのパスワードを入力してください。

### 3-2. インストールの確認

```bash
devbox version
```

バージョンが表示されればインストール完了です。

### 3-3. Nix のインストール (初回のみ)

Devboxは内部でNixを使用します。初回 `devbox shell` 実行時に自動でNixのインストールが行われます。  
プロンプトで `Y` を選択して進めてください。

### 3-4. nix-daemon の自動起動を有効化 ⭐ **重要**

WSLを再起動するたびに手動で起動するのは面倒なので、**自動起動を有効化**しておきます。

```bash
sudo systemctl start nix-daemon
sudo systemctl enable nix-daemon
```

確認:

```bash
systemctl is-enabled nix-daemon
# → enabled と表示されればOK
```

> ⚠️ ここで `Unit nix-daemon.service not found` が出る場合は、Determinate Nixのサービス登録漏れです。  
> [`setup_error_handling.md` E-NIX-02](./setup_error_handling.md#e-nix-02-unit-nix-daemonservice-not-found) を参照してください。

---

## STEP 4. プロジェクトの取得

### 4-1. リポジトリをクローン

```bash
cd ~
git clone <このリポジトリのURL>
cd <リポジトリ名>
```

> ⭐ **重要: 作業場所のルール**  
> プロジェクトは **必ず WSL のホームディレクトリ配下 (`~/`) に置いてください。**
> 
> - ✅ 良い例: `~/myproject` (= `/home/<your-name>/myproject`)
> - ❌ 悪い例: `/mnt/c/Github/myproject` (Windowsファイルシステム)
> 
> `/mnt/c/` 配下は I/O が10倍以上遅く、`pnpm install` などで権限エラーが頻発します。  
> Windows側からファイルを見たい場合は、エクスプローラーのアドレスバーに以下を入力すればアクセスできます。  
> `\\wsl$\Ubuntu\home\<your-name>\myproject`

---

## STEP 5. Devbox shell に入る

プロジェクトルートで以下を実行します。

```bash
devbox shell
```

初回は依存パッケージのダウンロード・ビルドが走るため、数分かかる場合があります。

成功すると以下のようなメッセージが表示され、Devbox環境のシェルに入ります。

```
🚀 Devbox shell に入りました
📖 よく使うコマンド, サーバー起動手順は docs/commands.md を参照
```

---

## ✅ セットアップ完了

ここまでで Devbox 環境のセットアップは完了です。  
次は [`2_setup.md`](./2_setup.md) を参照し、フロント/バックエンドサーバーの起動手順に進んでください。

---

## 🔧 トラブルシューティング

セットアップ中に発生したエラーは、ほぼすべて [`setup_error_handling.md`](./setup_error_handling.md) に対処法をまとめています。

代表的なもの:

| 症状 | 参照先 |
|---|---|
| `Failed to connect to bus: Host is down` | [E-NIX-01](./setup_error_handling.md#e-nix-01-failed-to-connect-to-bus-host-is-down) |
| `Unit nix-daemon.service not found` | [E-NIX-02](./setup_error_handling.md#e-nix-02-unit-nix-daemonservice-not-found) |
| `Connection refused` (nix-daemon) | [E-NIX-03](./setup_error_handling.md#e-nix-03-cannot-connect-to-socket-at-nixvarnixdaemon-socketsocket-connection-refused) |
| `Permission denied` (`/nix/var/nix/db/big-lock`) | [E-NIX-04](./setup_error_handling.md#e-nix-04-opening-lock-file-nixvarnixdbbig-lock-permission-denied) |
| `devbox: command not found` | [E-NIX-05](./setup_error_handling.md#e-nix-05-devbox-command-not-found) |
| `devbox shell` が固まる/失敗する | [E-NIX-06](./setup_error_handling.md#e-nix-06-devbox-shell-の初回ビルドで止まるエラーで落ちる) |
| `/mnt/c/` 配下で開発していて遅い | [E-WSL-03](./setup_error_handling.md#e-wsl-03-プロジェクトを-mntc-配下に置いていて遅い権限エラー) |

### VSCode から WSL に接続したい

VSCodeに [Remote - WSL](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-wsl) 拡張機能をインストールし、WSL側のターミナルで以下を実行します。

```bash
code .
```
