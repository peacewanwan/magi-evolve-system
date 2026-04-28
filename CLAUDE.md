# MAGI EVOLVE SYSTEM — note記事生成パイプライン

## 作業開始前に必ず読むこと

以下を順番に実行してから着手:

1. **セッション引き継ぎの読込**
   - 下記「セッション引き継ぎ」セクション + 共通仕様書 `_handoffs/README.md`（v0.5、サイズ肥大化リマインド追加）に従う
   - `_handoffs/handoff-magi-*.md` の最新（自リポ分）
   - `_handoffs/handoff-eventmap-*.md` の最新（event-map 側分）
   - `_handoffs/handoff-meta-*.md` の最新（meta 側分）
   - 他リポ側に自リポの作業方針に影響する変更があれば1行報告
   - 読んだ handoff（自・他いずれも）に `## Codex メモ` セクションがあれば、1〜2 行で要約してユーザーに先に共有（v0.4 から）
   - `_handoffs/` 直下サイズチェック：`ls -l _handoffs/handoff-*.md 2>/dev/null | awk '{sum+=$5} END {printf "%.2f MB (%d files)\n", sum/1024/1024, NR}'` で合計サイズ取得（`du` は iCloud 下で 0B 誤計測のため不可）、**5MB 超**ならユーザーに「N MB に達しています、古いものを archive に移動しますか？」と 1 行提案。5MB 未満はサイレント（v0.5 から）

2. **初回セットアップチェック**
   - 必須ファイル（`NOTE_CONCEPT.md` / `STYLE_GUIDE.md` / `voice_profile.md`）が欠けていたら下記「初回セットアップ（オンボーディング）」発火

## セッション終了時に必ず書くこと

以下のトリガー文言で handoff 作成が発火:

- 「引き継いで」「引き継ぎ作って」「引き継ぎお願い」「引き継ぎよろしく」
- 「セッション切り替える（から引き継いで）」
- 「ハンドオフ作って」「handoff 作成」

handoff MD を `_handoffs/handoff-magi-YYYY-MM-DD-HHMM-{descriptor}.md` に作成する。

詳細は下記「セッション引き継ぎ」セクション + 共通仕様書 `_handoffs/README.md` を参照。

曖昧なケース（話題として出ただけ等）は「ハンドオフ MD を作成しますか？」と1行確認してから発火する。

---

## 初回セットアップ（オンボーディング）

**このセクションはClaude Codeが最初に読む。以下の条件に該当する場合、他の作業より先にオンボーディングを実行すること。**

### 起動時チェック

以下の3ファイルの存在と内容を確認する:

1. `NOTE_CONCEPT.md` — 存在しない or 中身が空 or `{` プレースホルダーが残っている
2. `STYLE_GUIDE.md` — 存在しない or 中身が空
3. `voice_profile.md` — 存在しない or 中身が空

**1つでも該当したら、オンボーディングを開始する。**

### オンボーディングの進め方

ユーザーに対話で以下を聞く。一度に全部聞かず、1-2問ずつ自然に会話する。

**Phase 1: あなたについて（→ NOTE_CONCEPT.md + voice_profile.md の素材）**

1. 「noteでどんな記事を書きたいですか？テーマや方向性を教えてください」
2. 「読者にどんな人物として認識されたいですか？（例: 健康オタクのおじさん、AI実験好きなエンジニア、等）」
3. 「普段の文章はどんなトーンですか？（例: カジュアル、硬め、自虐混じり、等）」
4. 「一人称は何を使いますか？（僕/私/自分/俺/etc）」

**Phase 2: 文章の癖（→ voice_profile.md + STYLE_GUIDE.md の素材）**

5. 「過去に書いたブログ、SNS投稿、メールなど、あなたの文章が残っているものはありますか？あればファイルパスやURLを教えてください。分析してvoice_profileを作ります」
6. 「文章で絶対にやりたくないことはありますか？（例: 絵文字を使う、です/ます調で書く、等）」

