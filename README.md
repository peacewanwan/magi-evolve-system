# MAGI EVOLVE SYSTEM — note記事生成パイプライン

Claude Codeで動く、個人note.com記事の半自動生成システムです。

「構想は人間、肉付けはAI」をコンセプトに、あなたの思考・体験をベースにした記事を、AIが補強・執筆補助・品質管理するパイプラインです。

> **`{OWNER}` について:** CLAUDE.md やエージェント定義に登場する `{OWNER}` はあなた自身の名前に読み替えてください。初回のオンボーディングで自動的に設定されます。

> **設計思想の詳細:** このシステムの設計思想や開発過程の詳細はnote記事で解説しています。 {NOTE_ARTICLE_URL}

---

## このパッケージに含まれるもの

### 全体像（21ファイル / 144KB）

| カテゴリ | ファイル数 | 内容 |
|---------|----------|------|
| **設計書** | 2 | CLAUDE.md（パイプライン全体設計）、README.md（本ファイル） |
| **エージェント定義** | 13 | 基本6体 + MES発散層3体 + MES収束層3体 + MES読者層1体 |
| **コマンド** | 1 | `/publish` — パイプライン実行スラッシュコマンド |
| **テンプレート** | 3 | 構想・STYLE_GUIDE・voice_profile の骨格 |
| **サンプル** | 2 | pipeline-trace・voice-patterns の実例 |

### エージェント一覧と役割

**基本パイプライン（6体）:**

| エージェント | モデル | 役割 |
|------------|--------|------|
| **enricher** | Sonnet | 構想の記憶検証・ナレッジベース紐付け・外部リサーチ |
| **writer** | Opus | 構想ベースで密度を維持した執筆。voice-patternsの自動回避 |
| **voice_checker** | Opus | AI臭排除・レポート化防止・人間味注入。5軸チェック |
| **editor** | Opus | 4パス品質チェック（密度→事実→面白さ→校正）。PUBLISH/REVISE判定 |
| **publisher** | Sonnet | タイトル考案・SEO・ハッシュタグ・画像プロンプト・投稿最適化 |
| **calibrator** | Sonnet | AIドラフト vs 手直し版の差分分析→パターン蓄積→次回自動改善 |

**MAGI EVOLVE SYSTEM — 発散層（3体）:**

| エージェント | 役割 |
|------------|------|
| **MELCHIOR** | 逆張り・揺さぶり。構想の盲点を突く。構成代替案 |
| **BALTHASAR** | ナレッジベースから意外な素材を発掘。★評価で優先順位付け |
| **CASPAR** | 冒頭・失敗描写・転換点・締めの具体的な書き出しサンプルを生成 |

**MAGI EVOLVE SYSTEM — 収束層（3体）:**

| エージェント | モデル | 役割 |
|------------|--------|------|
| **MICHAEL** | Opus | 密度チェック。1文1段落の情報量を測定し、薄い箇所を特定 |
| **SANDALPHON** | Opus | 声の保護。voice_profileとの乖離を検出し、人格の一貫性を守る |
| **METATRON** | Opus | 構成の面白さ。記事全体の読者体験をシミュレーションし、だれを検出 |

**MAGI EVOLVE SYSTEM — 読者層（1体）:**

| エージェント | モデル | 役割 |
|------------|--------|------|
| **AZRAEL** | Opus | ゼロコンテキスト読者レビュー。記事本文だけを読み、率直な感想を返す |

### このパッケージの価値

**1. 情報隔離設計**
各エージェントに「必須/推奨/禁止」のコンテキスト定義がある。全情報を全員に渡す方式と違い、トークン消費を抑えつつ、各エージェントが自分の役割に集中できる。

**2. 学習サイクル**
calibratorが「AIが書いたもの」と「人間が直したもの」の差分をパターン化。3記事で効果が出始め、使い込むほどAIが自分の癖を覚える。

**3. 合議制**
収束層の3体が異なる視点（密度・声・構成）で同時にレビューし、🔴全員一致/🟡多数一致/⚪個別で分類。人間が最終判断する仕組み。

**4. ゼロコンテキスト検証**
AZRAELはパイプラインの文脈を一切持たない。チェックリストで合格した記事でも「読者として面白くない」を検出できる。

**5. 育てて使う設計**
初回起動でClaudeが対話であなたの文体を分析。参考文章を読み込ませるほど初速が上がる。参考文章がなくてもcalibratorが3記事で育てる。使うほど精度が上がる設計。

**6. 拡張性**
エージェントは.mdファイル1つで定義されている。追加・削除・カスタマイズが容易。

---

## 前提条件

| 項目 | 必須/推奨 | 備考 |
|------|---------|------|
| **Claude Code** | 必須 | Pro プラン以上。記事を量産する場合は Max 推奨 |
| **Obsidian** | 推奨（任意） | vault検索に `obsidian` CLI を使う。なくても動く |
| **defuddle** | 推奨（任意） | Web情報のクリーン取得。`npm install -g defuddle` |

