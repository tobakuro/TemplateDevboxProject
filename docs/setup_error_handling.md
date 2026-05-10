<!-- WSL / Devbox / Nix 関連のセットアップで起きがちなエラーと解決策をまとめた -->

# 🛠️ セットアップ エラーハンドリング集

`docs/1_setup_devbox.md` の手順を進める中で、WSL / Devbox / Nix 関連で **実際に踏んだエラー** とその解決策をまとめています。  
詰まったら **エラーメッセージで Ctrl+F** して該当箇所を探してください。

---

## 🗺️ 目次

- [WSL関連](#wsl関連)
  - [E-WSL-01. `wsl --install` が失敗する](#e-wsl-01-wsl---install-が失敗する)
  - [E-WSL-02. `wsl --shutdown` してもsystemdが起動しない](#e-wsl-02-wsl---shutdown-してもsystemdが起動しない)
  - [E-WSL-03. プロジェクトを `/mnt/c/` 配下に置いていて遅い・権限エラー](#e-wsl-03-プロジェクトを-mntc-配下に置いていて遅い権限エラー)
- [Devbox / Nix関連](#devbox--nix関連)
  - [E-NIX-01. `Failed to connect to bus: Host is down`](#e-nix-01-failed-to-connect-to-bus-host-is-down)
  - [E-NIX-02. `Unit nix-daemon.service not found`](#e-nix-02-unit-nix-daemonservice-not-found)
  - [E-NIX-03. `cannot connect to socket at '/nix/var/nix/daemon-socket/socket': Connection refused`](#e-nix-03-cannot-connect-to-socket-at-nixvarnixdaemon-socketsocket-connection-refused)
  - [E-NIX-04. `opening lock file "/nix/var/nix/db/big-lock": Permission denied`](#e-nix-04-opening-lock-file-nixvarnixdbbig-lock-permission-denied)
  - [E-NIX-05. `devbox: command not found`](#e-nix-05-devbox-command-not-found)
  - [E-NIX-06. `devbox shell` の初回ビルドで止まる/エラーで落ちる](#e-nix-06-devbox-shell-の初回ビルドで止まるエラーで落ちる)
  - [E-NIX-07. `Command 'nodenv' not found` 警告](#e-nix-07-command-nodenv-not-found-警告)
- [Git / GitHub関連](#git--github関連)
  - [E-GIT-01. HTTPS clone/push でパスワード認証エラー](#e-git-01-https-clonepush-でパスワード認証エラー)
  - [E-GIT-02. 改行コード警告 (LF/CRLF)](#e-git-02-改行コード警告-lfcrlf)
- [VSCode関連](#vscode関連)
  - [E-VSC-01. `code .` が動かない](#e-vsc-01-code--が動かない)

---

## WSL関連

### E-WSL-01. `wsl --install` が失敗する

**症状**

```
WslRegisterDistribution failed with error: 0x80370102
```

または `wsl --install` 自体が認識されない。

**原因**

- BIOS/UEFIで仮想化機能(Intel VT-x / AMD-V)が無効
- Windowsのバージョンが古い (Windows 10 ビルド19041未満など)
- 「Windowsの機能の有効化または無効化」で必要機能がオフ

**解決策**

1. Windows Updateを最新まで適用
2. BIOS/UEFIに入り、仮想化を有効化(項目名はメーカーごとに違う:`Intel Virtualization Technology` / `SVM Mode`など)
3. PowerShellを管理者で開き、機能を明示的に有効化:

```powershell
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```

4. 再起動 → `wsl --install` 再実行

---

### E-WSL-02. `wsl --shutdown` してもsystemdが起動しない

**症状**

`/etc/wsl.conf` に `systemd=true` を書いて `wsl --shutdown` したのに、再起動後に `ps -p 1 -o comm=` の結果が `init` のまま。

**原因**

**Docker Desktop が WSL を掴んでいる**ため、`wsl --shutdown` が中途半端に終わっている。

**解決策**

1. **Docker Desktop を完全終了**
   - タスクトレイのDockerアイコンを右クリック → `Quit Docker Desktop`
2. PowerShellで:

```powershell
wsl --shutdown
```

3. 数秒待ってから WSL を再度起動
4. 確認:

```bash
ps -p 1 -o comm=
# → systemd と表示されればOK
```

それでもダメなら、**Windowsを再起動**するのが最も確実。

---

### E-WSL-03. プロジェクトを `/mnt/c/` 配下に置いていて遅い・権限エラー

**症状**

- `pnpm install` や `npm install` が非常に遅い (5分以上かかる)
- `ERR_PNPM_EACCES` や `EACCES: permission denied` が出る
- `devbox shell` の動作が異様に遅い

**原因**

`/mnt/c/...` (例: `/mnt/c/Users/<name>/projects/...`) はWindowsファイルシステムを経由しており、I/Oが10倍以上遅くなる。さらに権限管理の都合で書き込みエラーが起きやすい。

**解決策**

**プロジェクトは必ず WSL のホームディレクトリ配下 (`~/`) に置く**。

```bash
# ❌ 悪い例
cd /mnt/c/Github/myproject

# ✅ 良い例
cd ~/myproject
```

すでに `/mnt/c/` で作業を始めてしまった場合は、丸ごとコピーして移動:

```bash
cp -r /mnt/c/Github/myproject ~/
cd ~/myproject
rm -rf node_modules .devbox  # 古い成果物は消す
```

Windows側からこのファイルを見たい場合は、エクスプローラーのアドレスバーに以下を入力:

```
\\wsl$\Ubuntu\home\<your-name>\myproject
```

---

## Devbox / Nix関連

### E-NIX-01. `Failed to connect to bus: Host is down`

**症状**

```
$ sudo systemctl start nix-daemon
System has not been booted with systemd as init system (PID 1). Can't operate.
Failed to connect to bus: Host is down
```

**原因**

WSLでsystemdが有効化されていない。

**解決策**

`/etc/wsl.conf` を編集してsystemdを有効化する:

```bash
sudo nano /etc/wsl.conf
```

以下を記載 (既に `[boot]` セクションがあれば `systemd=true` のみ追記):

```ini
[boot]
systemd=true
```

PowerShellでWSLを完全終了:

```powershell
wsl --shutdown
```

> Docker Desktopを起動している場合は先に終了してから実行 (詳細は [E-WSL-02](#e-wsl-02-wsl---shutdown-してもsystemdが起動しない))

WSLを再度開いて確認:

```bash
ps -p 1 -o comm=
# → systemd と表示されればOK
```

---

### E-NIX-02. `Unit nix-daemon.service not found`

**症状**

```
$ sudo systemctl start nix-daemon
Failed to start nix-daemon.service: Unit nix-daemon.service not found.
```

systemd は有効だが、サービスが登録されていない。

**原因**

**Determinate Nix** (Devboxが使うNixインストーラ) は `/nix/store/...` 以下にサービスファイルを置くが、**WSL環境特有のタイミング問題**で `/etc/systemd/system/` へのシンボリックリンク作成がスキップされることがある。

**解決策**

1. サービスファイルの実体を探す:

```bash
find /nix -name "nix-daemon.service" 2>/dev/null
```

出力例:
```
/nix/store/bb08df99wjxq3jmydyjcl2046w7rhnn2-determinate-nix-3.16.3/lib/systemd/system/nix-daemon.service
```

2. 見つかったパスをコピーしてシンボリックリンクを作成:

```bash
# ↓ ここのパスは上のfindコマンドの出力に置き換える
sudo ln -s /nix/store/<実際のパス>/lib/systemd/system/nix-daemon.service /etc/systemd/system/nix-daemon.service
```

3. systemdに認識させる:

```bash
sudo systemctl daemon-reload
sudo systemctl start nix-daemon
sudo systemctl enable nix-daemon  # 自動起動も有効化
```

4. 確認:

```bash
sudo systemctl status nix-daemon
# active (running) と出ればOK
```

これで `devbox shell` が通るようになります。

---

### E-NIX-03. `cannot connect to socket at '/nix/var/nix/daemon-socket/socket': Connection refused`

**症状**

```
$ devbox shell
Error: ... cannot connect to socket at '/nix/var/nix/daemon-socket/socket': Connection refused
Run with DEVBOX_DEBUG=1 for a detailed error message ...
```

**原因**

nix-daemonが起動していない。WSLの再起動後に自動起動が設定されていないと毎回起きる。

**解決策**

```bash
sudo systemctl start nix-daemon
sudo systemctl enable nix-daemon  # 次回以降の自動起動を設定
```

確認:

```bash
systemctl is-enabled nix-daemon
# enabled と表示されればOK
```

これでもダメなら [E-NIX-02](#e-nix-02-unit-nix-daemonservice-not-found) の手順 (シンボリックリンク作成) を試してください。

---

### E-NIX-04. `opening lock file "/nix/var/nix/db/big-lock": Permission denied`

**症状**

```
opening lock file "/nix/var/nix/db/big-lock": Permission denied: exit code 1
```

**原因**

nix-daemonが起動しておらず、ユーザー権限で直接 `/nix/var/nix/db/` を触ろうとして失敗している。

**解決策**

[E-NIX-03](#e-nix-03-cannot-connect-to-socket-at-nixvarnixdaemon-socketsocket-connection-refused) と同じ。`nix-daemon` を起動すれば解消する。

```bash
sudo systemctl start nix-daemon
. /nix/var/nix/profiles/default/etc/profile.d/nix-daemon.sh
devbox shell
```

それでもダメなら、所有権を確認:

```bash
ls -la /nix/var/nix/db/
```

通常は `root:nixbld` 所有になっているはず。そうでない場合はNixの再インストールを検討。

---

### E-NIX-05. `devbox: command not found`

**症状**

Devboxをインストールしたのに、ターミナルを開き直すと:

```
$ devbox version
devbox: command not found
```

**原因**

PATHが通っていない。Devboxのバイナリは `~/.local/bin/devbox` に入る。

**解決策**

`~/.bashrc` (zshの場合は `~/.zshrc`) に以下を追記:

```bash
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

確認:

```bash
which devbox
# → /home/<user>/.local/bin/devbox
devbox version
```

---

### E-NIX-06. `devbox shell` の初回ビルドで止まる/エラーで落ちる

**症状**

`devbox shell` を実行すると:

- 進捗ログが10分以上止まる
- HTTP/2 エラーが大量に出る
- 大文字の `Error:` で落ちる

例:
```
warning: error: unable to download ...: HTTP error 200 (curl error: Stream error in the HTTP/2 framing layer); retrying from offset ...
```

**原因**

- 初回は依存パッケージのダウンロードが大量にあり **15〜30分かかる** ことがある
- WSL2のネットワーク特性でリトライが発生しやすい

**解決策**

1. **まずは待つ** (15分以上は普通)
2. 完全に止まっている場合は `Ctrl+C` で中断 → 再実行 (キャッシュが効くので続きから再開)
3. それでも落ちる場合は:

```bash
# Nixキャッシュをクリーンアップ
nix-collect-garbage -d

# 再度実行
devbox shell
```

4. ネットワークが不安定な場合、有線接続やテザリングなど環境を変えて再試行

---

### E-NIX-07. `Command 'nodenv' not found` 警告

**症状**

`devbox shell` 起動時に:

```
Command 'nodenv' not found, did you mean:
  command 'nodeenv' from deb nodeenv (0.13.4-1.1)
```

**原因**

過去にnodenvを使っていた残骸が `~/.bashrc` に残っている。

**影響**

**機能には影響なし**。完全に無視してOK。

**気になる場合の解決策**

`~/.bashrc` を開いて `nodenv` の行を削除:

```bash
nano ~/.bashrc
# nodenv 関連の行を削除して保存
source ~/.bashrc
```

---

## Git / GitHub関連

### E-GIT-01. HTTPS clone/push でパスワード認証エラー

**症状**

```
remote: Support for password authentication was removed on August 13, 2021.
fatal: Authentication failed for 'https://github.com/...'
```

**原因**

GitHubはHTTPSでのパスワード認証を廃止している。トークン認証またはSSHを使う必要がある。

**解決策**

3つの選択肢から選ぶ:

#### 方法A: GitHub CLI (推奨・最も楽)

Devbox shell 内で:

```bash
# devbox.json の packages に "gh" を追加してから
devbox shell

gh auth login
```

ブラウザが開いてGitHubにログイン → 完了。以降 `git push` で認証不要。

#### 方法B: Personal Access Token (PAT)

1. GitHub: Settings → Developer settings → Personal access tokens → Generate new token
2. スコープ: `repo` にチェック
3. 生成されたトークンをパスワードとして使う:

```bash
git clone https://<ユーザー名>:<トークン>@github.com/<org>/<repo>.git
```

#### 方法C: SSH (永続的に楽)

```bash
# SSH鍵を生成 (なければ)
ssh-keygen -t ed25519 -C "your_email@example.com"
# Enter連打でOK

# 公開鍵を表示してコピー
cat ~/.ssh/id_ed25519.pub
```

GitHubの Settings → SSH and GPG keys → New SSH key に貼り付け。  
以降は SSH URL でcloneする:

```bash
git clone git@github.com:<org>/<repo>.git
```

---

### E-GIT-02. 改行コード警告 (LF/CRLF)

**症状**

```
warning: in the working copy of 'xxx', LF will be replaced by CRLF the next time Git touches it
```

または、Linux/Mac側で動くスクリプトがWindowsで改行コードが変わって動かなくなる。

**原因**

WindowsはCRLF、Linux/MacはLFを使う。WSLとWindowsを跨ぐ際に改行コードが混ざりやすい。

**解決策**

WSL側で:

```bash
git config --global core.autocrlf input
```

これで「コミット時はLFに統一、チェックアウト時は変換しない」設定になる。  
チームで揃える場合はリポジトリ直下に `.gitattributes` を置く:

```
* text=auto eol=lf
```

---

## VSCode関連

### E-VSC-01. `code .` が動かない

**症状**

WSL内のターミナルで `code .` を実行しても起動しない、または:

```
code: command not found
```

**原因**

VSCodeに「WSL拡張機能」が入っていない、またはWindows側のVSCodeがインストールされていない。

**解決策**

1. Windows側にVSCodeをインストール (https://code.visualstudio.com)
2. VSCodeを開いて拡張機能 (Ctrl+Shift+X) で `WSL` を検索 → Microsoft製の `WSL` 拡張をインストール
3. WSLターミナルで:

```bash
code .
```

これで現在のディレクトリがWSL接続モードでVSCodeに開く。

---

## 💡 困ったときの汎用デバッグ手順

どの症状にも当てはまらない場合は、まず以下を確認:

```bash
# 1. WSLのバージョン確認
wsl --version           # PowerShellで実行

# 2. systemd の状態
ps -p 1 -o comm=

# 3. nix-daemon の状態
systemctl status nix-daemon

# 4. Devboxのデバッグ実行
DEVBOX_DEBUG=1 devbox shell

# 5. Nix関連サービス一覧
systemctl list-units | grep nix

# 6. ディスク容量 (Nixは大量に食う)
df -h
```

これらの結果を貼って Issue / Slack / チームに相談すると、解決が早まります。

---

## 📚 参考リンク

- [Devbox公式ドキュメント](https://www.jetify.com/docs/devbox/)
- [WSL公式ドキュメント (Microsoft)](https://learn.microsoft.com/ja-jp/windows/wsl/)
- [Nix公式マニュアル](https://nixos.org/manual/nix/stable/)
- [GitHub Personal Access Tokens](https://github.com/settings/tokens)