**Phase 3: ナレッジベース（→ knowledge-base/ の設定）**

7. 「Obsidianや他のノートツールを使っていますか？使っていればvaultのパスを教えてください。リンクします」
8. 「ノートツールがなくても、自分の経験・思考をまとめたファイルがあれば knowledge-base/ に置いてください」

### ファイル生成

対話で得た情報から以下を自動生成する:

- **NOTE_CONCEPT.md** — `templates/style_guide_template.md` は参考にしつつ、対話内容から生成
- **STYLE_GUIDE.md** — 最低限の文体ルール。薄くてOK。calibratorが育てる
- **voice_profile.md** — 過去の文章があれば分析して生成。なければ対話内容から骨格だけ作る

生成後、ユーザーに内容を見せて確認を取る。「後からいつでもClaudeと対話して更新できます」と伝える。

### プロフィール更新

オンボーディング後も、ユーザーが以下のような発言をした場合は該当ファイルを更新する:
- 「文体を変えたい」「トーンを変えたい」→ STYLE_GUIDE.md
- 「こういう人物像にしたい」「発信の軸を変えたい」→ NOTE_CONCEPT.md
- 「この癖を直したい」「この表現は使いたくない」→ voice_profile.md / STYLE_GUIDE.md

---

## プロジェクト概要

ナレッジベースを元にnote.com記事を半自動生成するパイプライン。

**基本方針：**
- **構想は人間、肉付けはAI** — 記事の核（主張・構成・視点）はオーナーが書く。AIは補強・執筆補助・品質管理を担う
- 目的は「どういう思考をする人間か」を見せること。役に立つ記事を書くのが目的ではない
- 一次情報（体験・実測・判断）がないネタは記事化しない
- 密度 > 文字数。短く濃い記事を作る。量より質
- OK/NGパターンを蓄積し、繰り返し精度を上げる学習サイクルを回す
- NOTE_CONCEPT.md でアカウント全体のコンセプト・発信軸を定義。各記事はこの文脈の中に位置づける

**記事の4レイヤー：**
| # | レイヤー | 内容 | 担当 |
|---|---------|------|------|
| 1 | ベース | 誰が何を発信しているか。記事の文脈 | NOTE_CONCEPT.md → writer/voice_checker |
| 2 | 主張 | 自分の考え・判断・発見 | オーナーの構想 → writer |
| 3 | 裏付け | 世の中の状況、データ、エビデンス | enricher → writer |
| 4 | 面白さ | 切り口の意外性、構成の妙 | editor（Pass 3） |

**最終形態：** このワークスペースで執筆→推敲した記事を、オーナーが手直しして note.com に投稿

## オーナー

knowledge-base/ 内のパーソナリティ情報ファイルに定義されている。作業開始時にまずパーソナリティ情報を読み、オーナーの人物像・価値観・思考構造を把握すること。

<!--
設計メモ: なぜオーナー情報を読ませるか
→ AIが「誰の声で書くか」を把握するため。パーソナリティ情報なしだと汎用的なAI文章になる。
   ここに置くのは：価値観、思考パターン、経歴、興味関心、判断基準など。
   具体的な書き方は voice_profile_guide.md を参照。
-->

## ディレクトリ構成

