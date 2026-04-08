# IoC (Indicators of Compromise)

> [!IMPORTANT]
> 全ての URL・コントラクトアドレスは**実在する活動中のマルウェア基盤**です。
> 検証やブロック登録目的でのみ使用し、絶対にアクセスしないでください。

## ファイルハッシュ

| 検体 | アルゴリズム | ハッシュ |
|---|---|---|
| `Module.jar` (第二段階) | SHA-256 | `a29b15477ef19421cc12b332f78a25fa10d94294b753931733678c89bf4c145e` |
| `Module.jar` (第二段階) | SHA-1 | `eb84957b8eb866c212fb0f5bb6ee1724aa374ac0` |
| `Module.jar` (第二段階) | MD5 | `787dda769291e21e1b5367218cfd2555` |
| `MoneyDupeXLimit-1.2.jar` | サイズ | 97,333 bytes |
| `Comet-V1.21.jar` | - | Defenderにより取得前検疫 |

## C2 インフラ

### 第一段階 C2 (動的 — ETH コントラクト経由で配信)

| 種類 | 値 |
|---|---|
| ホスト名 | `whreceiverrrrrrrrr[.]ru` |
| 解決IP | `185.178.208.129` |
| TLS フロント | `Server: ddos-guard` |
| 国 | RU (ロシア) |

### 既知エンドポイント

| パス | メソッド | 用途 |
|---|---|---|
| `/api/delivery/handler` | POST | 窃取資格情報の受信 |
| `/files/jar/module` | GET | 第二段階 `Module.jar` の配布 (`Content-Length: 7016081`, `Content-Disposition: attachment; filename=Module.jar`) |

### Ethereum スマートコントラクト C2 レジストリ

| 項目 | 値 |
|---|---|
| ネットワーク | Ethereum Mainnet |
| コントラクトアドレス | `0x1280a841Fbc1F883365d3C83122260E0b2995B74` |
| 関数セレクタ | `0xce6d41de` (read-only) |
| 戻り値形式 | `<URL>\|<base64 RSA-SHA256 署名>` |
| 観測時の戻り値 | `https://whreceiverrrrrrrrr.ru\|<署名>` (2026/04/08) |

### 利用される公開 ETH RPC エンドポイント (フォールバック 17 本)

```
https://eth.api.onfinality.io/public
https://ethereum-rpc.publicnode.com
https://ethereum.rpc.subquery.network/public
https://ethereum-json-rpc.stakely.io
https://ethereum-public.nodies.app
https://mainnet.gateway.tenderly.co
https://ethereum-mainnet.gateway.tatum.io
https://public-eth.nownodes.io
https://rpc.mevblocker.io/fast
https://rpc.mevblocker.io/noreverts
https://rpc.mevblocker.io/fullprivacy
https://eth-mainnet.nodereal.io/v1/1659dfb40aa24bbb8153a677b98064d7
https://eth-mainnet.public.blastapi.io
https://ethereum.public.blockpi.network/v1/rpc/public
https://eth-mainnet.rpcfast.com?api_key=...
https://eth.drpc.org
https://rpc.flashbots.net/fast
https://gateway.tenderly.co/public/mainnet
https://eth.merkle.io
https://api.zan.top/eth-mainnet
https://endpoints.omniatech.io/v1/eth/mainnet/public
https://1rpc.io/eth
```

## 暗号要素

### URL 署名検証用 RSA 公開鍵 (PKCS#1, 2048bit)

```
MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAtmNzDf4737/iYWvscWg6
vQg9dHa/yUchfQY9r5htNTLZ3ZDAbqrzN93I0ctZHa27oRnkpB7XpowI4NH8eIRm
aMThggpTYRXzHzLvUjhyrFFPkIOo/HI1gZF5IV7/XmvYWqgEsSpxl0iesOUlaWO5
A8QlTu0QLsZAzZtzZyLj/v1XbPT02rTvZkuRhE6nzpUR4GN3Jp4Bn8zQAWdFDe17
PWZxOi19uUTMPzgFj9n3h7DprwBmE3fR7IMsbiFacAoSHfqkTpEwY7A8ArK1DQ1y
JXPog/PQ4aTU9gU38WC20wtct796ImZiuRYdNWcSzHda5ZbvZdvpw6RHh0zQqGVh
RQIDAQAB
```

署名アルゴリズム: `SHA256withRSA`

## DNS over HTTPS (Cloudflare DNS) エンドポイント

```
https://cloudflare-dns.com/dns-query?name=%s&type=A
https://1.1.1.1/dns-query?name=%s&type=A
https://dns.google/resolve?name=%s&type=A
```

## マルウェア内部識別子

| 項目 | 値 |
|---|---|
| キャンペーン ID | `17c1db77-cc87-4783-88d1-7ceca88d88c5` |
| 第一段階エントリ (mod) | `dev.impl.SpecRenderer.onInitialize` |
| 第一段階エントリ (standalone) | `dev.impl.LayoutService.main` |
| 第二段階エントリクラス | `dev.majanito.Main` |
| 第二段階エントリメソッド | `initializeWeedhack` |
| Fabric mod id | `moneydupex` |
| 偽装作者 | `Ranqz` |
| 第二段階パッケージ | `dev.majanito` (Helper, IMCL, RPCHelper, TelemetryHelper, CloudflareDNS, Main) |
| ネイティブローダー | `dev.jnic.LMWJYW.JNICLoader` ほか 12 クラス |

## アンチウイルス検出名

| ベンダー | 検出名 |
|---|---|
| Microsoft Defender | `Trojan:Win32/Egairtigado!rfn` |

## 被害送信ペイロード形式

3 つのスキーマが観測されている：

```json
// flat
{"username":"…","uuid":"…","accessToken":"…","minecraftInfo":"<campaign>"}

// fabric
{"executionEnvironment":"Fabric",
 "minecraftInfo":{"username":"…","uuid":"…","accessToken":"…"},
 "userId":"<campaign>"}

// doubleclick (standalone 起動時)
{"executionEnvironment":"DoubleClick",
 "minecraftInfo":{"username":"…","uuid":"…","accessToken":"…"},
 "userId":"<campaign>"}
```
