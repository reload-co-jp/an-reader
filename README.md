# AN Reader

AI/LLM 最新情報アグリゲーションサイト

https://an-reader.reload.co.jp

- 想定用途: LLMの新モデル、AIプラットフォーム更新、価格改定、キャンペーン、割引、重要OSS/研究動向を自動収集し、重要度順に確認できる専用ニュースサイトを構築する
- 想定実装者: Codex / Claude Code / 開発者
- 優先度: まずはMVPを最短で動かし、その後に通知・重複排除・検索・個人最適化を追加する

---

## 1. 目的

AI/LLM領域では、以下の情報を手動で追うのが難しくなっている。

- 新モデルのリリース
- API / SDK / プラットフォームの更新
- 価格改定
- 無料枠変更
- クレジット配布や割引キャンペーン
- 新しい主要OSSモデルやフレームワーク
- 重要な研究発表や実運用に影響する発表

本システムは、これらの情報を複数ソースから自動収集し、要約・分類・重要度判定・重複排除したうえで、ニュースサイト形式で一覧・検索・絞り込み・通知できるようにすることを目的とする。

---

## 2. ゴール

### 2.1 ユーザーゴール

ユーザーは以下を実現したい。

- LLMやAIプラットフォームの最新情報を取りこぼさず把握できる
- 重要な更新だけを優先的に見られる
- 割引やキャンペーン情報を別枠で確認できる
- 同じ内容の重複ニュースに時間を奪われない
- 後から「いつ、どこで、何が変わったか」を検索できる

### 2.2 プロダクトゴール

- 自動収集されたAIニュースを1か所に集約する
- 収集データを構造化して保存する
- LLMを用いてカテゴリ分類・要約・重要度スコアリングを行う
- Webサイトとして一覧性高く表示する
- 将来的に通知やパーソナライズにも拡張できる構成にする

---

## 3. スコープ

### 3.1 MVPスコープ

MVPでは以下を実装する。

- 複数ソースからの定期収集
- 記事データの正規化
- LLMによる要約・カテゴリ分類・重要度判定
- 重複候補の簡易判定
- JSONベースの保存
- ニュースサイト形式の一覧表示
- カテゴリ別表示
- 重要ニュース表示
- Deal / Discount専用表示
- 検索（タイトル・要約・タグの簡易全文検索）
- 手動再収集の管理用エンドポイントまたはCLI

### 3.2 初期スコープ外

初期実装では以下は必須ではない。

- ユーザーアカウント機能
- SNSログイン
- コメント機能
- ブックマーク同期
- 高度なレコメンド
- 多言語UI
- 有料課金機能
- 本格的なベクトルDB

### 3.3 拡張候補

- Slack / Discord / Email通知
- Embedding検索
- ソース信頼度学習
- ユーザーごとの興味カテゴリ設定
- 管理画面
- X投稿収集の高度化
- スポンサー/アフィリエイト連携

---

## 4. 想定ユーザー

### 4.1 主要ユーザー

- AIプロダクト開発者
- エンジニア
- 起業家
- テックリサーチャー
- AI活用を検討する事業責任者

### 4.2 利用シーン

- 毎朝、重要ニュースだけ短時間で確認したい
- 新モデル公開や価格改定をすぐ知りたい
- APIやプラットフォームの仕様変更を見逃したくない
- 割引やクレジット配布を取り逃したくない
- 最近の重要トピックを後から検索したい

---

## 5. コンセプト

本サイトは単なるRSSリーダーではなく、以下の思想で設計する。

- 広く集める
- 構造化する
- LLMで絞る
- 重要度順に見せる
- Deal系を分離する
- 同一話題の重複を抑える

つまり「AIニュースの受信箱」ではなく、「使える情報だけが残るダッシュボード」を目指す。

---

## 6. 情報ソース要件

### 6.1 初期対応ソース

初期対応候補は以下。

#### 公式ブログ / 公式更新ページ