```
note-pipeline/
├── CLAUDE.md                  … このファイル
├── NOTE_CONCEPT.md            … noteアカウントのコンセプト定義（発信の軸・読者像）
├── STYLE_GUIDE.md             … 文体・ルール
├── voice_profile.md           … オーナーの発言パターン分析
├── .claude/
│   ├── agents/
│   │   ├── enricher.md        … 記憶検証・ナレッジ紐付け・リサーチ
│   │   ├── writer.md          … 構想ベースで執筆
│   │   ├── voice_checker.md   … 人格チェック・書き直し
│   │   ├── editor.md          … 推敲・最終判定
│   │   ├── publisher.md       … 投稿最適化・タイトル考案
│   │   ├── calibrator.md      … OK/NG比較・パターン蓄積
│   │   ├── melchior.md        … MES発散層：逆張り・揺さぶり
│   │   ├── balthasar.md       … MES発散層：素材発掘・接続
│   │   ├── caspar.md          … MES発散層：具現・書き出し
│   │   ├── michael.md         … MES収束層：密度チェック
│   │   ├── sandalphon.md      … MES収束層：声チェック
│   │   ├── metatron.md        … MES収束層：構成チェック
│   │   └── azrael.md          … MES読者層：コンテキストゼロ読者
│   ├── commands/
│   │   └── publish.md         … パイプライン実行コマンド
│   └── settings.json
├── templates/                 … 記事型テンプレ（型1/2/3）
├── output-backup/             … 完成原稿のローカルバックアップ（正本はknowledge-base側）
├── archive/                   … NG例・旧パイプライン成果物
│   └── ng-examples/           … 過去のNG記事（品質改善の参照用）
└── knowledge-base/            … ナレッジベース（Obsidian vault等）
    └── NotePublishing/        … note記事制作の親フォルダ
        ├── Ideas/             … 構想置き場（オーナーが書く。パイプラインの起点）
        │   └── _template.md   … 構想テンプレート
        ├── Output/            … 完成原稿（正本。1記事=1フォルダ）
        │   └── {日付}-{タイトル}/
        │       ├── {日付}-{タイトル}.md
        │       └── {日付}-{タイトル}_meta.md
        ├── Drafts/            … 中間生成物（1記事=1フォルダ）
        │   ├── {日付}-{タイトル}/
        │   │   ├── enriched-{日付}-{タイトル}.md
        │   │   ├── draft-{日付}-{タイトル}.md
        │   │   ├── voice-checked-{日付}-{タイトル}.md
        │   │   ├── voice-report-{日付}-{タイトル}.md
        │   │   └── editor-review-{日付}-{タイトル}.md
        │   └── ng-examples/   … NG記事アーカイブ
        ├── Calibration/       … 学習サイクル用
        │   ├── voice-patterns.md  … AI vs オーナーの乖離パターン集
        │   ├── ok/            … オーナーが手直しした最終版
        │   └── diff/          … AIドラフト vs 最終版の差分記録
        └── pipeline.base      … パイプラインダッシュボード（Obsidian Bases / 任意）
```

## knowledge-base/ の位置づけ

**ナレッジベースへの参照（基本は書き込み禁止）**

このワークスペースから knowledge-base/ を参照する際の役割：
1. **ネタ帳** — ナレッジノートから記事の素材を抽出
2. **パーソナリティDB** — パーソナリティ情報から「オーナーという人間」を再現
3. **参照のみ** — knowledge-base/ への書き込みは行わない（ナレッジベース側で精製された情報のみを読む）

**例外：`knowledge-base/NotePublishing/` への書き込みは許可**
- NotePublishing/ 配下は記事制作の作業領域として全サブフォルダへの書き込みを許可
- Ideas/, Output/, Drafts/, Calibration/ への書き込みはパイプラインの正常動作に必要
- 既存ナレッジへの書き込みは引き続き禁止

**関連ファイルの位置づけ：**
- `NOTE_CONCEPT.md` — 「文脈」（アカウント全体のコンセプト、発信の軸、記事の位置づけ）
- `STYLE_GUIDE.md` — 「ルール」（何を書くか、書き方の枠）
- `voice_profile.md` — 「声の指紋」（どう話すか、発言パターン）
- `voice-patterns.md` — 「学習済みパターン」（AIがやりがちなミスの蓄積）
- `knowledge-base/` — 「人格」（何を思考するか、価値観・経験）

