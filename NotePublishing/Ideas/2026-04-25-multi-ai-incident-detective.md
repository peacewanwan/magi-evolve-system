---
created: 2026-04-25
tags:
  - note-candidate
  - ai-with-human-flaws
  - multi-ai-collaboration
  - real-incident
status: flash
source_session: meta セッション 2026-04-25（ホーム .git 誤生成事件 + 犯人特定）
related:
  - 2026-04-24-ai-amnesia-and-hedging.md
  - note-idea-special-10-ai-also-makes-mistakes.md
---

# AI が起こした事故を、別の AI に追わせて犯人を特定した夜

## 主題

Mac の SSD が突然 4.8GB しか空いてなくて、iCloud（Obsidian Vault）まで evict され始めた事件。**犯人は AI ツールだった**。しかも追跡作業で **Claude の盲点を Codex がカバーして真犯人を特定**した。multi-AI 並走運用の「本当の価値」が、こんな雑な形で出ることがある、という話。

特殊シリーズ 10「AI も普通に思い込みで見落とす」、2026-04-24「AI 健忘症と保身ヘッジ」に続く**人間っぽい AI 失敗パターン集** 第 3 弾。

---

## 骨子

### 起：iCloud が消えてる

「昨日 40GB 空けたのに、今日見たら 4.8GB しかない。Obsidian の Vault が空っぽになってきてる」

ユーザーから半泣きの問い合わせ。前日のディスク掃除（Parallels の ISO 4.5GB を削除、Claude のキャッシュ 100MB を整理）が完全に無に帰してる。

`du -sh ~/*` で一発：**`~/.git` が 63GB**。ホームディレクトリに `.git` がある時点で異常事態。

### 承：64GB の正体は失敗 pack ファイル群

`~/.git/objects/pack/` を覗くと：

```
8.2G   tmp_pack_0qVNID  2026-04-24 16:30  ← 巨大失敗
1.4G   pack-6d278...     2026-04-24 15:59  ← 唯一の正規 pack
917M   tmp_pack_098SOt  2026-04-24 19:48
917M   tmp_pack_0YLz7Q  2026-04-24 20:48
... （24h 以上にわたり 917M tmp_pack が約 1 時間間隔で蓄積）
```

**何かが定期的に `git fetch` を試みて失敗、tmp_pack を残している**。

`~/.git` 削除で 62GB 即時回収、iCloud は disk pressure 解消で自動復旧。応急処置 OK。**でも犯人がわからない**。

### 転：Claude の調査、トレイリングスペースで詰む

私（Claude）が調査開始：

- ✅ launchd で git 系ジョブ：なし
- ✅ ~/scripts/ 内の git 操作：LLMPJ/event 対象、ホームを触らない
- ✅ /Library/LaunchDaemons：git 関連なし
- ✅ ヒストリ grep `grep -E "git (fetch|clone|init|pull) " ~/.bash_history`：**マッチなし**
- 結論：「犯人特定不能、Codex に任せましょう」

### 結：Codex が一発で真犯人を見つけた

Codex の調査結果：

> A. ヒストリ系
> `~/.bash_history` に `git init` → `git remote add origin ...` が残っていた

え、待って。**私さっき grep したけど見つからなかったよ？**

確認したら、私の正規表現が **`git (fetch|clone|init|pull) `**（末尾にスペース）だった。`git init` は引数なしで実行されると末尾スペースなしで履歴に残るので、**マッチしない**。

Codex の正規表現はもっと緩く、`git init` をしっかり捕まえてた。

### Codex はさらに踏み込む

`~/.codex/config.toml` を確認：

```toml
[projects."/Users/takeomba"]
trust_level = "trusted"
```

ホーム直下が **Codex の trusted project として登録されてた**。

タイムラインを照合：

```
2026-04-24 15:58:22  Codex thread 開始
                     cwd=/Users/takeomba/LLMPJ
                     git_origin_url=peacewanwan/car-event-map_pre.git
2026-04-24 15:59     ~/.git/objects/pack/ に最初の pack が出現  ← 1 分後！
```

**Codex 自身が犯人**。`/Users/takeomba/LLMPJ` を開いた時、親ディレクトリを遡って `~/.git` を「親 repo」として自動認識 → 内部の Git 自動 refresh 機能が定期 `git fetch` を試行 → 24h 失敗を繰り返して 64GB の tmp_pack を残した。

---

## ここがネタとして美味しい

### 1. AI ツールが起こす「便利機能の副作用」事故

Codex の「親ディレクトリの `.git` を自動認識する」機能は普通に便利機能。でも**ホームに `.git` が誤生成されてた状態**と組み合わさると、ホーム全体を repo として scan + 定期 fetch を仕掛ける罠になる。

「便利機能 × 環境の偶発的状態 = 想定外の事故」は AI に限らないが、**AI ツールはこの組み合わせ爆発が起きやすい**（自動認識・自動補完・自動更新が多いので）。

### 2. AI が AI の盲点を補完する具体例

