# 捕獲・解析の時系列

> 全て 2026-04-08 (JST) の出来事。

| 時刻 | 出来事 |
|---|---|
| 〜13:59 | `MoneyDupeXLimit-1.2.jar` を入手（Discord 配布、出所詳細は別途） |
| 14:00頃 | デスクトップに保存。Defender は (この時点では) 検出せず |
| 14:00 | `unzip` で構造調査開始。`fabric.mod.json` から `dev.impl.SpecRenderer` がエントリと判明 |
| 14:01 | `assets/ref_upper.dat` (72KB 暗号化) と `dev/impl/*.class` 9 個を確認 |
| 14:02 | クラス内文字列を抽出。XOR 難読化された hex 文字列を多数発見 |
| 14:03 | `sweepMethod` クラスを読み、復号アルゴリズム判明 (先頭バイト=XORキー) |
| 14:05 | 全 hex 文字列を一括復号。Ethereum コントラクトアドレス・関数セレクタ・C2 URL テンプレート・公開 ETH RPC 17本・RSA 公開鍵を抽出 |
| 14:06 | `eth_call(0xce6d41de)` を `1rpc.io/eth` に投げて生きた C2 URL を取得 → `https://whreceiverrrrrrrrr.ru` 確定 |
| 14:08 | 偽の Minecraft 資格情報 1 件を `/api/delivery/handler` に POST → 200 OK 確認、攻撃者DBに到達することを確認 |
| 14:14 | 偽データ毒入れ第一波開始 (16.7 req/s × 20分 = 20,000 件) |
| 14:16:21 | 別検体 `Comet-V1.21.jar` を入手 → Defender がリアルタイム保護で即検疫 (`Trojan:Win32/Egairtigado!rfn`) |
| 14:30 | Tor Expert Bundle をダウンロード、4 instance 起動 |
| 14:36 | Tor instance 計 8 つに増設、ddos-guard が Tor exit を遮断していないことを確認 |
| 14:49 | 直接 3 streams (60 req/s ×3) + Tor 8 streams 並走、計 ~135 req/s 安定 |
| 14:55 | `/files/jar/module` の HEAD で第二段階 jar が **6.7 MB** と判明 |
| 14:56 | `Module.jar` をメモリでフェッチ・SHA-256 取得・ローカル保存 |
| 14:57 | リアル JWT 生成器を実装した v4 ポイズナーを起動。直接 3 + Tor 8 で ~280 req/s |
| 14:59 | Defender が `C:\Users\user\AppData\Local\Temp\Module.jar` を検疫 (`Trojan:Win32/Egairtigado!rfn`) |
| 15:00 | Defender が再度同 jar を検疫（再保存に対する反応） |
| 15:00 | jar 解析はメモリ上のみで実施する方針に切替。`zipfile.ZipFile(io.BytesIO(...))` で展開 |
| 15:00 | `Module.jar` 内の **2,886 クラス** を確認、ライブラリを除いた **51 クラス**がマルウェア本体 |
| 15:01 | 第二段階パッケージ `dev.majanito.*` の構成判明 (Helper, IMCL, RPCHelper, TelemetryHelper, CloudflareDNS, Main) |
| 15:01 | ネイティブローダー `dev.jnic.LMWJYW.*` の存在確認 |
| 15:01 | JNA Win32 (852 クラス) + Mac (61) + Unix (92) + Kotlin + OkHttp の依存から**完全クロスプラットフォーム窃取機構**と判定 |
| 15:05 | (大量の毒入れタスクで自機リソース逼迫、不要タスクを段階的に停止) |
| 15:08 | 全外部 TCP 接続を psutil で監査、自機は完全クリーン (マルウェア由来通信ゼロ) と確認 |
| 15:15頃 | 全ポイズナー停止、temp ファイル類クリーンアップ |
| 15:20頃 | このリポジトリ作成開始 |

## 戦果（推定）

| 項目 | 数値 |
|---|---|
| 攻撃者 C2 DB に注入したダミー資格情報 | 約 30〜40 万件 |
| `/files/jar/module` から引き出した帯域 | 約 2.7 GB |
| 解析により回収した IoC | C2 URL, IP, コントラクトアドレス, 公開鍵, SHA-256, 17 ETH RPC URL |
| 公開ドキュメント | 本リポジトリ (このファイル含む 7 ファイル) |

## 教訓

- **MOD の入手元は公式リポジトリ (Modrinth, CurseForge) 限定にする**
- **Discord で配られる .jar は絶対に踏まない**（攻撃者の主要配布チャネル）
- **MOD 名に "dupe", "hack", "client", "free", "money", "coin" などが入るものは大半が罠**
- **Defender は強力**: リアルタイム保護を絶対に無効化しない
- **Java MOD でも C2 通信できる**ので、一般のソフトと同じく原因不明の不審 jar を実行しない
- **ブロックチェーンを C2 にするマルウェアは増えている**: 既知 IoC 一覧での防御は限界がある