- OpenAI
- Anthropic
- Google AI / Google Cloud AI
- Microsoft Azure AI
- AWS AI / Bedrock
- Meta AI
- Hugging Face
- Mistral AI
- Cohere
- Perplexity
- Replicate
- Stability AI
- xAI
- Groq
- Together AI
- Fireworks AI
- Cerebras

#### OSS / 開発系

- Hugging Face model announcements
- GitHub Releases
- GitHub Trending（AI関連のみ抽出）

#### キュレーション / メディア

- AI系ニュースレターの公開記事
- Product Hunt のAIカテゴリ
- Hacker News のAI関連投稿
- Reddit のAI関連Subreddit

#### Deal / Campaign / Pricing系

- 価格改定ページ
- キャンペーン告知ページ
- コミュニティ投稿
- Product Hunt / SaaS Deals系ページ

### 6.2 ソース種別

各ソースは以下のいずれかで収集する。

- RSS
- Atom
- HTMLスクレイピング
- 公開API
- GitHub API
- 手動登録URL

### 6.3 ソース設定要件

各ソースには以下の設定を持たせる。

- source_id
- source_name
- source_type
- base_url
- feed_url
- enabled
- polling_interval_minutes
- language
- trust_score
- parser_type
- parser_config
- tags_default

---

## 7. システム全体構成

## 7.1 全体フロー

```text
Scheduler
  -> Source Fetcher
  -> Raw Content Store
  -> Normalizer
  -> LLM Analyzer
  -> Duplicate Detector
  -> Structured Store
  -> Web Frontend / Notification
```

## 7.2 レイヤー構成

### A. 収集レイヤー

- 定期実行
- ソースごとの取得
- 生データ保存

### B. 正規化レイヤー

- タイトル・本文・URL・日時・出典を統一フォーマットへ変換

### C. AI解析レイヤー

- 要約
- 分類
- 重要度判定
- Deal判定
- 重複候補判定補助

### D. 保存レイヤー

- JSON
- 生データと整形済みデータを分離保存

### E. 表示レイヤー

- 一覧ページ
- 詳細ページ
- カテゴリページ
- Deal一覧
- 検索ページ

### F. 通知レイヤー（拡張）

- 高重要度のみ通知
- Dealのみ通知

---

## 8. 技術スタック方針

### 8.1 MVP推奨スタック

- Frontend: Next.js
- Styling: CSS in JSX
- Data: JSONファイル
- Fetch/ETL: TypeScript
- Scheduler: GitHub Actions
- LLM API: OpenAI API または互換API
- Hosting: GitHub Pages / 静的ホスティング

### 8.2 推奨理由

- まずはDB不要で速く作れる
- JSON管理により差分確認しやすい
- 静的サイト化しやすい
- 後からDBへ移行しやすい

### 8.3 後の拡張候補

- DB: Supabase / Postgres
- Search: Meilisearch / Typesense / Vector DB
- Queue: Redis / Cloud Tasks
- Worker: serverless function or background worker

---

## 9. データモデル設計

### 9.1 RawItem

収集直後の生データ。

```ts
export type RawItem = {
  id: string
  sourceId: string
  fetchedAt: string
  originalUrl: string
  originalTitle?: string
  rawHtml?: string
  rawText?: string
  rawJson?: unknown
  publishedAt?: string
  author?: string
}
```

### 9.2 NewsItem

サイト表示用の正規化済みデータ。

```ts
export type NewsItem = {
  id: string
  canonicalUrl: string
  title: string
  sourceId: string
  sourceName: string
  sourceType: "official" | "media" | "community" | "oss" | "pricing" | "deal"
  publishedAt: string | null
  fetchedAt: string
  language: string | null
  contentText: string
  excerpt: string | null
  tags: string[]
  categories: NewsCategory[]
  importanceScore: number
  importanceLevel: "critical" | "high" | "medium" | "low"
  isDeal: boolean
  dealType: DealType | null
  companies: string[]
  products: string[]
  summaryShort: string
  summaryLong: string
  actionPoints: string[]
  pricingChange: PricingChange | null
  releaseInfo: ReleaseInfo | null
  duplicateGroupId: string | null
  relatedItemIds: string[]
  trustScore: number
  llmModel: string
  analysisVersion: string
  createdAt: string
  updatedAt: string
}
```

