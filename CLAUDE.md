# MAGI EVOLVE SYSTEM — note記事生成パイプライン

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
- **NotePublishing/ のファイルを作成・更新した場合は、必ず Obsidian vault 側にもコピーすること**
  - コピー先：`~/Library/Mobile Documents/iCloud~md~obsidian/Documents/ObsidianVault/24offmap/NotePublishing/` 配下の対応フォルダ
  - オーナーがObsidianから直接確認・振り返りできるようにするため

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

## 関連ファイル

- `NOTE_CONCEPT.md` — noteアカウントのコンセプト定義・発信の軸
- `STYLE_GUIDE.md` — 文体・ルール・禁止表現
- `voice_profile.md` — 発言パターン分析
- `voice-patterns.md` — AI vs オーナーの乖離パターン集（Calibration/配下）
- `.claude/agents/*.md` — 各エージェントの詳細プロンプト
- `.claude/commands/publish.md` — パイプライン実行コマンド

---

## 関連リポジトリ：二輪四輪オフマップ（開発リポジトリ）

note記事のメインテーマであるWebサービスの開発リポジトリ。

- **リポジトリ**：`/Users/takeomba/car-event-map-pre`
- **GitHub**：https://github.com/peacewanwan/car-event-map-pre
- **サービス**：https://24offmap.jp
- **CLAUDE.md**：上記パスのCLAUDE.mdを参照

### 関係性
- オフマップの開発過程・学び・失敗談がnote記事のネタになる
- 開発中に気づいた記事ネタは Ideas/ にメモとして残す
- オフマップのCLAUDE.mdに技術スタック・DB構成・フォルダ構成・デザイン方針等の開発情報あり
- 記事で技術的な内容に触れる場合はオフマップのCLAUDE.mdを参照すること
- セッション中にオフマップ側のファイルを変更した場合、コミットはあちら側のリポジトリで行う
- 作業開始時や状況が不明な場合は、両方のリポジトリのgit logを確認して最新状態を把握すること

### Obsidian Vault（共通ナレッジベース）

両リポジトリ共通の参照先。過去のセッション履歴・チャット履歴・開発ナレッジが全て格納されている。

- **Vault パス**：`~/Library/Mobile Documents/iCloud~md~obsidian/Documents/ObsidianVault/`
- **注意**：macOS FinderからiCloudのObsidianコンテナは直接ブラウズ不可（Terminalからはアクセス可能）

| フォルダ | 内容 |
|---------|------|
| `24offmap/sessions/` | セッションまとめ（session-summary-*.md） |
| `24offmap/chats/` | Claude Export変換済みチャット履歴 |
| `24offmap/knowledge/` | project_knowledge |
| `24offmap/note-articles/` | note記事関連 |
| `24offmap/未分類MD/` | 各種ダウンロードMD（調査結果等） |
| `24offmap/NotePublishing/` | 記事制作の作業領域（Ideas/Drafts/Output） |

状況が不明な場合やオーナーの過去の判断・発言を確認したい場合は、sessions/ や chats/ を参照すること。
