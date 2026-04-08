# 感染した可能性がある場合の対処手順

> [!CAUTION]
> 偽 MOD を一度でも `Minecraft` 起動中の状態で `mods/` に置いた、もしくはダブルクリックで起動した可能性がある場合、**Minecraft アクセストークンが盗まれていると仮定して**ただちに対応してください。
> Defender が即時検疫した場合は実害ゼロの可能性が高いですが、念のため下記を実施するのが安全です。

## 緊急対応 (60分以内)

### 1. Microsoft アカウントのセッション全失効

最優先。盗まれたアクセストークンは Mojang/Microsoft 認証を経由せず Minecraft 内の操作に使えるため、まず**トークン側を無効化**します。

1. https://account.live.com/proofs/Manage にサインイン
2. 「セキュリティ」→「サインインアクティビティ」を確認、見覚えのないログインがないか確認
3. 「すべてのデバイスからサインアウト」を実行
   - これで現在発行されている全アクセストークンが無効化される
4. パスワードを変更（過去に他サービスで使ったものは絶対に避ける）
5. 二段階認証 (2FA) を有効化していなければ即有効化
6. https://account.microsoft.com/devices で見覚えのないデバイスを削除

### 2. Minecraft Launcher の再ログイン

1. Minecraft Launcher を完全終了
2. 再起動して再ログイン（古いトークンは前項で失効済み）

### 3. 走っているプロセスを確認

```powershell
Get-CimInstance Win32_Process | Where-Object { $_.Name -match '^(java|javaw)' } |
    Select-Object ProcessId, ParentProcessId, Name, CommandLine | Format-List
```

正常な Minecraft プロセスは:
- `Name: javaw.exe`
- `ParentProcessId` が `Minecraft.exe` または `Minecraft Launcher`

それ以外の `javaw.exe`（特に親が `cmd.exe`, `explorer.exe`, または親PIDが見えないもの）は即停止：

```powershell
Stop-Process -Id <PID> -Force
```

### 4. Microsoft Defender のフルスキャン

```powershell
Start-MpScan -ScanType FullScan
```

または GUI: `Windows セキュリティ` → `ウイルスと脅威の防止` → `スキャンのオプション` → `フル スキャン`

検出名 `Trojan:Win32/Egairtigado!rfn` が出たら検疫を実行。

## 24時間以内の対応

### ブラウザ系の全セッション失効

第二段階 (`dev.majanito.Main`) はブラウザのCookie・パスワードDB・ウォレット拡張を盗む可能性が高いため:

1. **全ブラウザを閉じる**
2. **保存パスワードを別管理に移行**:
   - Chrome/Edge/Brave: `chrome://settings/passwords` でエクスポート → **エクスポート後すぐ削除**
3. **Cookie 全削除**: 全ブラウザの全プロファイルでクリア
4. **重要サービスのパスワード変更**:
   - Google, Microsoft, Apple
   - GitHub, GitLab
   - Discord, Steam, Riot, Epic, Origin
   - SNS (X, Instagram, Facebook, TikTok)
   - メール各種
5. **重要サービスは「全デバイスからサインアウト」を実行**:
   - Discord: `User Settings` → `Devices` → `Log Out All Known Devices`
   - Steam: Settings → Family → Manage Devices
   - Google: https://myaccount.google.com/security
6. 全パスワードマネージャーのマスターパスワードを変更

### 暗号通貨ウォレットの保護

ウォレットを使用している場合は **致命的リスク**:

1. **クリーンな端末**（別PC、スマホ、または OS再インストール後）でシードフレーズから新しいウォレットを作成
2. 全資産を新ウォレットに移動
3. 古いウォレットは破棄
4. 取引所の API キー全失効・再発行

### Discord アカウントの保護

`RPCHelper` クラスがあることから Discord トークン窃取の可能性高:

1. https://discord.com/login で再ログイン
2. `User Settings` → `My Account` → `Change Password`
3. `Devices` → 全セッション失効
4. 2FA 有効化（必須）
5. 既参加のサーバで自分のアカウントが**勝手に DM 送信していないか**確認、心当たりない投稿は削除して周知

## 1週間以内の対応

### 永続化の確認

マルウェアが起動時自動実行を仕込んでいないか確認:

#### スタートアップフォルダ

```powershell
Get-ChildItem "$env:APPDATA\Microsoft\Windows\Start Menu\Programs\Startup\"
Get-ChildItem "$env:ProgramData\Microsoft\Windows\Start Menu\Programs\StartUp\"
```

不審な `.jar`, `.bat`, `.lnk` ファイルがあれば削除。

#### レジストリ Run キー

```powershell
Get-ItemProperty "HKCU:\SOFTWARE\Microsoft\Windows\CurrentVersion\Run"
Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Run"
Get-ItemProperty "HKCU:\SOFTWARE\Microsoft\Windows\CurrentVersion\RunOnce"
Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\RunOnce"
```

`java`, `javaw`, `Module.jar`, `whreceiverrrrrrrrr` を含むエントリは削除。

#### スケジュールタスク

```powershell
Get-ScheduledTask | Where-Object {
    $_.Actions.Execute -match 'java|jar' -and
    $_.TaskPath -notmatch 'Microsoft|OneDrive'
} | Format-List TaskName, TaskPath, Actions
```

#### `%TEMP%`, `%APPDATA%` の監査

```powershell
Get-ChildItem -Recurse "$env:TEMP","$env:APPDATA" -Filter *.jar -ErrorAction SilentlyContinue |
    Select-Object FullName, Length, LastWriteTime
```

身に覚えのない `.jar` は中身確認せず削除。

### ネットワーク監視

数日間、`whreceiverrrrrrrrr.ru` および `185.178.208.129` への通信が**ホストのファイアウォール／pihole／ルータログに出ないか**監視。

```powershell
# Windows ファイアウォールで明示的にブロック
New-NetFirewallRule -DisplayName "Block whreceiverrrrrrrrr C2" `
    -Direction Outbound -Action Block `
    -RemoteAddress 185.178.208.129
```

## 完全リセットを推奨するケース

以下に該当する場合は、Windows のクリーンインストール（完全初期化、データ移行は手動でファイル単位）を強く推奨します:

- 仮想通貨ウォレットでの保有額が高額
- 機密情報を扱う業務で使用している PC
- 親 PID 不明の `javaw.exe` が確認された
- Defender の検出履歴に `Trojan:Win32/Egairtigado` 以外の不審な検出もある
- 24時間以内に Microsoft アカウントへの不審なログイン試行があった

## 共有 / 通報

同じ偽 MOD が出回っているのを見かけたら:

1. **配布元の Discord サーバに通報** (運営に DM、`#mod-help` 等で警告投稿)
2. **MalwareBazaar に SHA-256 で照会・サンプル提出** (https://bazaar.abuse.ch/)
3. **VirusTotal に投稿** (https://virustotal.com/)
4. **Microsoft セキュリティに報告**: https://www.microsoft.com/wdsi/filesubmission
5. **Discord Trust & Safety に通報**: https://dis.gd/request

検出名と SHA-256 は [IOCS.md](IOCS.md) を共有してください。