### 9.3 Category定義

```ts
export type NewsCategory =
  | "model_release"
  | "platform_update"
  | "api_update"
  | "sdk_update"
  | "pricing_change"
  | "deal_campaign"
  | "open_source"
  | "research"
  | "benchmark"
  | "security"
  | "policy"
  | "funding"
  | "tooling"
  | "other"
```

### 9.4 DealType定義

```ts
export type DealType =
  | "discount"
  | "free_credit"
  | "extended_trial"
  | "coupon"
  | "limited_campaign"
  | "price_drop"
  | "bundle"
  | "other"
```

### 9.5 PricingChange

```ts
export type PricingChange = {
  hasPricingChange: boolean
  direction: "up" | "down" | "mixed" | "unknown"
  productName: string | null
  oldPriceText: string | null
  newPriceText: string | null
  note: string | null
}
```

### 9.6 ReleaseInfo

```ts
export type ReleaseInfo = {
  hasRelease: boolean
  company: string | null
  productName: string | null
  version: string | null
  releaseType: "model" | "api" | "feature" | "platform" | "sdk" | "other" | null
  announcedAt: string | null
}
```

### 9.7 SourceConfig

```ts
export type SourceConfig = {
  id: string
  name: string
  type: "rss" | "atom" | "html" | "api" | "github" | "manual"
  baseUrl: string
  feedUrl?: string
  enabled: boolean
  pollingIntervalMinutes: number
  language?: string
  trustScore: number
  parserType: string
  parserConfig?: Record<string, unknown>
  tagsDefault?: string[]
}
```

---

## 10. 保存方式

### 10.1 MVP保存構成

```text
/data
  /sources
    sources.json
  /raw
    2026-04-22T10-00-00Z_openai_xxx.json
  /items
    2026-04-22_item_xxx.json
  /indexes
    latest.json
    by-category.json
    by-source.json
    by-date.json
    deals.json
```

### 10.2 代替案

MVPの次段階ではSQLiteも可能。

- JSON: 実装が速い、Git差分が見やすい
- SQLite: 検索と参照が少し楽
- Supabase/Postgres: 将来の多ユーザー対応向け

### 10.3 保存要件

- Rawと整形済みデータを分ける
- 再解析できるように原文を可能な範囲で残す
- analysisVersionを持ち、後から再判定可能にする

---

## 11. 収集機能要件

### 11.1 定期収集

- 1時間ごとに定期収集を実行する
- 高速に動き、失敗ソースがあっても全体処理を止めない
- ソースごとに最後の取得時刻を記録する

### 11.2 収集処理

- RSS / Atom の取得
- HTMLページの取得と本文抽出
- 公開APIの取得
- GitHub Release / GitHub API収集

### 11.3 重複取得防止

以下のいずれかで既取得判定する。

- canonical URL一致
- title + publishedAt + source一致
- content hash一致

### 11.4 リトライ

- 取得失敗時は最大3回リトライ
- 4xxは必要に応じて即失敗扱い
- 5xxやタイムアウトは指数バックオフ

### 11.5 ログ

- source名
- 件数
- 成功/失敗
- 実行時間
- エラー内容

---

## 12. 本文抽出・正規化要件

### 12.1 正規化項目

抽出対象。

- title
- canonicalUrl
- publishedAt
- sourceName
- contentText
- excerpt
- author
- tags候補

### 12.2 本文抽出ルール

- script / style / nav / footer など不要要素を除外
- 本文長が極端に短い場合は失敗扱い候補にする
- 本文が取れない場合はdescriptionやsnippetで代用可能