Claude（私）の調査ミス：trailing space で `git init` を取りこぼした。**自分の正規表現を信じすぎた**典型的な確証バイアス。

Codex はそれを見つけた。同じヒストリファイルを違う角度で grep するだけで、別の AI なら見つけられた。

multi-AI 並走運用の価値ってこういうところ。「片方が間違える時、もう片方は同じ方向には間違えにくい」。**人間でいう「同僚にコードレビューしてもらう」と同じ構造**。

### 3. AI ツールの設定キャッシュは盲点になりやすい

`~/.codex/config.toml` の `trusted_projects` なんて、普段見ない。**GUI アプリの設定キャッシュは「見えない場所」に蓄積される**ので、トラブル時の調査リストに最初から入れておく必要がある。

ユーザーの大半は「AI 用ディレクトリに何が登録されてるか」を知らない。これは将来 AI ツールが増えるほど深刻化する。

---

## 含められる要素

### 事件のナラティブ
- ディスク満杯 → iCloud evict → 焦る → 調査 → AI 同僚に分担 → 解決、の 1 時間半
- 「Mac SSD 228GB は 2026 年の開発機としてはキツい」という現実
- 復旧後の 67GB 空きで「あ、AI のキャッシュゴミだったのか」感

### 技術的気づき
- `~/.git` を作ってしまうと、AI ツールがそれを「親 repo」として勝手に拾う事故
- `GIT_CEILING_DIRECTORIES` という環境変数の存在（ベテランでも知らない人多い）
- sentinel ファイル（`.git` を**ファイル**として置いて `git init` を物理的にブロック）
- iCloud は disk pressure で勝手に evict する（macOS の自動最適化挙動）

### AI 運用の気づき
- AI 同士で調査を分担する効果（同じ盲点を共有しないようツール選択）
- 自分の grep を信じない（regex を緩めに、複数 AI で別観点）
- AI ツールの「設定キャッシュ」は GUI から見えない領域にあり、定期 audit が必要
- multi-AI は「並走させる」だけでこの程度の保険になる

### ライフハック寄り
- ホームに `.git` 作っちゃダメ
- 防御策：sentinel ファイル + `GIT_CEILING_DIRECTORIES`
- Codex を含む AI ツールを workspace に開く時、root はピンポイントに（ホームを開かない）

---

## キーワードメモ

- multi-AI コラボ / Claude × Codex 並走
- AI が AI の盲点を補完
- AI 同僚にレビューしてもらう
- 確証バイアスは AI も人間も
- 便利機能の副作用 / 自動認識の罠
- ディスク逼迫の定期検診（`df -h` 自動通知）
- ホーム直下に git は禁忌
- GIT_CEILING_DIRECTORIES（ベテランも知らない）
- sentinel ファイルというトリック
- 「AI 用ディレクトリに何が登録されてるか」audit の習慣

---

## 着地候補

| シリーズ位置 | タイトル候補 | 性格 |
|---|---|---|
| 「人間っぽい AI 失敗パターン」第 3 弾 | 「AI が起こした事故を、別の AI に追わせて犯人を特定した夜」 | ナラティブ重視、シリーズ感 |
| Tips 系 | 「ホームに `.git` を作ったら 64GB 食われた話と、その防御策」 | 実用系、検索流入狙い |
| AI 運用論 | 「multi-AI 並走の本当の価値は、こういう形で出る」 | 思想寄り、特殊シリーズ向き |
| Quick エッセイ | 「Codex が Claude のミスを 1 分で見つけた話」 | 短め、軽め |

特殊シリーズ 11 として「AI が AI の盲点を補完する話」、シリーズ 12 として「AI ツールの設定キャッシュは盲点になる話」、と分けて書く案もあり。

---

## 副産物（記事化の素材）

このセッション中に作った成果物：

- `meta/decisions/decision-2026-04-25-home-git-incident.md` — postmortem 完全版（時系列 / 犯人 / 対策 / 学び）
- `~/.git` sentinel ファイル（444 permission）
- `~/.zshrc` に `GIT_CEILING_DIRECTORIES=/Users/takeomba`
- `launchctl setenv` で GUI アプリにも適用
- M5 Pro 移行 decision に「Phase 2 で最初に防御策セットアップ」を追記
- Codex 設定 `~/.codex/config.toml` から `/Users/takeomba` trusted project 削除（Codex 自身が実施）

**事件 → 完全 postmortem → 恒久対策実装 → ドキュメント化 → 移行手順への組み込み → note ネタ化**まで 1 セッションで完結している。これも multi-AI 運用の良いケーススタディ。

---

## 連作シリーズとしての位置づけ

```
特殊 10  AI も思い込みで見落とす話         （確証バイアス + アンカリング）
2026-04-24  AI 健忘症と保身ヘッジ            （context window 限界 + 訓練由来癖）
2026-04-25  AI が起こした事故を AI が解決    （便利機能の副作用 + multi-AI 価値）  ← 本記事
```

3 本セットで「**AI と人間の働き方**」シリーズ化の素材としても十分。
