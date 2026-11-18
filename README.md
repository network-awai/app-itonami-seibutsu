# app-itonami-seibutsu — Seibutsu

生物 — 生物分類（taxon）と形質（traits）と観察記録を扱う、公開データ型の生物多様性カタログです。画像からの種の同定（推論）は含みません。

Tauri **3.0.0-alpha.6** の独立 app（`tauri-runtime-wry`、OS の WebView）と、同じファイルの static Web ビルドです。記録はすべてこの端末の中（localStorage、key `itonami.seibutsu.v1`）に保存し、外部へは通信しません。外部操作の宣言もありません（`dist/descriptor.json` の `operations: []`）。保存に失敗したときは画面に表示します。

## できること

| 操作 | 内容 |
| --- | --- |
| 分類を登録 (registerTaxon) | 親の分類（parentTaxonId）は先に登録しておく必要があります。 |
| 分類を見る (getTaxon) | — |
| 分類の一覧 (listTaxa) | — |
| 形質を記録 (deriveTraits) | — |
| 形質の一覧 (listTraits) | — |
| 観察を記録 (ingestObservation) | — |
| 観察の一覧 (listObservations) | — |
| 記録の数 (coverage) | — |
| プリセットの分類を読み込む | kami-vegetation の 7 科（と親の 4 門）を、分類と形質としてこの端末に登録します。登録済みのものはそのままにします。 |

記録（collection）: `cloud.itonami.apps.seibutsu.observation`、`cloud.itonami.apps.seibutsu.taxon`、`cloud.itonami.apps.seibutsu.traits`

## 補足

- 「プリセットの分類を読み込む」は、kami-vegetation の 7 科と、その親の 4 門（rank は `phylum`）を domain の検査を通して登録します。何度実行しても重複しません。旧 `seed.ts`（外部 PDS へ token つきで書く script）の置き換えです。
- プリセットの `heightRange` は描画用の相対値（cm ではない）なので、`matureHeightCm` には入れていません。domain にない habit（`succulent`、`mat`）は `other` にしています。
- 画像からの種の同定（推論）は含みません。
- `appview/.../cljs/`（旧 UI）は残しています。この app のビルド・テストでは使いません。

## 構成

| パス | 役割 |
| --- | --- |
| `src/domain/` | domain logic（TypeScript）。ビルド時に Node の type stripping で JS にします（bundler・依存なし） |
| `src/local-store.js` | この端末内の record store。domain が使う `read` / `write` を localStorage で実装 |
| `src/app-def.js` | 画面の操作と domain 関数・collection の対応 |
| `src/ui/` | 依存なしの UI。DOM はテキストノードだけで組み、HTML 文字列を使いません（CSP `style-src 'self'`） |
| `vendor/` | kotoba-lang/kotoba の `kotoba-client-runtime.js` / consent CSS と `PINS.json`。`scripts/build.mjs` が digest のずれで失敗します |
| `src-tauri/` | Tauri v3 host。read-only command は `awai_host_info` の 1 つだけ。filesystem・network・shell・installer 権限なし |
| `appview/` | 旧 appview（Svelte / cljs UI）。この app のビルド・テストでは使いません |

## コマンド

```sh
npm ci
npm test              # build してから domain と local-only のテスト（local-only.test.mjs, seibutsu.test.ts）
npm run build         # dist/ と SHA-256 の release.json
npm run build:desktop # native desktop 実行ファイル（bundle・署名なし）
```

## 状態と制限（AWAI-3079, 2026-11-17）

- 旧構成（Cloudflare Worker の XRPC dispatcher / adapter、`wrangler.jsonc`、`@etzhayyim/sdk`）は削除しました。履歴は git にあります。
- 検証済み: `npm test`、`npm run build`、`cargo check`（Tauri 3.0.0-alpha.6）。
- 未検証: native window の起動、`tauri build`、Android、署名、配布。アイコンは app-soshiki の仮アイコンです。

## Android（Itonami Store）

AWAI-3502 で Android 版を加えました。Store の ID は `network.itonami.seibutsu` です。

- `npm run build:android`（`scripts/android-apk.sh`）が、`tauri android build --debug --apk --target aarch64` で APK を作り、SHA-256 を表示します。Android SDK・NDK 27・JDK 21・Rust の `aarch64-linux-android` が要ります。
- 出力は `src-tauri/gen/android/app/build/outputs/apk/universal/debug/app-universal-debug.apk`（約 20 MB、手元の Android debug 鍵で署名）です。
- manifest に `INTERNET` 権限はありません。OS が app の通信をすべて止めます。
- ネイティブライブラリは 16 KB 境界にそろえ（Android 15 以降の 16 KB ページ）、debug 情報を除いています。画面はシステムバーと切り欠きを避け、明暗の配色に合わせます。
- Store への掲載は network-awai/awai-os の `android/local-apps.json` と `build.py --local-apps-dir` で行います。
- 確認済み: APK の作成、パッケージ名・権限・署名・16 KB 境界の確認。エミュレータでの起動は awai-os 側の記録を見てください。実機・署名済みの配布は未確認です。