<!--
設計メモ: なぜこの5つを分離しているか
→ 役割ごとに参照するエージェントが違う。
   enricherはknowledge-baseを読むがSTYLE_GUIDEは不要。
   voice_checkerはvoice_profileを読むがknowledge-baseの深掘りは不要。
   各エージェントに必要最小限のコンテキストだけ渡すことで、
   トークン節約とロール分離を実現している。
-->

## パイプライン（基本 + MAGI EVOLVE SYSTEM）

### 基本パイプライン

```
Ideas/（オーナーの構想）
  ↓
enricher（記憶検証・ナレッジ紐付け・リサーチ）
  ↓
writer（構想ベースで執筆）
  ↓
voice_checker（人格チェック・書き直し）
  ↓
editor（推敲・最終判定）
  ↓
publisher（投稿最適化・タイトル考案）
  ↓
Output/（完成原稿）
  ↓
（オーナーが手直し → note.com投稿）
  ↓
calibrator（AIドラフト vs 最終版の比較 → パターン蓄積）
```

### MAGI EVOLVE SYSTEM（発散→収束→読者の3層構造）

基本パイプラインに加えて、発散層・収束層・読者層の3層で記事の質を上げるシステム。

```
構想
  ↓
enricher（補強）
  ↓
┌─────────────────────────┐
│ 発散層（三賢者）           │
│ MELCHIOR: 逆張り・揺さぶり  │
│ BALTHASAR: 素材発掘        │
│ CASPAR: 書き出しサンプル     │
└─────────────────────────┘
  ↓ オーナーが選択・判断
writer（執筆）
  ↓
voice_checker（人格チェック）
  ↓
┌─────────────────────────┐
│ 収束層（天使）             │
│ MICHAEL: 密度チェック       │
│ SANDALPHON: 声の保護       │
│ METATRON: 構成の面白さ     │
└─────────────────────────┘
  ↓ オーナーが判断（🔴🟡⚪️分類）
editor（最終判定）
  ↓
┌─────────────────────────┐
│ 読者層                    │
│ AZRAEL: コンテキストなし読者 │
└─────────────────────────┘
  ↓
publisher（投稿最適化）
```

### エージェント一覧（13体）

**基本パイプライン（6体）:**

| エージェント | モデル | 主な役割 |
|------------|--------|--------|
| enricher | Sonnet | 曖昧な記憶の検証、ナレッジ紐付け、外部リサーチ |
| writer | Opus | 構想ベースで密度を維持した執筆 |
| voice_checker | Opus | レポート化防止、人間味確保、AI臭排除 |
| editor | Opus | 4パス品質チェック、最終判定 |
| publisher | Sonnet | 投稿最適化・タイトル考案・PV向上 |
| calibrator | Sonnet | OK/NG比較、乖離パターン蓄積 |

**MES発散層（3体）:**

| エージェント | モデル | 主な役割 |
|------------|--------|--------|
| MELCHIOR | Opus | 逆張り・揺さぶり。構想の盲点を突く |
| BALTHASAR | Opus | ナレッジベースから意外な素材を発掘 |
| CASPAR | Opus | 冒頭・転換点・締めの書き出しサンプル生成 |

**MES収束層（3体）:**

| エージェント | モデル | 主な役割 |
|------------|--------|--------|
| MICHAEL | Opus | 密度チェック。冗長・水増し・AI臭を斬る |
| SANDALPHON | Opus | 声の保護。voice_profileとの乖離を検出 |
| METATRON | Opus | 構成の面白さ。読者体験をシミュレーション |

**MES読者層（1体）:**

| エージェント | モデル | 主な役割 |
|------------|--------|--------|
| AZRAEL | Opus | ゼロコンテキスト読者レビュー。記事本文だけで感想を返す |

**フィードバックループ：** editor が REVISE 判定した場合、voice_checker に差し戻す（最大2回）

### Obsidian スキル活用（任意）

Obsidianを使っている場合、以下のスキルで精度が上がります:

| スキル | 活用箇所 |
|--------|---------|
| **obsidian-cli** | enricher: `obsidian search` でvault意味検索、`obsidian backlinks` で逆参照 |
| **obsidian-markdown** | writer/editor: Obsidian記法で原稿作成（wikilink, frontmatter） |
| **obsidian-bases** | pipeline.base: 全記事の進捗ダッシュボード |
| **defuddle** | enricher: WebFetchの代わりにクリーンなWeb情報取得（トークン節約） |

### 学習サイクル（calibrator）

1. editor が PUBLISH → Output/ に完成原稿
2. オーナーが手直し → Calibration/ok/ に最終版を保存
3. `calibrator` が差分分析 → voice-patterns.md にパターン蓄積
4. 3回以上出現したパターンは「確定」→ writer/voice_checker が自動回避
5. 月次レビューで STYLE_GUIDE.md に昇格

<!--
設計メモ: なぜ学習サイクルが重要か
→ AIの出力は毎回「同じ種類のズレ」を起こす。
   calibratorがこのズレを記録し、パターン化することで、
   記事を重ねるごとにAIの出力精度が上がる。
   3記事目から明確に効果が出始める。
   これがこのパイプラインの最大の差別化要素。
-->

## 記事の型（3種類）

### 型1: 構造を暴く
具体的な事象を取り上げて、表面から見えない構造を暴く。

### 型2: 実測で語る
自分で測った・やった・作ったデータを見せて、一般論を上書きする。

### 型3: 考えが変わった
ある出来事をきっかけに前提が覆れた話。思考の変化過程が主役。

## 品質保証の仕組み

### 品質チェック体制（基本 + MES）

| エージェント | モデル | 主な役割 | 参照する学習データ |
|------------|--------|---------|-----------------|
| enricher | Sonnet | 曖昧な記憶の検証・エビデンス補強 | — |
| writer | Opus | 構想ベースで密度を維持した執筆 | voice-patterns.md（確定パターン回避） |
| voice_checker | Opus | レポート化防止・人間味確保・AI臭排除 | voice-patterns.md + Calibration/ok/（OK例） |
| MICHAEL | Opus | MES収束層: 密度・冗長・AI臭を斬る | voice-patterns.md + STYLE_GUIDE |
| SANDALPHON | Opus | MES収束層: 声の保護・5軸評価 | voice-patterns.md + Calibration/ok/ |
| METATRON | Opus | MES収束層: 構成・面白さ・テンション | Calibration/ok/ |
| editor | Opus | 事実確認・面白さ・表現校正・最終判定 | voice-patterns.md + ng-examples/（NG例） |
| AZRAEL | Opus | MES読者層: ゼロコンテキスト読者検証 | （一切なし — 記事本文のみ） |
| publisher | Sonnet | 投稿最適化・タイトル考案・装飾・PV向上 | note.comベストプラクティス + 過去記事タグ傾向 |
| calibrator | Sonnet | AIドラフト vs 最終版の差分パターン蓄積 | 全OK/NG蓄積データ |

### voice_checker の5軸
1. **レポート化** — データと論理だけで埋まっていないか（最重要）
2. **人間味** — 自己ツッコミ、留保、感情の短文があるか
3. **敬体ミックス** — 常体一辺倒でないか
4. **AI臭** — 禁止フレーズ、テンプレート構成がないか
5. **一人称・基本** — STYLE_GUIDE.mdで定義した一人称ルール、過剰敬語排除

## セットアップ

```bash
cd ~/Projects/note-pipeline
claude
```

**必要なツール：**
- Obsidian CLI（`obsidian` コマンド）— vault検索・ノート操作に使用（任意）
- defuddle（`npm install -g defuddle`）— Web情報のクリーン取得に使用（任意）

**初回：** `.claude/agents/` 配下のエージェント定義ファイルをすべて確認すること。

## 制約・必須ルール