### 12.3 日付処理

- タイムゾーンはUTC保存
- 表示時にローカルタイムへ変換
- 日付不明の記事は null 許容

---

## 13. AI解析要件

### 13.1 解析目的

- 要約生成
- カテゴリ分類
- 重要度判定
- Deal判定
- 価格改定の有無検出
- リリース情報抽出
- アクションポイント生成

### 13.2 出力形式

LLMはJSON固定で返す。

```json
{
  "summaryShort": "...",
  "summaryLong": "...",
  "categories": ["model_release"],
  "importanceScore": 0.94,
  "importanceReason": "新しい主要モデルの正式リリースで実務影響が大きい",
  "isDeal": false,
  "dealType": null,
  "companies": ["OpenAI"],
  "products": ["GPT-5.x"],
  "actionPoints": [
    "API価格と制限を確認する",
    "既存プロダクトへの適用可否を検討する"
  ],
  "pricingChange": {
    "hasPricingChange": false,
    "direction": "unknown",
    "productName": null,
    "oldPriceText": null,
    "newPriceText": null,
    "note": null
  },
  "releaseInfo": {
    "hasRelease": true,
    "company": "OpenAI",
    "productName": "GPT-5.x",
    "version": "5.x",
    "releaseType": "model",
    "announcedAt": null
  },
  "tags": ["llm", "release", "api"]
}
```

### 13.3 重要度スコア方針

0.0〜1.0の範囲で算出する。

#### 高スコアに寄せる条件

- 新モデル正式発表
- API / 価格 / 利用制限変更
- 無料枠や割引の追加・終了
- 実装影響の大きいSDK更新
- セキュリティや規約変更

#### 低スコアに寄せる条件

- 単なる雑談や感想
- 宣伝色のみ強い内容
- 技術的/実務的影響が小さいもの
- 他媒体からの重複転載

### 13.4 importanceLevel変換

- 0.90以上: critical
- 0.75以上: high
- 0.45以上: medium
- それ未満: low

### 13.5 LLMプロンプト要件

- 必ずJSONのみ返すよう制約する
- 推測で断定しない
- 不明な場合は null を返す
- Deal / Pricing / Releaseは見逃しを減らす
- タイトルだけでなく本文も見て判定する

### 13.6 失敗時フォールバック

- JSON parse失敗時は再試行
- 解析失敗時は簡易ルールベース分類へフォールバック
- LLM API障害時でもRaw保存は継続する

---

## 14. 重複排除要件

### 14.1 目的

同一ニュースが複数メディア・コミュニティ・転載で流れてくるため、重複を減らす。

### 14.2 MVP重複判定

以下の組み合わせで判定する。

- canonical URL一致
- 正規化タイトル類似
- 公開日近接
- 本文hash類似

### 14.3 拡張重複判定

将来的にEmbeddingによる近似判定を追加する。

### 14.4 重複時の扱い

- 代表記事を1件選ぶ
- 関連記事として紐づける
- 一覧では原則代表記事のみ表示
- 詳細では関連ソースを表示可能にする

---

## 15. フロントエンド要件

### 15.1 サイト構成

最低限のページ。

- `/` トップ
- `/news` 一覧
- `/news/[id]` 詳細
- `/categories/[category]` カテゴリ別一覧
- `/deals` Deal / Discount一覧
- `/sources` ソース一覧
- `/search` 検索結果

### 15.2 トップページ要件

表示ブロック。

- 重要ニュース
- 最新ニュース
- Deal / Discount
- モデルリリース
- 価格改定
- OSS / Research
- ソース別の最近の更新

### 15.3 一覧カード表示項目

- タイトル
- ソース名
- 公開日時
- summaryShort
- カテゴリタグ
- importanceLevel
- isDealバッジ
- 会社名 / 製品名

### 15.4 詳細ページ要件

