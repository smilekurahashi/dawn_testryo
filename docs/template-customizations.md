# Template Customizations

このドキュメントは、Dawnの既存テンプレートファイルへ加えた直接編集の
意図と内容を記録するためのものです。
CLAUDE.mdの「原則1：既存ファイルの直接編集は最終手段」に該当する変更は、
すべてここに記載してください。

---

## templates/product.json

### 変更概要

商品ページにブランドストーリー演出セクションを2つ、すべての商品に対して
コードレベルで常設するため、`templates/product.json` を直接編集しました。

これは「テーマカスタマイザーで個別商品ごとに追加してもらう」運用も可能でしたが、
以下の経営判断によりコードレベルでの構成管理を優先しています：

- 全商品で一貫したブランド体験を提供する
- 設定をリポジトリ管理下に置き、Git経由で履歴・レビューを残す
- 環境（staging/production）間で同じテンプレートを再現可能にする

### 追加したセクション

| キー | type | 役割 |
|------|------|------|
| `custom_product_story` | `custom-product-story` | 商品の物語（リード文・見出し・本文・画像）を語るセクション |
| `custom_craftsmanship` | `custom-craftsmanship` | 「目利き／熟成／手切り」の3つの職人技を漢数字装飾で並べるセクション |

### `order` 配列の変更

変更前:

```json
["main", "image-with-text", "multicolumn", "related-products"]
```

変更後:

```json
["main", "custom_product_story", "custom_craftsmanship",
 "image-with-text", "multicolumn", "related-products"]
```

`main`（商品本体）の直後に2セクションを差し込み、その後の Dawn デフォルト
セクション（`image-with-text` / `multicolumn` / `related-products`）は順序・
設定とも無改変。

### デフォルト設定値の根拠（コピーライティング）

#### custom_product_story
- `lead`: 「七十年受け継がれる、一品の物語」
  - ブランドの歴史を端的に示すリード。70年（昭和30年=1955年創業）の継続性を強調。
- `heading`: 「この一品について」
  - 商品個別の物語に誘導する控えめな導入。商品名と被らないよう抽象的に。
- `body`: 創業ストーリー。商品ページでの初回露出として店舗の出自を伝える。
  - 商品単位で個別化したい場合は将来 metafield 化（後述「拡張方法」参照）。
- `bg_color`: `paper`（和紙風ベージュ #EFE8DA）
  - main セクションの白背景との対比で「読み物」の領域に視覚的に切り替え。
- `padding_size`: `normal`
  - 仕様書では `"medium"` が指定されていたが、セクションschemaの選択肢は
    `normal` / `wide` / `extra` の3段階。最も近い `normal` を採用。

#### custom_craftsmanship
- `heading`: 「職人の手仕事」
- 3ブロックは老舗肉屋の典型的な3工程「目利き → 熟成 → 手切り」。
- 装飾文字は漢数字「壱・弐・参」で和の格式を表現。

### スキーマ・キーのマッピング注記

要件書のJSONスニペットで指定されていた一部のキー名は、前タスクで作成した
セクションschemaの実際のキーと異なっていたため、以下のマッピングで実装しました。
（schemaに存在しないキーをJSONに書いてもShopifyは黙って無視するため、
 デフォルト値を効かせるには実schemaのキーで書く必要があります）

| 要件書のキー | 実際のschemaキー | 備考 |
|-------------|-----------------|------|
| `lead_text` | `lead` | |
| `show_vertical_text` | `vertical_text` | |
| `background_color` | `bg_color` | |
| `section_padding: "medium"` | `padding_size: "normal"` | 値の選択肢は normal/wide/extra |
| block `type: "craftsmanship_item"` | `craft_item` | |
| block setting `decoration` | `symbol` | |
| block setting `subheading` | `title` | |

### 将来的に変更したい場合の対応箇所

- **コピー文言の変更**：`templates/product.json` の該当セクション `settings`
  を編集してコミット。
- **個別商品ごとに本文を差し替えたい**：
  `body` を商品の metafield 参照に切り替えるため、セクション側
  `sections/custom-product-story.liquid` で
  `product.metafields.custom.story` 等を優先表示するロジックを追加し、
  schema の `body` をフォールバックとして残す方針が望ましい。
- **特定商品だけセクションを非表示にしたい**：
  セクションLiquid側で `product.metafields.custom.hide_story == true` を
  チェックして早期 return するロジックを追加。
- **商品カテゴリ単位で出し分け**：
  collection / product type 判定をセクション内で実装するか、
  商品テンプレートを `product.story.json` のように別出しにし、
  Shopify管理画面で商品ごとにテンプレートを割り当てる。

### Dawn アップデート時の注意点

- `templates/product.json` は **Dawn 公式が更新する可能性が高い**ファイル。
  upstream merge 時は必ずコンフリクトが発生する前提で対応する。
- merge 時の手順：
  1. 公式の最新 `product.json` を取り込む
  2. `main` セクションの設定変更（新規block追加など）を取り込む
  3. `custom_product_story` / `custom_craftsmanship` を `main` の直後に
     再挿入する
  4. `order` 配列に2キーを `main` の直後に挿入する
  5. 既存セクション（`image-with-text` 等）の有無は upstream に従う
- セクションファイル本体（`sections/custom-*.liquid`）は Dawn が触らないため
  衝突しない。schema変更があった場合は `templates/product.json` の
  settings/blocks 側の追従が必要。
- このドキュメント（`docs/template-customizations.md`）も合わせて更新する。