### knowledge-base/ への書き込みルール
- **基本：書き込み禁止** — ナレッジノートへの書き込みは禁止
- **例外：NotePublishing/ 配下は全て書き込み許可**（Ideas/, Output/, Drafts/, Calibration/）

### 一次情報の必須性
- 体験していない、実測していない、判断していないテーマは記事化しない
- AIが書いた汎用的な解説記事を量産しない
- 「自分だけが言えることは何か」を常に問う

### 投稿フロー
- note.com に公式APIなし
- `knowledge-base/NotePublishing/Output/{記事フォルダ}/` に完成原稿を出力（正本）
- `output-backup/` にローカルバックアップ
- オーナーが手直し後、note.com に手動でコピペ投稿
- 投稿後、手直し版を `Calibration/ok/` に保存し `/publish --calibrate` で学習

## 作業フロー

1. オーナーが Ideas/ に構想を書く（スマホ or Claude.ai）
2. `/publish` でパイプライン起動
3. enricher が記憶検証・ナレッジ紐付け・リサーチ → **オーナーが検証結果を確認**
4. MES発散層（MELCHIOR/BALTHASAR/CASPAR）が並行で提案 → **オーナーが選択・判断**
5. writer が構想 + 選択結果ベースで執筆 → **オーナーが下書きに加筆**
6. voice_checker で人格チェック
7. MES収束層（MICHAEL/SANDALPHON/METATRON）が並行でレビュー → 合議（🔴🟡⚪️分類）→ **オーナーが判断**
8. editor で最終判定
9. AZRAEL がゼロコンテキストで読者レビュー
10. PUBLISH → publisher で投稿最適化（タイトル・装飾・タグ・画像プロンプト）
11. Output/ + output-backup/ に出力。REVISE → voice_checker 差し戻し
12. オーナーが手直し → note.com 投稿
13. `/publish --calibrate` で学習サイクル

## シリーズ現状（2026-04-28時点）

### タイトルフォーマット
- **本編**: `非エンジニアなのに、〇〇 〜意識低い系おじさんがAIと何かを作る話(12)〜`
  - 「非エンジニアなのに、」の後は**反語**（〇〇した/できた/語ってみる等）でないとおかしい
  - 「知らなかった」「できなかった」等の当たり前の内容はNG
- **番外編**: `【番外編(4)】【意識低い系おじさんのAI活用術】○○`
- **特別編**: `【特別編④】【意識低い系おじさんのAI活用術】○○`
- 番号はかっこ書き (1)(2)... で統一（丸数字は機種依存のため廃止。公開済み記事は既存のまま）

### 公開状況

**本編:**
| # | タイトル | 状態 |
|---|---------|------|
| (1)〜(15) | — | 公開済み |
| (16) | 誰も来ないサイトを分析してる | 公開済み |
| (17) | AIと認知戦略を練ってる | 公開済み（2026-04-28、ストック枯渇） |
| (18) | UIで印象が変わると知った（PC/モバイル） | ドラフト・添削待ち |
| (19) | APIの請求額に震えた | ドラフト・添削待ち（クローラーHaiku切替 $100→$5 統合可） |
| (20) | AIにセキュリティチェックを任せた | 構想あり（Supabase RLS / Resend漏洩 統合可） |
| (21) | noteがサービス導線になった | 仮構想 |
| (22) | X運用を一括生成に切り替えた | 仮構想（X API有料→予約投稿） |

**番外編:**
| # | タイトル | 状態 |
|---|---------|------|
| 番外(1)(2)(3)(4)(5) | — | 公開済み |
| 番外(6) | 経由地ポチポチをやめて道は自分で決める（ドライブルートビルダー紹介） | Published-Current済・note下書き済（唯一のストック） |

**特別編:**
| # | タイトル | 状態 |
|---|---------|------|
| ①②③④⑤ | — | 公開済み |