- タイトル
- 出典リンク
- ソース情報
- 公開日時
- 要約（short/long）
- カテゴリ
- 重要度
- アクションポイント
- 価格改定情報
- リリース情報
- 関連記事
- 原文への外部リンク

### 15.5 検索要件

MVPでは簡易全文検索。

対象項目。

- title
- summaryShort
- summaryLong
- tags
- companies
- products

### 15.6 絞り込み要件

- カテゴリ
- ソース
- importanceLevel
- Dealのみ
- 会社名
- 期間

### 15.7 ソート要件

- 新着順
- 重要度順
- Deal優先

---

## 16. UI/UX要件

### 16.1 デザイン方針

- テキスト中心で一覧性を高くする
- 重要なものがすぐ目に入る
- Deal系は明確に分離する
- 派手さより運用性重視

### 16.2 バッジ例

- `CRITICAL`
- `HIGH`
- `DEAL`
- `PRICE CHANGE`
- `MODEL RELEASE`
- `API UPDATE`

### 16.3 視認性

- 重要度によってカードの強調度を変える
- Critical / Highは上部固定または先頭表示
- Dealは色やアイコンで区別する

### 16.4 モバイル対応

- スマホでニュース確認しやすいこと
- カードの高さを抑える
- フィルタはドロワーでよい

---

## 17. 管理・運用機能要件

### 17.1 管理対象

- ソースの有効/無効
- 手動収集実行
- 再解析実行
- 失敗ログ確認
- ブラックリストURL管理

### 17.2 管理UI

MVPでは管理UI不要でも可。
CLIまたは簡易管理ページで代替してよい。

### 17.3 再解析

- analysisVersionを更新した場合、既存記事に対して再解析できる
- 再解析後はupdatedAtを更新する

---

## 18. 通知機能要件（拡張）

### 18.1 通知条件

以下を通知対象とする。

- importanceLevel = critical
- isDeal = true
- pricingChange.hasPricingChange = true
- category に model_release を含む

### 18.2 通知先候補

- Slack
- Discord
- Email
- Telegram

### 18.3 通知内容

- タイトル
- 要約
- 出典
- 重要度
- Deal / Price change / Releaseの別

---

## 19. 非機能要件

### 19.1 パフォーマンス

- トップページ初回表示は軽量であること
- 一覧は静的生成可能であること
- JSONが肥大化した場合は index を分割すること

### 19.2 可用性

- 一部ソース取得失敗でも全体が継続する
- LLM解析失敗でも最低限の一覧表示はできる

### 19.3 保守性

- ソース追加を容易にする
- パーサを差し替えやすくする
- 分析プロンプトのバージョン管理ができる

### 19.4 拡張性

- JSONからSQLite / DBへ移行可能な構造にする
- 通知機能や個人設定を追加しやすい構成にする

### 19.5 セキュリティ

- APIキーはenv管理
- 管理系エンドポイントには保護をつける
- 外部HTMLの表示時はサニタイズする

---

## 20. ディレクトリ構成案

```text
project-root/
  app/
    page.tsx
    news/
      page.tsx
      [id]/page.tsx
    categories/
      [category]/page.tsx
    deals/
      page.tsx
    sources/
      page.tsx
    search/
      page.tsx
  components/
    NewsCard.tsx
    DealCard.tsx
    FilterBar.tsx
    SourceBadge.tsx
    ImportanceBadge.tsx
    SearchBox.tsx
  lib/
    data.ts
    search.ts
    filters.ts
    scoring.ts
  scripts/
    fetch-all.ts
    fetch-source.ts
    normalize.ts
    analyze.ts
    dedupe.ts
    build-indexes.ts
  data/
    sources/
      sources.json
    raw/
    items/
    indexes/
  prompts/
    analyze-news.md
  types/
    news.ts
    source.ts
  public/
  package.json
  README.md
```

---

## 21. バッチ処理設計

### 21.1 実行順