### コスト目安

- **Pro プラン（$20/月）で動作します。** Opus を多用しますが、週1-2本ペースなら十分
- **記事を量産する場合（週3本以上）は Max プラン（$100/月）推奨。** レートリミットに余裕が出ます
- 実際のボトルネックはトークン量ではなくレートリミット。発散層・収束層の並行実行時にリクエストが詰まることがある
- 人間の判断待ち時間（15-25分）がレートリミットの自然なバッファになるため、通常運用では問題になりにくい

---

## ディレクトリ構成の作り方

```bash
# 1. ワークスペースを作成
mkdir -p ~/Projects/magi-evolve-system
cd ~/Projects/magi-evolve-system

# 2. パッケージのファイルを配置（このパッケージを展開済みの想定）
# CLAUDE.md          → ルートに配置
# agents/            → .claude/agents/ に配置
# commands/          → .claude/commands/ に配置
# templates/         → templates/ に配置

# 3. ディレクトリ構成
mkdir -p .claude/agents .claude/commands
mkdir -p templates
mkdir -p output-backup
mkdir -p archive/ng-examples
mkdir -p NotePublishing/{Ideas,Output,Drafts,Calibration/{ok,diff}}

# 4. ファイルを配置
cp CLAUDE.md .
cp agents/*.md .claude/agents/
cp commands/publish.md .claude/commands/
cp templates/* templates/

# 5. （任意）Obsidian vault をリンク
# Obsidianを使う場合、vault内にNotePublishingフォルダを置くとダッシュボード管理が便利
# ln -s /path/to/your/obsidian/vault knowledge-base

# 6. settings.json を作成
cat > .claude/settings.json << 'EOF'
{
  "permissions": {
    "allow": [
      "Read",
      "Write",
      "Glob",
      "Grep",
      "Bash(obsidian *)",
      "Bash(defuddle *)",
      "WebSearch",
      "WebFetch"
    ]
  }
}
EOF
```

### 最終的なディレクトリ構成

```
magi-evolve-system/
├── CLAUDE.md                  … パイプライン全体の設計書
├── NOTE_CONCEPT.md            … あなたのnoteアカウントのコンセプト（要作成）
├── STYLE_GUIDE.md             … 文体ルール（テンプレートあり）
├── voice_profile.md           … あなたの発言パターン分析（作り方ガイドあり）
├── .claude/
│   ├── agents/                … 全エージェント定義
│   │   ├── enricher.md
│   │   ├── writer.md
│   │   ├── voice_checker.md
│   │   ├── editor.md
│   │   ├── publisher.md
│   │   ├── calibrator.md
│   │   ├── melchior.md        … MES発散層
│   │   ├── balthasar.md
│   │   ├── caspar.md
│   │   ├── michael.md         … MES収束層
│   │   ├── sandalphon.md
│   │   ├── metatron.md
│   │   └── azrael.md          … MES読者層
│   ├── commands/
│   │   └── publish.md
│   └── settings.json
├── templates/
│   ├── ideas_template.md      … 構想テンプレート
│   ├── style_guide_template.md … STYLE_GUIDEの骨格
│   └── voice_profile_guide.md  … voice_profileの作り方
├── output-backup/             … 完成原稿のローカルバックアップ
├── archive/ng-examples/       … NG記事（品質改善の参照用）
├── knowledge-base/            … あなたのナレッジベース（Obsidian vault等。任意）
└── NotePublishing/            … note記事制作の親フォルダ
    ├── Ideas/                 … 構想置き場（あなたが書く。パイプラインの起点）
    ├── Output/                … 完成原稿（1記事=1フォルダ）
    ├── Drafts/                … 中間生成物
    └── Calibration/           … 学習サイクル用
        ├── voice-patterns.md
        ├── ok/                … 手直し後の最終版
        └── diff/              … AIドラフト vs 最終版の差分記録
```

---

## 初期設定手順

### 一番簡単な方法: Claudeに任せる

ディレクトリ構成を作ったら、Claude Codeを起動して何でもいいので話しかけてください。

```bash
cd ~/Projects/magi-evolve-system
claude
```

```
> 記事を書きたい
```

「こんにちは」でも「始めたい」でも何でもOK。NOTE_CONCEPT.md等が未作成なら、Claudeが先に対話でプロフィールを聞いてきます。答えるだけで必要なファイルが自動生成されます。

後からプロフィールや文体を変えたくなったら、Claudeに「文体を変えたい」「トーンをもっとカジュアルにしたい」等と伝えれば更新してくれます。

### 手動で作る場合

自分でファイルを書きたい場合は以下の手順で。

#### 1. NOTE_CONCEPT.md を作成

あなたのnoteアカウントのコンセプトを定義します。以下の項目を書いてください:

```markdown
# NOTE_CONCEPT

## 誰が書いているか
{あなたの簡単なプロフィール。読者に伝えたい人物像}

## 何を発信するか（発信の軸）
1. {軸1: 例「健康の自己実験」}
2. {軸2: 例「AI活用のリアル」}
3. {軸3}

## 読者像
{どんな人に読んでほしいか}

## このアカウントの一言
{「〇〇な人の思考ログ」のような一言定義}
```

#### 2. STYLE_GUIDE.md を作成

`templates/style_guide_template.md` を元に、あなたの文体ルールを作成してください。最初は薄くてOK。calibratorが学習サイクルで自動的に育てます。

#### 3. voice_profile.md を作成（推奨）

`templates/voice_profile_guide.md` に従い、あなたの発言パターンを分析してください。ChatGPTやClaudeのチャット履歴があると精度が上がります。

### 4. パーソナリティ情報を配置

`knowledge-base/` にあなたのナレッジを配置します。最低限:
- 自分の価値観・考え方をまとめたノート
- 過去の体験・経験のメモ
- 仕事や趣味に関するノート

Obsidianを使っている場合はvaultをリンクするだけでOKです。

---

## 最初の1本を回す手順

### Step 1: 構想を書く

`NotePublishing/Ideas/` に構想ファイルを作成:

```markdown
---
title: "テスト記事"
type: 2
date: 2026-04-10
status: idea
tags:
  - note-idea
---

# テスト記事

## 何を言いたいか（主張）
- {あなたの主張}

## なぜそう思うか（体験・実測・判断）
- {体験ベースの根拠}

## 型
2（実測で語る）

## メモ
{思いついたこと}
```

### Step 2: パイプライン起動

```bash
cd ~/Projects/magi-evolve-system
claude

# Claude Code 内で:
/publish
```

### Step 3: 各ステージで確認

パイプラインは以下の順序で進みます。各ステージでユーザー確認ポイントがあります:

1. **enricher** → 記憶の検証結果を確認。間違いがあれば修正
2. **writer** → 下書きに加筆。足りない視点や具体例を追加
3. **voice_checker** → 修正箇所のbefore/afterを確認
4. **editor** → 最終判定。PUBLISH or REVISE
5. **publisher** → タイトル候補を確認。装飾・タグは自動

### Step 4: 手直し → 投稿 → 学習

Output/ の完成原稿を手直しして note.com にコピペ投稿。手直し後の最終版を Calibration/ok/ に保存して:

```
/publish --calibrate {記事フォルダ名}
```

AIドラフトと手直し版の差分を分析し、voice-patterns.md にパターンを蓄積。次回から同じ修正を繰り返さなくなります。

### 成長の目安

このシステムは使うほど精度が上がります:

| 記事数 | 状態 |
|--------|------|
| **1本目** | パイプラインはあなたのことを知らない。参考文章を読み込ませていれば多少は寄る |
| **2本目** | パターンが見え始める。まだ弱い |
| **3本目** | 確定パターンが出始める。writerとvoice_checkerが自動回避するようになる |
| **5本目〜** | パイプラインが「自分のもの」になってくる |

**箱は最初から完成品。中身は使いながら育てるもの。** 参考文章をたくさん読ませるほど初速が速くなりますが、最終的にはcalibratorの学習サイクルが最も効きます。

---

## MES（MAGI EVOLVE SYSTEM）を使う場合

通常のパイプラインに加えて、発散→収束→読者の3層構造で記事の質を上げるシステムです。

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
  ↓ あなたが選択・判断
writer（執筆）
  ↓
┌─────────────────────────┐
│ 収束層（天使）             │
│ MICHAEL: 密度チェック       │
│ SANDALPHON: 声の保護       │
│ METATRON: 構成の面白さ     │
└─────────────────────────┘
  ↓
┌─────────────────────────┐
│ 読者層                    │
│ AZRAEL: コンテキストなし読者 │
└─────────────────────────┘
  ↓
publisher（投稿最適化）
```

**使いどころ:** 切り口に迷うとき、構想の角度を広げたいとき、品質を最大限上げたいとき。

---

## トラブルシューティング

### Q: voice_profile.md がないと動かない？
A: なくても動きます。ただし「あなたの声」の再現精度が大幅に下がります。最低限のメモでもあると違います。

### Q: Obsidianがないと動かない？
A: `obsidian` CLI の代わりに Grep/Glob でフォールバックします。知識ベースの検索精度は落ちますが動作します。

### Q: calibratorは何記事目から効果が出る？
A: 3記事目から。3回以上出現したパターンが「確定」に昇格し、writer/voice_checker が自動回避するようになります。

### Q: コストを下げるには？
A: Pro プランで十分動作します。レートリミットが気になる場合は、writer/voice_checker/editor のモデルを Sonnet に変更できます（CLAUDE.md内で指定）。品質は下がりますが、レートリミットに余裕が出ます。

---

## ライセンス

MIT License. 詳細は [LICENSE](./LICENSE) を参照。
