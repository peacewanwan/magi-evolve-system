# /publish — note記事生成パイプライン

## 概要
{OWNER}の構想ファイル（Ideas/）を起点に、note記事を生成する4ステージパイプラインを実行する。

## 方針
- **構想は人間、肉付けはAI** — 記事の核（主張・構成・視点）は{OWNER}が書く。AIは補強・執筆補助・品質管理を担う
- **量より質** — 1本の記事に時間をかける
- **学習サイクル** — OK/NGパターンを蓄積し、繰り返し精度を上げる

## 使い方
```
/publish                              # Ideas/ から構想を選んで全工程実行
/publish --from {file}                # 指定した構想ファイルから開始
/publish --stage {stage} --id {name}  # 特定ステージから再開
/publish --calibrate {記事フォルダ名}   # 投稿後の学習（calibrator起動）
```

## パイプラインフロー

```
Ideas/（{OWNER}の構想）
  ↓
Stage 1: enricher — 記憶検証・ナレッジ紐付け・リサーチ
  ↓
Stage 2: writer — 構想ベースで執筆
  ↓
Stage 3: voice_checker — 密度・AI臭・声チェック
  ↓
Stage 4: editor — 最終判定（PUBLISH / REVISE）
  ↓
Stage 5: publisher — 投稿最適化・タイトル考案・PV向上
  ↓
Output/（完成原稿）
  ↓
（{OWNER}が手直し → note.com投稿）
  ↓
calibrator — AIドラフト vs 最終版の比較 → パターン蓄積
```

## 各ステージ詳細

### Stage 0: 構想選択（ユーザー操作）

Ideas/ の構想ファイルを一覧表示し、{OWNER}に選んでもらう。

```bash
# Obsidian CLIで Ideas/ のノート一覧を取得（Obsidian使用時）
obsidian vault="{your-vault}" search query="path:NotePublishing/Ideas" limit=20
```

フォールバック: `ls NotePublishing/Ideas/`

**ユーザー確認ポイント**: 構想を選択。

### Stage 1: enricher（補強）
- エージェント: enricher を起動（Sonnet）
- 構想ファイルの曖昧な記憶を検証（最重要）
- ナレッジ紐付け・外部エビデンス収集
- 独自性チェック
- 出力: `NotePublishing/Drafts/{日付}-{タイトル}/enriched-{日付}-{タイトル}.md`
- **ユーザー確認ポイント**: 記憶の検証結果を確認

### Stage 2: writer（執筆）
- エージェント: writer を起動（Opus）
- 構想ファイル + enriched レポートをベースに執筆
- voice-patterns.md の確定パターンを事前回避
- 出力: `NotePublishing/Drafts/{日付}-{タイトル}/draft-{日付}-{タイトル}.md`
- **ユーザー確認ポイント**: `draft-*_edit.md` を作成し{OWNER}に渡す。加筆修正後、以降のステージは `_edit.md` を入力にする

### Stage 3: voice_checker（人格チェック）
- エージェント: voice_checker を起動（Opus）
- voice-patterns.md の確定パターンを必ず検出・修正
- 5軸評価
- 出力: voice-checked + voice-report
- **ユーザー確認ポイント**: 修正箇所のbefore/afterを表示

### Stage 4: editor（推敲・最終判定）
- エージェント: editor を起動（Opus）
- 4パス品質チェック
- 判定:
  - **PUBLISH** → Output/ + output-backup/ に出力
  - **MINOR_FIX** → editor修正後にOutput/
  - **REVISE** → voice_checkerに差し戻し（最大2回）

### Stage 5: publisher（投稿最適化・タイトル考案）
- エージェント: publisher を起動（Sonnet）
- **文章の本質は一切変えない**
- タイトル考案、Markdown互換変換、可読性最適化、SEO、タグ、画像プロンプト
- 出力: Output/ の原稿・_meta.mdを上書き更新
- **ユーザー確認ポイント**: タイトル候補を確認

### 完了時
- 完成原稿のパスを表示
- note投稿用情報を表示
- **pipeline-trace を表示**

### Post-publish: calibrator（学習）
```
/publish --calibrate {記事フォルダ名}
```
- AIドラフト vs 最終版の差分を分析
- 乖離パターンを voice-patterns.md に蓄積
- 3回以上出現で「確定」

## パイプライントレース

記事ごとに pipeline-trace を生成。各エージェントがステージ完了時に追記。

**目的:** 最終原稿だけでは「どのエージェントが品質ボトルネックか」がわからない。トレースで可視化。

## 中断・再開
```
/publish --stage voice_checker --id 20260406-記事タイトル
/publish --stage editor --id 20260406-記事タイトル
```

## Ideas/ 構想ファイルの書き方

テンプレート: `NotePublishing/Ideas/_template.md`

最低限必要なもの:
- 何を言いたいか（主張）
- なぜそう思うか（体験・実測・判断）
- どの型か（1: 構造を暴く / 2: 実測で語る / 3: 考えが変わった）