### シーズン構造
- **シーズン1** (1)〜(6): 概念編（AI使い分け、考え方、壁打ち）— 公開済み
- **シーズン2** (7)〜(15): 実装編（土台→メイン機能→地図完成）— 公開済み
- **シーズン3** (16)〜: 届ける編（公開後の認知・集客・運用・セキュリティ）
- **番外編**: (1)(2)(3)(4)(5) 公開済み、(4) 用語集はシーズン2→3 の橋渡し位置、(5) 緊急対応で不定位置、(6) ドライブルートビルダー紹介（本編1の派生サービス）
- **特別編**: ①②③④⑤ 公開済み、今後追加なし（本編2に集約方針）

### 公開待ち直近キュー
```
番外(6)（note下書き済、唯一のストック） → 本編⑱⑲（ドラフト添削待ち） → ⑳…
```

### 構成案（仮番）
`NotePublishing/Ideas/series-plan.md` に詳細あり。(18)以降は仮番で、途中分割あり得る。最終回は決めない。

### 新シリーズ「AIで第二の脳を作った話」（未着手）
- **塊1（メインシリーズ）**: 8〜10章 / 40,000〜70,000字 / 週1-2本ペース想定
  - 候補初稿: 「なぜ第二の脳を作ろうと思ったか」 or 「音声メモパイプライン」
- **塊2（外出し小ネタ6本、独立単発）**: 推奨順 B→F→D→E→A→C
  - B 掟 / F 音声メモ / D 引越し大工事 / E 設計シンプル運用複雑 / A 見落とし / C ダッシュボード
- 詳細は `knowledge-base/未分類MD/2026-04-23-note-series-reorganized.md`

### 分割履歴
- (16): 旧版「公開したのに誰も来ない」を**前半（技術対応）/後半（戦略・命名）** に分割
  - (16) = Analytics/sitemap/OGP（技術的にやれることをやった編）
  - (17) = サイト名/3つのAIに認知戦略/コールドスタート（戦略・認知編）
- 旧(17) PC/モバイル → **(18)** にリナンバー
- 旧(18) APIコスト → **(19)** にリナンバー
- 番外(4) は **(15) と (16) の間** に挿入予定（シーズン2→3の橋渡し、地図で技術的完成の区切り）
- 番外(5) 2026-04-24 緊急追加（ロードスターMTG終了Yahoo!ニュース契機、event-map側LP導線停止と連動）
- 公開ペース: 毎日公開中（2026-04-28時点、本日本編⑰公開でストック枯渇、明日番外(6)で繋ぐ想定）

---

## セッション引き継ぎ

セッション引き継ぎの運用は **両リポ共通仕様書** `_handoffs/README.md` を参照すること:

```
~/Library/Mobile Documents/iCloud~md~obsidian/Documents/ObsidianVault/24offmap/_handoffs/README.md
```

共通仕様書に記載されてる内容:
- 保存場所・命名規則・frontmatter スキーマ
- テンプレート骨格（共通セクション）
- トリガー文言（発火条件）・曖昧時の確認フロー
- 開始時の読込手順（自リポ → 他リポ → 差分報告）
- 鮮度チェック（24h / 7日基準）
- フォールバック（エラーで止まらず続行）
- iCloud 同期遅延対策

以下は **magi 側固有の運用**。共通仕様と併せて参照。

### ファイル命名（magi 側）

```
handoff-magi-YYYY-MM-DD-HHMM-{descriptor}.md
```

保存先は共通仕様書記載の `_handoffs/` 直下。

`{descriptor}` は共通辞書（`session-end` / `wip` / `pipeline-complete` / `article-published` / `draft-complete` / `review-pending` 等）から選ぶ。辞書にないケースは主題を3-5語・英小文字ハイフン区切りで自由命名。

### magi 固有セクション（handoff MD の「各リポ固有セクション」に含める）

共通セクション（完了した作業 / 現在の状態 / 次にやるべきこと / 注意事項・学び / 重要ファイル・パス）の後に、以下の magi 固有項目を含める:

