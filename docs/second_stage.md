# 第二段階 `Module.jar` の構成

第一段階 `MoneyDupeXLimit-1.2.jar` が実行時に Ethereum コントラクト経由で取得・メモリロードする本体ペイロード。

## ファイルメタデータ

| 項目 | 値 |
|---|---|
| 配布元 | `https://whreceiverrrrrrrrr[.]ru/files/jar/module` |
| サイズ | 7,016,081 bytes (約 6.7 MB) |
| `Content-Type` | `application/java-archive` |
| `Content-Disposition` | `attachment; filename=Module.jar` |
| `Last-Modified` | Sun, 05 Apr 2026 01:30:30 GMT |
| `Cache-Control` | `public, max-age=14400` |
| `ETag` | `"1775332851.5885766-7016081-3547009595"` |
| SHA-256 | `a29b15477ef19421cc12b332f78a25fa10d94294b753931733678c89bf4c145e` |
| Defender | `Trojan:Win32/Egairtigado!rfn` |

## エントリポイント

| 項目 | 値 |
|---|---|
| クラス | `dev.majanito.Main` |
| メソッド | `initializeWeedhack` |
| 呼び出し方 | `InternalWrapper` がリフレクション経由で別スレッド実行 |

メソッド名 `Weedhack` は典型的な「Minecraft cheat client」スラング（"weed" + "hack"）。攻撃者の趣味嗜好の名残。

## 構成パッケージ

総クラス数: **2,886**（ライブラリ含む）

### マルウェア本体クラス (51 個)

#### `dev.majanito.*` (本体ロジック)

| クラス | 推定役割 |
|---|---|
| `Main` | エントリ。`initializeWeedhack(String)` で第一段階から JSON コンテキストを受け取る |
| `Helper` | 共通ユーティリティ |
| `IMCL` | (恐らく) Minecraft Launcher 連携・プロファイル読み取り |
| `RPCHelper` | Discord RPC 連携 (Discord トークン窃取の可能性) |
| `TelemetryHelper` | C2 への送信処理 |
| `CloudflareDNS` | 第一段階の `UnusedStack` と同じ DoH + 生TLS 実装の再利用 |
| `CloudflareDNS$1`, `$2` | 全証明書受理 TrustManager |
| `CloudflareDNS$CacheEntry` | DNS キャッシュ |

#### `dev.jnic.LMWJYW.*` (ネイティブローダー、難読化済み)

| クラス | 役割 |
|---|---|
| `JNICLoader` | JNI ネイティブライブラリの動的ロード |
| `D`, `H`, `I`, `L`, `M`, `Q`, `U`, `d`, `f`, `g`, `k`, `y` | ヘルパー（1文字名で完全難読化） |

`LMWJYW` は意味不明なプレフィックスで、難読化のためのランダム文字列。

### 同梱ライブラリ (2,835 クラス)

| 名前空間 | クラス数 | 用途 |
|---|---|---|
| `com.sun.jna.platform.win32` | 852 | **Windows API 直接呼び出し**（レジストリ、DPAPI、COM、サービス、プロセスなど） |
| `com.sun.jna.platform.win32.COM` | 99 | **COM 操作**（Outlook、IE/Edge などの統合） |
| `com.sun.jna` | 98 | JNA 本体 |
| `com.sun.jna.platform.unix` | 92 | Unix/Linux ネイティブ |
| `com.sun.jna.platform.mac` | 61 | **macOS** Keychain アクセス想定 |
| `com.sun.jna.platform` | 49 | プラットフォーム抽象 |
| `com.sun.jna.platform.unix.aix` | 38 | AIX 対応 |
| `kotlin.*` | 約 480 | Kotlin Standard Library |
| `okhttp3.*` | 105 | HTTP クライアント |
| `okhttp3.internal.http2` | 49 | HTTP/2 対応 |
| `okio.*` | 84 | I/O 抽象 |
| `org.json.*` | 約 25 | JSON 処理 |

## 同梱ネイティブバイナリ (`com/sun/jna/<arch>/`)

JNA がプラットフォーム検出時にロードする `libjnidispatch` がアーキ別に含まれる:

```
linux-aarch64       linux-mips64el      sunos-x86-64
linux-armel         linux-ppc64le       win32-x86
linux-armhf         linux-riscv64       win32-x86-64
linux-loong64       linux-s390x         w32ce-arm
linux-mips64        linux-x86-64        ...
openbsd-x86-64      darwin (mac)
```

→ **完全クロスプラットフォーム対応の窃取機構**であることを示唆。

## 推定される動作（クラス名と依存ライブラリから）

> [!NOTE]
> 以下は実装の逆解析ではなく、**クラス名・依存ライブラリ・ファミリー(Egairtigado)の既知挙動から推定**したもの。
> 確実な挙動を知るにはサンドボックス上での動的解析が必要。

### Windows での想定窃取対象

JNA + Win32 API による直接アクセスが想定されるもの:

- **Chromium 系ブラウザ** (Chrome, Edge, Brave, Opera, Vivaldi, Arc)
  - `Local State` の DPAPI 暗号化キー → `Login Data` SQLite から保存パスワード
  - `Cookies` SQLite → セッショントークン
  - `Web Data` → クレジットカード情報
- **Firefox 系**
  - `key4.db` + `logins.json` → 保存パスワード
  - `cookies.sqlite`
- **Discord**
  - `%APPDATA%\discord\Local Storage\leveldb\` → トークン
  - 派生クライアント (Lightcord, BetterDiscord 等)
- **Telegram Desktop**
  - `tdata\` セッション
- **Steam**
  - `ssfn*` ファイル (セッションファイル)
- **暗号通貨ウォレット拡張**
  - MetaMask, Phantom, Coinbase, Trust Wallet, Brave Wallet ほか
  - 拡張機能のローカルストレージ
- **デスクトップウォレット**
  - Exodus, Atomic, Electrum, Ledger Live
- **Minecraft launcher 系**
  - `IMCL` クラスから推測：Vanilla Launcher, MultiMC, Prism, ATLauncher など
- **DPAPI 経由の Windows 資格情報マネージャー**
  - Wi-Fi パスワード、リモートデスクトップ資格情報

### macOS での想定

- Keychain アクセス
- ブラウザ（Chrome, Safari, Firefox, Brave）
- Discord, Telegram のセッション

### Linux での想定

- ブラウザの平文 sqlite3 (Chromium-Linux はキー保護が弱い)
- `~/.config/discord/`

## C2 送信経路

`TelemetryHelper` クラスが `okhttp3` を使い、`CloudflareDNS` クラス経由で名前解決した上で送信すると推定される。送信先は第一段階と同じ `whreceiverrrrrrrrr[.]ru` か、サブパスを変えている可能性が高い。

## 関連リソース

- `assets/ref_upper.dat` (72 KB) ── 第一段階に同梱されている暗号化リソース。`dev.majanito.Main` が `getResourceAsStream("ref_upper.dat")` 経由で読み込むと推定（クラス名で参照されている形跡なし、文字列復号で発見される可能性あり）。中身は追加のキー or 設定 or 追加コードバイトコードと予想。

## 解析メモ

このリポジトリでは**第二段階 jar の動的解析は実施していません**。理由:

1. 攻撃者の C2 (生きている) と通信される可能性
2. ホストOSに対する実害リスク
3. 解析専用 VM・隔離ネットワークが必要

サンドボックス環境がある研究者向けに、SHA-256 を [IOCS.md](IOCS.md) に記録しています。VirusTotal や MalwareBazaar での既存解析結果と照合してください。
