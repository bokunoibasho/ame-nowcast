# ame-nowcast

現在地の **高解像度降水ナウキャスト** を地図上にサッと表示する PWA。
iPhone のホーム画面アイコンから、検索・広告なしで雨雲をすぐ確認できることが目的です。

> 気象庁の高解像度降水ナウキャストは精度が高いものの、毎回たどり着くのが手間
> （検索 → 該当ページ → 現在地に設定）です。それをワンタップに短縮します。

- 公開 URL: <https://bokunoibasho.github.io/ame-nowcast/>（GitHub Pages を有効化後に動作）
- ライセンス: [MIT](./LICENSE)

## 機能

- 地理院タイル（淡色地図）の上に、気象庁 高解像度降水ナウキャスト（hrpns）タイルを重ね表示
- 起動時に現在地へ自動センター（ズーム 12）
- 過去 〜 実況 〜 1 時間先予測のスライダー / コマ送り再生
- iOS 用のメタタグでホーム画面追加時にフルスクリーン化
- 最低限の Service Worker で殻だけオフラインでも開ける

## ホーム画面への追加（iPhone）

1. iPhone Safari で公開 URL を開く
2. 位置情報の許可を求められたら許可する
3. 共有メニュー → 「ホーム画面に追加」
4. 以降はホーム画面のアイコンからワンタップで起動

## 動かし方（ローカル）

ビルド不要の静的サイトです。HTTPS（または `localhost`）必須なのは、Geolocation と Service Worker が
セキュアコンテキストでしか動かないためです。

```sh
# 例: 標準ライブラリだけで簡易サーバを立てる
python3 -m http.server 8000
# → http://localhost:8000/ で開く
```

## 技術スタック

- 単一 HTML 中心の静的サイト（ビルド不要）
- 地図ライブラリ: [Leaflet](https://leafletjs.com/) 1.9.4（CDN）
- 背景地図: [地理院タイル（淡色地図）](https://maps.gsi.go.jp/development/ichiran.html#pale)
- 雨雲レイヤー: 気象庁 高解像度降水ナウキャスト（hrpns）タイル
- ホスティング: GitHub Pages を想定（Cloudflare Pages でも可。HTTPS 必須）

## 出典・データ提供元

- この地図は [国土地理院 地理院タイル](https://maps.gsi.go.jp/development/ichiran.html) を利用しています。
- 降水データは [気象庁 高解像度降水ナウキャスト](https://www.jma.go.jp/bosai/nowc/) を利用しています。

## 注意: 非公式エンドポイントの利用について

本アプリが利用している以下のエンドポイントは、気象庁が公式に「API」として公開しているものではなく、
ナウキャストのページが内部的に使っている経路です。**予告なく仕様変更・廃止される可能性があります**
（過去にも URL 構造の変更例があります）。動かなくなった場合は気象庁公式サイトをご利用ください。

- 時刻一覧 (JSON):
  - 実況: `https://www.jma.go.jp/bosai/jmatile/data/nowc/targetTimes_N1.json`
  - 予測: `https://www.jma.go.jp/bosai/jmatile/data/nowc/targetTimes_N2.json`
- タイル画像: `https://www.jma.go.jp/bosai/jmatile/data/nowc/{basetime}/none/{validtime}/surf/hrpns/{z}/{x}/{y}.png`

レスポンスは降順なので、利用側で昇順に並べ替えてから使います。

## デプロイ手順（GitHub Pages）

1. このリポジトリを `bokunoibasho/ame-nowcast` として GitHub に push（`main` ブランチ）
2. Settings → Pages → Build and deployment で **Source: Deploy from a branch**、**Branch: `main` / `/ (root)`** を選択
3. 数分後に `https://bokunoibasho.github.io/ame-nowcast/` で公開される
4. iPhone Safari で開いて、位置情報許可 → ホーム画面追加で実機確認

## 設計上の判断メモ

- **Apple Map（MapKit JS）は不採用**: 技術的には可能だが、Apple Developer Program（年 99 ドル）が
  必須で、トークン署名の手間もあり、個人ツールには割が合わないため見送り。
- **`open-` 接頭辞は付けない**: 既存クローズド製品の代替を名乗る慣習であり、本件は該当しない。
  OSS であることは public + MIT LICENSE で十分伝わる。
- **`rain-` ではなく `ame-`**: データ元が気象庁で本質的に日本専用のため、名前も日本由来が実態に合う。
- **地図ライブラリは Leaflet**: 今回はラスタータイルを重ねるだけなので、MapLibre よりも軽量な Leaflet が最適。

## 将来の拡張アイデア

- 雷・竜巻発生確度ナウキャストのレイヤー切り替え（同じ jmatile 系の別 elements）
- 雨雲の濃さ（透明度）と起動ズームの設定 UI
- 「○分後に雨」通知（PWA の Push。iOS の対応状況に依存するので要調査）
