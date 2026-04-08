# XOR 復号した難読化文字列一覧

## 復号方式

```
hex_string → bytes → key = bytes[0] → result = bytes[1:] XOR key → UTF-8
```

実装は `dev.impl.sweepMethod.atomicRange()` 参照。

復号スクリプト (Python):

```python
def deob(hex_str):
    b = bytes.fromhex(hex_str)
    key = b[0]
    return bytes(x ^ key for x in b[1:]).decode('utf-8')
```

## クラス別復号結果

### `dev.impl.SpecRenderer` (Fabric MOD エントリ)

| 暗号化 (hex) | 復号後 |
|---|---|
| `105d7f7430797e79643063647164752a305d20` | `Mod init state: M0` |
| `88c5e7eca8e1e6e1fca8fbfce9fcedb2a8c5b9` | `Mod init state: M1` |
| `773a1813571e191e035704031603124d573a45` | `Mod init state: M2` |
| `J2716104416434510100a44441f100a13...` | `Mod init state: M3` |
| `Mod init state: M4 ` | (平文残存) |
| `ee8188888287808b` | `offline` |
| `f6b5999882939882dba28f8693` | `Content-Type` |
| `5f041a1911027f3a2d31` | `[EFN] ern` |
| `Vdfefa7eeede7efbee7ebee99bdbcee99e7e7ec...` | `0x1280a841Fbc1F883365d3C83122260E0b2995B74` |

### `dev.impl.LayoutService` (スタンドアロン Main)

| 暗号化 | 復号後 |
|---|---|
| `J6051570351040257574d030358574d54...` | `17c1db77-cc87-4783-88d1-7ceca88d88c5` |
| `6b10490e130e081e1f0204052e051d02...` | `{"executionEnvironment":"DoubleClick","userId":"17c1db77-cc87-4783-88d1-7ceca88d88c5"}` |
| `80e2e9ee` | `bin` |
| `046e657265732a617c61` | `javaw.exe` |
| `96bbfcf7e4` | `-jar` |

### `dev.impl.InternalWrapper` (第二段階ローダー)

| 暗号化 | 復号後 |
|---|---|
| `a898d0999a9098c9909c99eecacb99ee...` | `0x1280a841Fbc1F883365d3C83122260E0b2995B74` |
| `d7f9b4bbb6a4a4` | `.class` |
| `ef8b8a99c1828e858e81869b80c1a28e8681` | `dev.majanito.Main` |
| `9ef7f0f7eaf7fff2f7e4fbc9fbfbfaf6fffdf5` | `initializeWeedhack` |
| `82d0e7f1edf7f0e1e7a2f1f6e3f6e7b8a2d1b1` | `Resource state: S3` |
| `/files/jar/module` | (平文) |

### `dev.impl.RegistrySelector` (ETH コントラクト C2 解決)

#### 公開鍵・コントラクト関連

| 暗号化 | 復号後 |
|---|---|
| `1828607b7d2e7c2c297c7d` | `0xce6d41de` (function selector) |
| `2b78636a191e1d5c425f4379786a` | `SHA256withRSA` |
| `e9a4a0a0aba083a8a7ab8e8298818280aed09e...` | (RSA 公開鍵 base64, [IOCS.md](IOCS.md) 参照) |
| `e19ac38b928e8f939182c3dbc3d3cfd1c3cdc38c...` | `{"jsonrpc":"2.0","method":"eth_call","params":[{"to":"%s","data":"%s"},"latest"],"id":1}` |
| `e68796968a8f8587928f8988c98c958988` | `application/json` |
| `3311415640465f47110911` | `"result":"` |
| `105e7f30627563657c64` | `No result` |

#### Ethereum RPC エンドポイント (17 本)

| 復号後 URL |
|---|
| `https://eth.api.onfinality.io/public` |
| `https://ethereum-rpc.publicnode.com` |
| `https://ethereum.rpc.subquery.network/public` |
| `https://ethereum-json-rpc.stakely.io` |
| `https://ethereum-public.nodies.app` |
| `https://mainnet.gateway.tenderly.co` |
| `https://ethereum-mainnet.gateway.tatum.io` |
| `https://public-eth.nownodes.io` |
| `https://rpc.mevblocker.io/fast` |
| `https://rpc.mevblocker.io/noreverts` |
| `https://rpc.mevblocker.io/fullprivacy` |
| `https://eth-mainnet.nodereal.io/v1/1659dfb40aa24bbb8153a677b98064d7` |
| `https://eth-mainnet.public.blastapi.io` |
| `https://ethereum.public.blockpi.network/v1/rpc/public` |
| `https://eth-mainnet.rpcfast.com?api_key=xbhWBI1Wkguk8SNMu1bvvLurPGLXmgwYeC4S6g2H7WdwFigZSmPWVZRxrskEQwIf` |
| `https://eth.drpc.org` |
| `https://rpc.flashbots.net/fast` |
| `https://gateway.tenderly.co/public/mainnet` |
| `https://eth.merkle.io` |
| `https://api.zan.top/eth-mainnet` |
| `https://endpoints.omniatech.io/v1/eth/mainnet/public` |
| `https://1rpc.io/eth` |

### `dev.impl.UnusedStack` (DoH + 生 TLS)

| 暗号化 | 復号後 |
|---|---|
| `5f1e3c3c3a2f2b` | `Accept` |
| `3958494955505a584d505657165d574a14534a56...` | `application/dns-json` |
| `05276164716427` | `"data"` |
| `29415d5d595a` | `https` |
| `ace5c2dacdc0c5c88cdec9dfdcc3c2dfc9` | `Invalid response` |
| `113123212131` | ` 200 ` |
| `abe5c48bc9c4cfd2` | `No body` |
| `137b67676360293c3c707f7c6677757f7261763e...` | `https://cloudflare-dns.com/dns-query?name=%s&type=A` |
| `8ae2fefefaf9b0a5a5bba4bba4bba4bba5eee4f9...` | `https://1.1.1.1/dns-query?name=%s&type=A` |
| `573f232327246d787833392479303838303b3278...` | `https://dns.google/resolve?name=%s&type=A` |

## 復号スクリプト (再現用)

```python
import re, os
from collections import OrderedDict

def deob(hex_str):
    """先頭バイトをXORキーとして残りを復号"""
    try:
        b = bytes.fromhex(hex_str)
        if len(b) < 2: return None
        key = b[0]
        out = bytes(x ^ key for x in b[1:])
        return out.decode('utf-8')
    except:
        return None

def extract_class_strings(class_path):
    """.classファイルから6文字以上のhex連続を抽出して復号を試す"""
    data = open(class_path, 'rb').read()
    results = OrderedDict()
    for m in re.finditer(rb'[0-9a-f]{6,}', data):
        h = m.group().decode()
        if len(h) % 2 == 0:
            r = deob(h)
            if r and r.isprintable():
                results[h[:60]] = r
    return results

# 使い方:
#   for cls in glob.glob('dev/impl/*.class'):
#       for h, r in extract_class_strings(cls).items():
#           print(f'{h:62} -> {r}')
```