```text
1. sources.json を読む
2. enabledなソースを巡回
3. 各ソースから記事候補取得
4. Raw保存
5. 正規化
6. 既存重複チェック
7. 新規記事のみLLM解析
8. 重複グループ判定
9. items保存
10. index再生成
```

### 21.2 インデックス生成

生成対象。

- latest.json
- by-category.json
- by-source.json
- deals.json
- critical.json
- search-index.json

### 21.3 失敗分離

- 収集失敗
- 本文抽出失敗
- LLM解析失敗
- 保存失敗

失敗種別ごとにログ・再実行可能にする。

---

## 22. 画面要件詳細

## 22.1 トップページ

### セクション

- Hero（サイト説明は控えめ）
- Critical / High priority
- New model releases
- Deals / Discounts
- Pricing changes
- Latest updates
- By source

### 要件

- 重要ニュースを最上部に表示
- 7日以内の重要情報を優先表示
- Dealは目立つ位置に表示

## 22.2 ニュース一覧ページ

### 要件

- ページングまたは無限スクロール
- 絞り込み可能
- URLクエリで状態保持

例:

```text
/news?category=model_release&level=high&deal=true
```

## 22.3 詳細ページ

### 要件

- 構造化情報を見やすく表示
- 原文リンクへ遷移可能
- 関連記事と関連ソースを表示

## 22.4 Deal一覧

### 要件

- Deal / Discount / Free credit / Trial延長を一覧化
- 終了時期が読める場合は表示
- 期限切れはアーカイブ可能

---

## 23. API / データ供給インターフェース案

フロントが静的生成前提でも、将来API化しやすいように設計する。

### 23.1 API例

- `GET /api/news`
- `GET /api/news/:id`
- `GET /api/deals`
- `GET /api/categories/:category`
- `GET /api/search?q=...`
- `POST /api/admin/fetch`
- `POST /api/admin/reanalyze`

MVPではファイル読み込みベースでもよい。

---

## 24. スコアリングルール案

LLM任せにしきらず、ルールベース補正を行う。

### 24.1 補正例

- 公式ソースなら +0.10
- pricing_change を含むなら +0.15
- model_release を含むなら +0.20
- isDeal = true なら +0.10
- 信頼度の低いコミュニティソースのみなら -0.10
- タイトルが釣り気味で本文が薄いなら -0.15

### 24.2 最終スコア

```text
finalImportance = min(1.0, max(0.0, llmScore + ruleAdjustments))
```

---

## 25. 検索仕様

### 25.1 MVP検索

- クライアント側またはサーバ側で全文検索
- 日本語/英語混在を許容
- タイトル一致を強めに重み付け

### 25.2 拡張検索

- embeddingベース類似検索
- 「最近のOpenAIの価格変更」など自然文検索
- 会社名やモデル名の表記揺れ対応

---

## 26. SEO・共有要件

### 26.1 SEO

- 各記事詳細にmeta title / description
- OGP画像は共通でもよい
- 構造化データは拡張時対応

### 26.2 共有

- 詳細ページのURL共有
- Deal一覧の共有
- クエリ付き検索結果共有は拡張でよい

---

## 27. 監視・ログ要件

### 27.1 監視対象

- 定期実行成功率
- ソースごとの取得件数
- LLM解析失敗率
- 直近24時間の新規件数

### 27.2 ログ出力

- fetch log
- normalize log
- analyze log
- dedupe log
- build log

### 27.3 アラート

以下で通知候補。

- 全ソース取得失敗
- 一定時間新規記事ゼロ
- LLM解析失敗率急増

---

## 28. 初期ソース登録例

```json
[
  {
    "id": "openai_blog",
    "name": "OpenAI Blog",
    "type": "rss",
    "baseUrl": "https://openai.com",
    "feedUrl": "https://openai.com/news/rss.xml",
    "enabled": true,
    "pollingIntervalMinutes": 60,
    "language": "en",
    "trustScore": 0.98,
    "parserType": "rss"
  },
  {
    "id": "anthropic_news",
    "name": "Anthropic News",
    "type": "html",
    "baseUrl": "https://www.anthropic.com",
    "enabled": true,
    "pollingIntervalMinutes": 60,
    "language": "en",
    "trustScore": 0.97,
    "parserType": "html_list"
  }
]
```

