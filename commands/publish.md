# /publish — note記事生成パイプライン（MAGI EVOLVE SYSTEM）

## 概要
{OWNER}の構想ファイル（Ideas/）を起点に、13体のエージェントでnote記事を生成する。

## 方針
- **構想は人間、肉付けはAI** — 記事の核は{OWNER}が書く。AIは補強・執筆補助・品質管理
- **選ぶことで魂が入る** — {OWNER}の判断ポイントが複数箇所。選択と加筆で記事に声が浸透する
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
Stage 0: 構想選択（{OWNER}）
  ↓
Stage 1: enricher — 記憶検証・ナレッジ紐付け・リサーチ
  ↓
Stage 2: 発散層（三賢者 — 3体並行）
  MELCHIOR（逆張り）+ BALTHASAR（素材発掘）+ CASPAR（書き出しサンプル）
  ↓
  {OWNER}が素材/角度を選択・判断                    ← 判断①
  ↓
Stage 3: writer — 構想 + 選択結果ベースで執筆
  ↓
  {OWNER}が加筆修正                                ← 判断②
  ↓
Stage 4: voice_checker — 人格チェック・5軸評価
  ↓
Stage 5: 収束層（天使 — 3体並行）
  MICHAEL（密度）+ SANDALPHON（声）+ METATRON（構成）
  → 合議（🔴全員一致/🟡多数一致/⚪️個別）
  ↓
  {OWNER}が判断（PUBLISH / REVISE）                ← 判断③
  ↓
Stage 6: editor — 最終判定
  ↓
Stage 7: AZRAEL — ゼロコンテキスト読者レビュー
  ↓
Stage 8: publisher — 投稿最適化・タイトル考案
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
- 出力: `NotePublishing/Drafts/{id}/enriched-{id}.md`
- **ユーザー確認ポイント**: 記憶の検証結果を確認

### Stage 2: 発散層（三賢者 — 3体並行）

3体を同時に起動し、構想を多角的に揺さぶる:

- **MELCHIOR（逆張り）** — 構想に対して建設的な反論と代替案。揺さぶり3案 + 構成代替案
- **BALTHASAR（素材発掘）** — ナレッジベースから「意外だけど言われてみれば繋がる」素材を発掘。★ランク付け
- **CASPAR（書き出しサンプル）** — 主要ブロック2-3パターンのサンプル文章。冒頭と締めは必ず複数候補

**ユーザー確認ポイント（判断①）**: 3体の出力を見て、使う素材・角度・サンプルを選択。コメントをつける。

### Stage 3: writer（執筆）
- エージェント: writer を起動（Opus）
- 構想 + enriched + 発散層の選択結果をベースに執筆
- voice-patterns.md の確定パターンを事前回避
- 出力: `NotePublishing/Drafts/{id}/draft-{id}.md`
- **ユーザー確認ポイント（判断②）**: `draft-{id}_edit.md` を作成し{OWNER}に渡す。加筆修正後、以降のステージは `_edit.md` を入力にする

### Stage 4: voice_checker（人格チェック）
- エージェント: voice_checker を起動（Opus）
- voice-patterns.md の確定パターンを必ず検出・修正
- 5軸評価（レポート化・人間味・敬体ミックス・AI臭・一人称）
- 出力: voice-checked + voice-report
- **ユーザー確認ポイント**: 修正箇所のbefore/afterを表示

### Stage 5: 収束層（天使 — 3体並行）

3体を同時に起動し、異なる軸で品質評価:

- **MICHAEL（密度）** — 冗長・水増し・AI臭を斬る。構想は読まない（バイアス排除）
- **SANDALPHON（声）** — 声の保護。5軸評価。メタナラティブ・思考の跳躍は絶対保護
- **METATRON（構成）** — 構成の面白さ。ドラフトのみ読む（構想もSTYLE_GUIDEも読まない）

3体の指摘を合議で分類:
- 🔴 全員一致 → 自動反映
- 🟡 多数一致 → {OWNER}が選択
- ⚪️ 個別意見 → 参考情報

**ユーザー確認ポイント（判断③）**: 合議結果から判断。PUBLISH / REVISE

### Stage 6: editor（推敲・最終判定）
- エージェント: editor を起動（Opus）
- 4パス品質チェック（密度→事実→面白さ→校正）
- 判定:
  - **PUBLISH** → 次ステージへ
  - **MINOR_FIX** → editor修正後に次ステージへ
  - **REVISE** → voice_checkerに差し戻し（最大2回）

### Stage 7: AZRAEL（読者レビュー）
- ゼロコンテキストで記事を読者として読む
- **記事本文のみ**を渡す。他のファイルは一切渡さない
- チェックリストなし。スコアリングなし。読んだ感想だけ返す
- 出力: 読者フィードバック（第一印象、つまらない箇所、好きな箇所、一言）
- 結果を{OWNER}に見せる

### Stage 8: publisher（投稿最適化・タイトル考案）
- エージェント: publisher を起動（Sonnet）
- **文章の本質は一切変えない**
- タイトル考案、Markdown互換変換、可読性最適化、SEO、タグ、画像プロンプト
- 出力: Output/ の原稿・_meta.mdを生成
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
- 3回以上出現で「確定」→ 次回からwriter/voice_checker/収束層が自動回避

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