#### 1. 必須ファイル整合性チェック
- `NOTE_CONCEPT.md` — シーズン構造・番号・公開状況が最新か
- `STYLE_GUIDE.md` — 存在するか
- `voice_profile.md` — 存在するか。セッション中に発見した声の特徴があれば追記
- `voice-patterns.md` — セッション中に発見したパターンが追記されているか
- `series-plan.md` — セッション中の構成変更が反映されているか
- `knowledge-base/` — シンボリックリンクが有効か

#### 2. シリーズ状態
本編 / 番外編 / 特別編 それぞれについて、各記事の状態を把握:
- 公開済
- 下書き登録済（note.com 側）
- 添削中
- 構成案中
- 未着手

#### 3. voice-patterns.md の学習サイクル更新状況
- セッション中に新パターン検出されたか
- Calibrator 実行したか
- P-XXX パターンの「観察 → 傾向 → 確定」昇格があったか

#### 4. パイプライン通過状態
並行進行中の記事それぞれについて、以下のどの段階にいるかを明記:
```
未着手 → 構想 → ドラフト → voice_check → editor → publisher → note下書き登録済 → 公開済
```

#### 5. 画像差し込みマーカーの状態
オーナー手動差し込み待ちリスト（例: MLM比較ピラミッド図、情報商材定義スクショ等）。

### 開始時の追加チェック（magi 固有）

`_handoffs/README.md` 記載の共通読込手順に加えて:

1. **パーソナリティ情報の読込** — `knowledge-base/` のパーソナリティ関連ファイルを読んでオーナー人物像を把握
2. **Published-Current の内容確認** — 最新の公開状況を把握
3. **voice-patterns.md の読込** — 学習済みパターンを把握（P-001〜）
4. **series-plan.md の読込** — 構成案の最新状態を把握

### 終了時の追加作業（magi 固有）

`_handoffs/README.md` 記載の共通フロー（handoff 作成）に加えて:

1. **必須ファイル整合性チェック**（上記「magi 固有セクション 1.」のリスト）→ 欠けていたら handoff 前に作成・修正
2. **未コミット変更のコミット**（記事・Ideas・Drafts・Output/Published-Current の更新分）
3. **リモートにプッシュ**（worktree ブランチ + main マージ）

### 既存 handoff の扱い

- `NotePublishing/Ideas/session-handoff-*.md`（5-6件）は**そのまま残す**
- 理由: 参照している他ファイル（`series-plan.md` 等）からのリンク切れを避けるため
- **新規分のみ** `_handoffs/` に出す
- 一括移動 or OLD 化は後日の別タスク（急がない）

---

## 関連ファイル

- `NOTE_CONCEPT.md` — noteアカウントのコンセプト定義・発信の軸
- `STYLE_GUIDE.md` — 文体・ルール・禁止表現
- `voice_profile.md` — 発言パターン分析
- `voice-patterns.md` — AI vs オーナーの乖離パターン集（Calibration/配下）
- `NotePublishing/Ideas/series-plan.md` — シリーズ構成案・ネタ候補
- `.claude/agents/*.md` — 各エージェントの詳細プロンプト
- `.claude/commands/publish.md` — パイプライン実行コマンド

## Chat側MD自動振り分け連携（2026-04-19）

Chat（claude.ai）側で `note-idea-*.md` として起票されたネタは、ダウンロード後にウォッチャーが自動で `NotePublishing/Ideas/` に流し込む（`ObsidianVault/magi` symlink経由）。

- **出力ルール正本**: `knowledge-base/decisions/decision-2026-04-19-chat-md-output-rules.md`
- **ウォッチャー全量**: `knowledge-base/knowledge/watcher-rules-current.md`
- **symlink**: `ObsidianVault/magi` → `/Users/takeomba/LLMPJ/magi/NotePublishing/`（vault側から張る逆方向）
- この連携により、Chatで「ネタとして起票して」→自動で `Ideas/` に着地する運用が成立
