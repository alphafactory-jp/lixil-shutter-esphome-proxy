# lixil-shutter-esphome-proxy

LIXIL の電動シャッター(アプリ「My Window」で操作する Bluetooth 対応のもの)を、Home Assistant から操作するための **ESPHome Bluetooth プロキシ設定と日本語の手順**です。

統合本体は [nogic1008/ha_lixil_shutter](https://github.com/nogic1008/ha_lixil_shutter) を使います。このリポジトリは、Home Assistant のサーバーがシャッターから遠い・Bluetooth が無い場合に、安い ESP32(M5Stack AtomS3 Lite)を中継機にする部分を扱います。

> English summary: ESPHome Bluetooth proxy config for M5Stack AtomS3 Lite, plus a Japanese setup guide for controlling LIXIL Bluetooth shutters from Home Assistant with [ha_lixil_shutter](https://github.com/nogic1008/ha_lixil_shutter).

これは非公式の手順です。LIXIL および統合の作者とは関係ありません。使用は自己責任でお願いします。

## 構成

```
Home Assistant ──Wi-Fi(ESPHome API)── AtomS3 Lite ──Bluetooth── シャッター
```

- シャッターとの通信は暗号化されています。最初に一度だけ「ペアリング」が必要です(手順 4)。
- iPhone の My Window アプリと併用できます。

## 必要なもの

| もの | 備考 |
| --- | --- |
| M5Stack AtomS3 Lite | 技適あり。USB-C ケーブルと USB 電源も |
| Home Assistant | 2026.11 以降ならマーケットプレイスから統合を入れられます |
| ESPHome | PC にインストール(`pip install esphome`)するか、ESPHome Device Builder を使う |
| シャッターのリモコン | ペアリングモードにするために使います |

## 手順

### 1. 設定ファイルを用意する

```sh
git clone https://github.com/alphafactory-jp/lixil-shutter-esphome-proxy
cd lixil-shutter-esphome-proxy/esphome
cp secrets.yaml.example secrets.yaml
```

`secrets.yaml` に Wi-Fi の SSID・パスワード、API 暗号化キー、OTA パスワードを入れます。API 暗号化キーは [ESPHome の Native API のページ](https://esphome.io/components/api/)で生成できます。`secrets.yaml` は `.gitignore` で除外しているので、誤って公開されません。

### 2. AtomS3 Lite に書き込む

AtomS3 Lite を USB で PC につないで:

```sh
esphome run ble-proxy-atoms3.yaml
```

初回は USB で書き込みます(書き込み先のポートを聞かれたら USB の方を選択)。2 回目以降は Wi-Fi 経由(OTA)で更新できます。

ログに `WiFi Connected` と IP アドレスが出れば成功です。ルーターで IP アドレスを固定(DHCP の固定割り当て)しておくと安心です。

### 3. Home Assistant に追加する

1. 「設定 → デバイスとサービス」に「ESPHome」の「発見」が出るので、「設定」を押します。
2. 手順 1 で作った API 暗号化キーを入力します。
3. 「BLE プロキシ 1」が追加され、Wi-Fi 電波などが見えれば OK です。

次に統合を入れます。Home Assistant 2026.11 以降は「設定 → マーケットプレイス」で `Lixil Bluetooth Shutter` を探してダウンロードします(それより前の版は HACS から)。

### 4. シャッターをペアリングする

1. シャッターをペアリングモードにします。手元の機種では、**リモコンのペアリングボタンを 5 秒押す**と、シャッターが少し動いて合図しました(機種によって違う場合があるので、取扱説明書も確認してください)。
2. **2 分以内**に、Home Assistant の「設定 → デバイスとサービス」に出る「発見」から追加します。出ない場合は「統合を追加」で `Lixil Bluetooth Shutter` を探します。
3. 追加できたら、開・止・閉が動くか確かめます。

> [!IMPORTANT]
> ha_lixil_shutter v0.2.3 では、ESPHome プロキシ経由のペアリングが `is not connected` で失敗することがあります。報告と直し方は [nogic1008/ha_lixil_shutter#95](https://github.com/nogic1008/ha_lixil_shutter/issues/95) にあります。本家で修正された版が出たら、その版を使ってください。

シャッターが複数あるときは、1 台ずつペアリングモードにして追加します。

## 動作確認した環境

- M5Stack AtomS3 Lite、ESPHome 2026.6.5(esp-idf)
- Home Assistant Container 2026.11
- ha_lixil_shutter v0.2.3(上の #95 の修正を手元で適用)
- LIXIL 電動シャッター(統合での製品種別は ShutterEaris)

## よくあること

- **途中で止めると状態が「不明」になる**: シャッターは「全開」と「途中で止まった」を区別して知らせないため、統合は意図して「不明」にしています(本家の仕様)。
- **シャッターが見つからない**: プロキシとシャッターの距離を縮めるか、プロキシを増やします。ESPHome のプロキシは同時につなげる台数に上限があります(既定 3 台)。
- **iPhone アプリが使えなくなった**: 未確認です。手元では Home Assistant と iPhone の両方から操作できています。

## ライセンス

MIT