※ 実URLやフィード有無は実装時に最新確認のうえ設定すること。

---

## 29. LLM解析プロンプト要件詳細

### 29.1 システムプロンプト方針

- あなたはAIニュース分析エンジンである
- 返答はJSONのみ
- 不確実なものはnull
- 重要度は「実務・開発・課金・運用への影響」で評価
- Deal / pricing / release を見逃さない
- 誇張表現に引きずられない

### 29.2 入力項目

- title
- sourceName
- sourceType
- publishedAt
- contentText
- url

### 29.3 出力バリデーション

- JSON schemaで検証
- enumにないカテゴリは禁止
- scoreは0〜1に収める

---

## 30. 受け入れ条件

### 30.1 MVP受け入れ条件

以下を満たしたらMVP完了。

- 複数ソースから定期収集できる
- 記事がJSONとして保存される
- 各記事に要約・カテゴリ・重要度が付く
- トップページで重要ニュースとDealが確認できる
- カテゴリ別一覧が見られる
- 検索ができる
- 重複記事が一定程度まとまる

### 30.2 品質基準

- 重要ニュースが埋もれない
- Deal情報が判別しやすい
- 同一話題の連投が過度に並ばない
- 収集失敗で全体が止まらない

---

## 31. 実装優先順位

### Phase 1: MVP最小構成

- sources.json
- RSS/HTML収集
- Raw保存
- 正規化
- LLM解析
- JSON保存
- トップ/一覧/詳細/Dealページ

### Phase 2: 実用強化

- 重複排除改善
- 検索改善
- フィルタ強化
- 重要度補正ロジック
- 管理CLI

### Phase 3: 通知と最適化

- Slack通知
- 高重要度通知
- Deal通知
- 再解析バッチ

### Phase 4: 高度化

- Embedding検索
- 表記揺れ吸収
- パーソナライズ
- DB移行

---

## 32. 実装メモ

- まずはソース数を増やしすぎないこと
- 公式ソースと価格系ソースを優先すること
- HTMLパースは壊れやすいのでparser差し替え前提にすること
- LLM解析コストを抑えるため、本文長の上限や要約前処理を設けること
- 既存記事の再解析を見越してanalysisVersionを必ず持つこと
- 長期的には「ニュースサイト」より「意思決定ダッシュボード」に寄せると価値が高い

---

## 33. 将来の差別化ポイント

- AI領域に特化した重要度判定
- Deal / Pricing / Free creditの専用抽出
- 公式発表とコミュニティ反応の分離表示
- 「今すぐ確認すべきもの」だけを前面表示
- 実務アクションポイント自動生成

---

## 34. 参考実装方針まとめ

### MVPとして最も現実的な構成

- Next.jsでUIを作る
- scriptsで定期収集する
- JSONに保存する
- OpenAI API等で分析する
- static exportまたは通常デプロイで配信する

### この構成の利点

- 早い
- 安い
- 修正しやすい
- 後からDB化しやすい

---

## 35. 最低限必要な環境変数

```env
SITE_URL=
DEFAULT_LOCALE=ja
DEFAULT_TIMEZONE=Asia/Tokyo
CRON_SECRET=
```

- 拡張時

```env
OPENAI_API_KEY=
SLACK_WEBHOOK_URL=
GITHUB_TOKEN=
SUPABASE_URL=
SUPABASE_ANON_KEY=
```

---

## 37. 完了定義

この仕様書に基づき、以下が確認できれば初期完成とする。

- 定期収集される
- 重要ニュースが見える
- Dealが別で見える
- 重複が整理される
- 検索できる
- 後から拡張しやすい

以上
