# ドキュメント矛盾・重複監査レポート（2026-05-15）

## 概要

- 監査対象: 32 ファイル（ルール 5 / メモリ 11 / handoff 4 / agent 13）
  - ルール: CLAUDE.md, STYLE_GUIDE.md, voice_profile.md, NOTE_CONCEPT.md, voice-patterns.md
  - メモリ: MEMORY.md + feedback 10 件
  - handoff: README.md, handoff-magi-*, handoff-eventmap-*, handoff-meta-*
  - agent: enricher, writer, voice_checker, editor, publisher, calibrator, melchior, balthasar, caspar, michael, sandalphon, metatron, azrael
- 抽出件数: 矛盾 **6** / 重複 **9** / 散在 **11** / 場所違い **5** / 死蔵 **7**（合計 **38** 項目）
- 内訳: Critical **9** / Medium **17** / Low **12**

---

## Critical（agent 動作に直接影響）

### C-01: 体言止め推奨 vs 動詞型体言止め禁止（直接矛盾）

- **種別**: 矛盾
- **箇所 A**: `STYLE_GUIDE.md` L14 — 「体言止め・名詞止めを積極的に使う」
- **箇所 B**: `voice-patterns.md` P-005（L72-77）— 「体言止めの多用」「動詞型体言止め『〜書く』『〜する』『〜作る』も同様に AI 臭を強める」「1段落1-2箇所まで」
- **影響**: writer / voice_checker / MICHAEL は STYLE_GUIDE と voice-patterns を両方参照する。STYLE_GUIDE が「積極的に」と書いている以上、writer は体言止めを多用する方向に寄る。voice_checker / MICHAEL が後段で削る、という非効率な往復が発生。実際に「記憶のリレー」で writer が 4 連発を出し、SANDALPHON も誤判定、MICHAEL でようやく検出（handoff L249-253）
- **推奨対応**: STYLE_GUIDE L14 を「体言止め・名詞止めは1段落1-2箇所まで（リズム狙いの限定使用）。動詞型体言止め（〜書く / 〜する）の連発は禁止」に書き換え。P-005 を SSoT として、STYLE_GUIDE がそれを参照する形に統一

### C-02: H1 = frontmatter title 一致ルール vs オーナー H1 編集保護の優先順位（潜在矛盾）

- **種別**: 矛盾
- **箇所 A**: `STYLE_GUIDE.md` L66-68 — 「**H1 = frontmatter title と一致**」「短縮形での H1 は禁止」
- **箇所 B**: `voice-patterns.md` P-012（L117-136）— 「オーナーの draft H1 編集保護」「H1 を writer 出力に戻すのは禁止」
- **箇所 C**: `feedback_h1_owner_edit_protection.md` 全文 — 「frontmatter title 側を H1 に合わせる」「逆方向は禁止」
- **箇所 D**: `feedback_h1_owner_edit_protection.md` L18-22 補足 — 「title を H1 に合わせる作業は厳密には不要（メタ整合性のため揃えてもよいが、必須ではない）」
- **影響**: voice_checker / editor が STYLE_GUIDE を読むと「H1 = title 一致」を機械適用してオーナー編集を破壊する（章2 で実際に発生）。voice_checker.md / editor.md の保護ルールセクションに P-012 が組み込まれていないため、STYLE_GUIDE の機械適用が優先される
- **推奨対応**: STYLE_GUIDE L66-68 に「ただし、オーナーが draft で H1 を書き換えた場合は frontmatter title を H1 に合わせる（P-012 / feedback_h1_owner_edit_protection）」の例外節を必ず追加。voice_checker.md「保護ルール」に H1 を追加（writer.md には説明型 H1 禁止のみあるが、保護ルールがない）

### C-03: H1 一致は必要 vs 不要（feedback 内部での揺れ）

- **種別**: 矛盾
- **箇所 A**: `STYLE_GUIDE.md` L66 — 「H1 = frontmatter title と一致（プレフィックス込みの全タイトル）」（強い必須トーン）
- **箇所 B**: `feedback_h1_owner_edit_protection.md` L18-22 — 「title を H1 に合わせる作業は厳密には不要…必須ではない」「title が古いまま放置されていても note 公開上の問題なし — 修正必要と早合点しない」
- **影響**: STYLE_GUIDE が「一致が必須」と言い、feedback が「不要」と言う。voice_checker / editor / publisher が title の古さを「修正対象」と扱うか「放置でいい」と扱うかが揺れる
- **推奨対応**: STYLE_GUIDE 側を「H1 と frontmatter title はメタ整合性のため揃える運用（必須ではない）」に緩和。または feedback 側で「メタ整合性のため揃える方が望ましい」に修正してトーンを揃える

### C-04: enricher.md に未実装ツール `obsidian search` / `obsidian backlinks` が記述（aspirational 違反）

- **種別**: 死蔵 + 場所違い（CLAUDE.md 制約違反）
- **箇所 A**: `enricher.md` L19-22 — `obsidian vault="..." search query="..."` / `obsidian backlinks file="..."` のコマンド例
- **箇所 B**: `enricher.md` L95-97 — 「検索手順 1. `obsidian search` で…キーワード検索 / 2. `obsidian backlinks` で逆参照取得」
- **箇所 C**: `CLAUDE.md` L?? 「制約・必須ルール / aspirational を書かない」セクション — 「将来こうしたい・こうなってると便利は明示ラベル付きで隔離」「過去の事故例: obsidian-cli の `obsidian search` / `obsidian backlinks` が CLAUDE.md と enricher.md に書かれていたが**実装ゼロ**だった（2026-05-08 発覚、Grep/Glob ベースに書き換え済）」
- **影響**: CLAUDE.md は「Grep/Glob ベースに書き換え済」と宣言しているが、enricher.md には書き換え漏れが残っている。enricher が起動するたびに存在しないコマンドを試して fallback、または書かれた手順に従って Grep/Glob を後手に回す
- **推奨対応**: enricher.md L19-22 のコマンド例を削除、または「未実装・将来候補」ラベル付きで隔離。L95-97 を「1. Grep/Glob で knowledge-base/ を構想キーワードで検索 / 2. 関連ノートから wikilink を辿る」に置き換え

### C-05: 「章N 呼称排除」が writer.md / voice_checker.md / editor.md prompt に展開されていない

- **種別**: 死蔵
- **箇所 A**: `feedback_no_chapter_number_in_conversation.md` 全文 — 「本文・会話・todo・指示・agent prompt の全ての場面で使わない」「agent prompt（writer / 再構成 / voice_checker など）にも『本文中の章N 露出禁止』を明示」
- **箇所 B**: `handoff-magi-2026-05-13-0446-...md` L237-247（注意事項・学び 2）— 「本文中の『章N』参照は…置換するルール。会話・todo・指示でも徹底」
- **箇所 C**: `writer.md` — 章N 呼称への言及なし
- **箇所 D**: `voice_checker.md` — 章N 呼称への言及なし
- **箇所 E**: `editor.md` — 章N 呼称への言及なし
- **箇所 F**: `publisher.md` — 章N 呼称への言及なし
- **影響**: feedback で「agent prompt に明示」と書かれているのに、4 つの主要 agent prompt に展開されていない。writer は章N をそのまま本文に書き、後段が拾えなければ公開時にオーナーが手作業で置換する負荷を背負う
- **推奨対応**: writer.md / voice_checker.md / editor.md / publisher.md 各々の「やってはいけないこと」または「チェック項目」に「本文・H2 中の章N 呼称（章1/章2/…）露出禁止、『前回』『今回』『以前『〜』を書いたとき』に置換」を明記。voice-patterns に「章N 呼称検出」を P-XXX として記録（現状は memory feedback のみ）

### C-06: 「Claude Code を Code と略すな」が agent prompt に展開されていない

- **種別**: 死蔵
- **箇所 A**: `feedback_no_claude_code_abbreviation.md` L11-15 — 「writer / voice_checker / editor / publisher 起動時の prompt に絶対指示として明示」
- **箇所 B**: `voice-patterns.md` P-019（L225-234）— 「voice_checker・editor・publisher のどこかで P-019 を必ずスキャンする工程が必要」「publisher テンプレートで自動置換ステップを入れるべき」
- **箇所 C**: `writer.md` / `voice_checker.md` / `editor.md` / `publisher.md` — 言及なし
- **影響**: 章4 公開版で AI 出力に 10+ 箇所「Code」単独が残った事実（voice-patterns L227）。feedback も voice-patterns もこの問題を指摘済だが、agent prompt 側に組み込まれていない
- **推奨対応**: writer.md / voice_checker.md / editor.md / publisher.md に「Claude Code を必ず半角スペース入りで表記、『Code』『ClaudeCode』禁止」を明示。publisher.md の Step に「Claude Code 表記の自動スキャン・置換」を追加

### C-07: 「具体的な日付・時刻・午前午後表現禁止」が agent prompt に展開されていない

- **種別**: 死蔵
- **箇所 A**: `feedback_no_concrete_datetime.md` L13-14, 28-31 — 「writer / voice_checker / editor / publisher で上記が本文に残っていたら検出・置換対象」
- **箇所 B**: `voice-patterns.md` P-018（L214-223）— 「writer / voice_checker / editor / publisher で…検出 → 抽象表現への置換を提案」
- **箇所 C**: `writer.md` / `voice_checker.md` / `editor.md` / `publisher.md` — 言及なし
- **影響**: 章4 draft で複数発生、オーナー直接指摘で全箇所修正したが、agent prompt 側にルールが組み込まれていない
- **推奨対応**: writer.md / voice_checker.md / editor.md / publisher.md に「本文中の具体日付・時刻・午前午後表現は禁止、抽象進行表現（『ある日』『しばらくして』等）に置換」を明示

### C-08: 「章タイトル・段落タイトルを説明くさくしない」が部分展開のみ

- **種別**: 散在 + 部分死蔵
- **箇所 A**: `feedback_no_explanatory_titles.md` L26-28 — 「writer / 再構成 / editor / publisher 起動時の prompt に絶対指示として明示」
- **箇所 B**: `writer.md` L159, L171 — 「H1 / H2 を説明型にする…禁止」「H1 / H2 は引きで読ませる」※ 反映済み
- **箇所 C**: `editor.md` L68 — Pass 4 内に「H1 / H2 の説明くささチェック」※ 反映済み
- **箇所 D**: `publisher.md` L29-34 — タイトル基準に「説明型を避ける」※ 反映済み
- **箇所 E**: `voice_checker.md` — 言及なし（feedback では voice_checker への明示指示はないが、5軸チェック内で見出し評価する場面で参照すべき）
- **箇所 F**: 再構成 agent — 存在しない（writer.md 内に再構成について言及なし、handoff の「再構成 agent 起動」は general-purpose で起動された）
- **影響**: 3 つの agent には組み込まれたが、voice_checker と「再構成」運用 prompt には漏れ。再構成は将来的に頻度が増える運用なので、prompt template 化されていないと毎回見落とす
- **推奨対応**: voice_checker.md 5軸チェックに「見出しの説明くささ」を追加。再構成 prompt template を `.claude/agents/restructurer.md` 等で正式化、または writer.md 内の再構成モードを定義

### C-09: snapshot 出力工程が writer.md のみ、他 draft 生成 agent に展開なし

- **種別**: 散在 + 場所違い
- **箇所 A**: `writer.md` L130-147 — `draft-writer-original-*.md` snapshot 保存ルール、「writer 原文の凍結」「voice_checker / MICHAEL / editor 段階で writer 原文 vs オーナー添削後の差分追跡が可能になる」
- **箇所 B**: `voice_checker.md` — voice-checked-*.md を出力するが、その後オーナーが添削した場合の snapshot 保全ルールなし
- **箇所 C**: `editor.md` / `publisher.md` — それぞれ Output / Output 上書きするが、上書き前 snapshot 保全ルールなし
- **影響**: writer 原文は保全されるが、voice_checker 後にオーナーが手を入れた版が editor で上書きされる可能性。voice-patterns P-012 の「オーナー H1 編集保護」は実質的に voice_checker snapshot がないことに依存している
- **推奨対応**: voice_checker.md / editor.md / publisher.md にも snapshot 工程（`*-voice-checker-original-*.md` 等）を追加するか、writer.md の snapshot だけで十分なら明示。または「オーナー編集の検出は writer snapshot との diff で行う」を共通ルール化

---

## Medium（運用上の混乱）

### M-01: 一人称ルールが voice_profile + STYLE_GUIDE で重複

- **種別**: 重複
- **箇所 A**: `STYLE_GUIDE.md` L8-10 — 「『自分』一択」「『僕』『私』『俺』『筆者』は使わない」
- **箇所 B**: `voice_profile.md` L9-11 — 「デフォルト: 自分」「『僕』『私』『俺』は使わない」
- **箇所 C**: `voice_checker.md` L77, L80 — 「STYLE_GUIDE.md で定義した一人称ルールに従っているか」（SSoT 参照）
- **影響**: voice_profile を読む agent（writer / SANDALPHON / CASPAR）と STYLE_GUIDE を読む agent（writer / voice_checker / MICHAEL / publisher）が二重ソース参照。片方を更新したら片方も同期する手間
- **推奨対応**: STYLE_GUIDE を SSoT として一人称ルールを集約。voice_profile L9-11 を「一人称は『自分』（詳細は STYLE_GUIDE.md）」の 1 行参照に圧縮

### M-02: 文体（常体/敬体）ルールが重複

- **種別**: 重複
- **箇所 A**: `STYLE_GUIDE.md` L4-6 — 「常体（だ・である調）が基本」「敬体は意図的なミックス」「ですます調で統一するのは禁止」
- **箇所 B**: `voice_profile.md` L14-17 — 「常体ベース、ときどき敬体を混ぜる」「ですます調で統一しない」「短文多め」「カジュアルな語尾」
- **箇所 C**: `NOTE_CONCEPT.md` L62 — 「noteでやらないこと: ですます調」
- **影響**: 3 箇所に分散。calibrator が文体ルールを参照する場合、SSoT が曖昧
- **推奨対応**: STYLE_GUIDE を SSoT に。voice_profile はカジュアル語尾の具体例（〜じゃん / 〜だよね 等）にフォーカスを絞り、文体方針自体は STYLE_GUIDE 参照

### M-03: 短文・1文の長さルールが分散

- **種別**: 散在
- **箇所 A**: `STYLE_GUIDE.md` L13 — 「一文は短く。迷ったら切る」
- **箇所 B**: `voice_profile.md` L16 — 「短文多め。1文=1段落もよく使う」
- **箇所 C**: `publisher.md` L51 — 「1文60字超 → 分割を提案」
- **影響**: 数値ルール（60 字）が publisher だけにあり、writer / voice_checker は持たない。writer が長文を出して publisher で分割が常態化
- **推奨対応**: STYLE_GUIDE L13 に「1文60字超は分割検討」の目安を追加し全 agent で参照可能に

### M-04: 自虐・ユーモア温度ルールが分散

- **種別**: 散在
- **箇所 A**: `STYLE_GUIDE.md` L28-32 — 「控えめに自然に」「おじさん感を消さない」「自虐はあるけど卑屈にはしない」
- **箇所 B**: `voice_profile.md` L38-42 — 自虐 / 事実淡々型 / ツッコミ / メタ注釈の4型
- **箇所 C**: `NOTE_CONCEPT.md` L13 — 「自虐はあるけど卑屈じゃない」
- **影響**: 同じメッセージが 3 箇所。voice_profile L38-42 の 4 型分類だけが具体的
- **推奨対応**: STYLE_GUIDE / NOTE_CONCEPT は短く参照、voice_profile に 4 型を集約

### M-05: 絵文字ルールが 3 箇所重複

- **種別**: 重複
- **箇所 A**: `STYLE_GUIDE.md` L53-54 — 「基本は使わない」「意外性として時々入れるのはあり」
- **箇所 B**: `voice_profile.md` L45 — 「絵文字の多用」（やらないこと）
- **箇所 C**: `NOTE_CONCEPT.md` L65 — 「絵文字の多用（意外性としてたまに使うのはあり）」
- **推奨対応**: STYLE_GUIDE を SSoT。他は削除または参照

### M-06: 禁止フレーズ・AI臭リストが分散

- **種別**: 散在
- **箇所 A**: `STYLE_GUIDE.md` L40-50 — 禁止表現リスト（11 項目）
- **箇所 B**: `voice_profile.md` L45-47 — 「啓発調」「技術ひけらかし」「AI臭い表現」
- **箇所 C**: `voice-patterns.md` P-004〜P-029 — 検出パターン群
- **箇所 D**: `writer.md` L33-37 — 「STYLE_GUIDE.md の禁止フレーズを完全に排除」
- **箇所 E**: `voice_checker.md` L70 — 「禁止フレーズが含まれていないか」
- **箇所 F**: `MICHAEL.md` L73 — 「STYLE_GUIDE.md の禁止フレーズ」
- **影響**: STYLE_GUIDE と voice-patterns の関係が明確でない（STYLE_GUIDE が「形式」、voice-patterns が「学習」と言いたいが、両方を見ないと完全な禁止リストが揃わない）
- **推奨対応**: STYLE_GUIDE に「形式的な禁止フレーズ」、voice-patterns に「観察された乖離パターン」と役割を明示分離。voice-patterns で確定昇格したものは STYLE_GUIDE に移管する運用を徹底（現在は P-008/P-009 のみ移管済）

### M-07: タイトルフォーマット記述が STYLE_GUIDE と CLAUDE.md で重複

- **種別**: 重複
- **箇所 A**: `STYLE_GUIDE.md` L56-68 — タイトルルール詳細（本編1/本編2/番外編/特別編/第二の脳）
- **箇所 B**: `CLAUDE.md` シリーズ現状セクション 「タイトルフォーマット」— ほぼ同内容
- **影響**: 1 箇所更新するともう片方も同期する必要、漏れリスク
- **推奨対応**: STYLE_GUIDE を SSoT、CLAUDE.md は 1 行で参照

### M-08: 文字数ポリシー（オーバー気にしない）が agent prompt に展開不十分

- **種別**: 死蔵 + 散在
- **箇所 A**: `feedback_word_count_policy.md` 全文 — 「writer / voice_checker / editor で文字数オーバーを警告する運用は廃止」
- **箇所 B**: `writer.md` L96 — 「全体の文字数が型の目安に収まっているか」※ 警告チェック残存
- **箇所 C**: `editor.md` L27 — Pass 1 内 「全体の文字数が型の目安に収まっているか」※ 警告チェック残存
- **影響**: feedback で「オーバー警告禁止」と書いた直後の方針が agent prompt に反映されていない。writer / editor が「文字数オーバー警告」を継続発出
- **推奨対応**: writer.md L96 / editor.md L27 から文字数チェック項目を削除、または「水増しがないか」だけに置換

### M-09: H1 = title 一致の運用が voice_checker / editor / publisher で分散

- **種別**: 散在
- **箇所 A**: `STYLE_GUIDE.md` L66-68 — 一致必須
- **箇所 B**: `voice_checker.md` — H1 チェックの明示記述なし（5軸にも含まれず）
- **箇所 C**: `editor.md` Pass 4 — 「記事間一貫性」のみ言及、H1 一致は明示なし
- **箇所 D**: `publisher.md` — タイトル考案のみ、H1 一致言及なし
- **影響**: P-012（オーナー編集保護）と STYLE_GUIDE（一致必須）のどちらを適用するかが、各 agent の解釈次第になる
- **推奨対応**: voice_checker.md「保護ルール」に H1 を追加、editor.md Pass 4 / publisher.md Step 1 に「H1 と frontmatter title の不一致検出時は writer snapshot と比較してオーナー編集を優先（P-012）」を明示

### M-10: NotePublishing/ パス vs knowledge-base/note/ パスの揺れ

- **種別**: 場所違い
- **箇所 A**: 多くの agent prompt（writer.md / voice_checker.md / editor.md / publisher.md / enricher.md / calibrator.md）— `NotePublishing/Ideas/`, `NotePublishing/Drafts/`, `NotePublishing/Output/`, `NotePublishing/Calibration/` 表記
- **箇所 B**: `CLAUDE.md` ディレクトリ構成 — `knowledge-base/note/Ideas/`, `knowledge-base/note/Drafts/`, etc.
- **影響**: 2026-05-08 の Vault 統合で `NotePublishing/` は廃止され `knowledge-base/note/` が正本になっているのに、agent prompt は旧 path のまま。agent が `NotePublishing/` を探して見つからずエラー、または fallback で knowledge-base/note/ を探す
- **推奨対応**: 全 agent prompt の `NotePublishing/` を `knowledge-base/note/` に置換（feedback_verify_path_before_concluding_absent.md が示す通り、path 揺れは事故の元）

### M-11: enricher.md の Obsidian CLI 構文 vs handoff の Grep/Glob 運用

- **種別**: 死蔵
- **箇所 A**: `enricher.md` L19-22, L95-97 — `obsidian search` / `obsidian backlinks` 使用前提
- **箇所 B**: `CLAUDE.md` 補助ツールセクション — 「obsidian-cli ❌ 未導入」「現状 Grep/Glob で代替」
- **影響**: C-04 と関連。enricher が起動するたびに存在しないコマンドを試す
- **推奨対応**: C-04 と統合対応

### M-12: 「写真・看板を象徴化しない」（P-006）が agent prompt に展開なし

- **種別**: 散在
- **箇所 A**: `voice-patterns.md` P-006（L79-86）— 「看板は全然絡めないでいい」
- **箇所 B**: writer.md / voice_checker.md / editor.md — 言及なし
- **影響**: 番外編 P.S. 系で再発の可能性
- **推奨対応**: writer.md「やってはいけないこと」に追加検討（観察パターンのため優先度は下げてよい）

### M-13: Ideas メタ指示の本文化禁止（P-007）が writer.md に展開なし

- **種別**: 死蔵
- **箇所 A**: `voice-patterns.md` P-007（L87-93）— 「Ideas のメタ指示が本文化される」「絶対禁止に追加」
- **箇所 B**: `writer.md` L52-56 「writerが生成しないもの」— メタナラティブの条件付き許容のみ、Ideas メタ指示の本文化禁止は明示なし
- **影響**: 第二の脳章0 v1 崩壊と同型の事故が再発しうる
- **推奨対応**: writer.md L52-56 に「Ideas に書かれた voice 注意は writer の態度の指針であり、本文に明示宣言として書かない」を追加

### M-14: voice_profile の voice_profile_guide 参照が浮いている

- **種別**: 場所違い
- **箇所 A**: `voice_profile.md` L58-62 — 「詳しくは templates/voice_profile_guide.md を参照」
- **箇所 B**: 実ファイル `templates/voice_profile_guide.md` の存在を `voice_profile.md` だけが指し示す（他の agent prompt には参照なし）
- **影響**: voice_profile_guide.md が存在するか、運用上どう使われるかが不明。voice_profile を読む agent はガイドを読まない
- **推奨対応**: voice_profile_guide.md の現状確認 → 存在しなければ参照削除、存在するなら CASPAR / SANDALPHON に「voice_profile_guide も合わせて読む」を追加（または不要なら参照を削除）

### M-15: タイトル32字制限の運用揺れ

- **種別**: 矛盾候補
- **箇所 A**: `publisher.md` L26 — 「32文字以内」
- **箇所 B**: `STYLE_GUIDE.md` — タイトル例にプレフィックス込みで 32 字超のものあり（例: `非エンジニアなのに、〇〇 〜意識低い系おじさんがAIと何かを作る話(12)〜` は本編1 で 30+ 字確定）
- **箇所 C**: 公開済み第二の脳プロローグ「【劣化した脳をAIで補完する話プロローグ】今の外部脳の仕組み」は 30 字超
- **影響**: publisher が「32 字以内」を厳格適用するとシリーズ固有のプレフィックス込みタイトルが収まらない
- **推奨対応**: publisher.md L26 を「主題部分（プレフィックスを除く）32 字以内を目安」に緩和、またはシリーズプレフィックスの扱いを明示

### M-16: voice_checker 5軸 vs SANDALPHON 5軸が重複

- **種別**: 重複
- **箇所 A**: `voice_checker.md` L33-80 — 5軸評価（レポート化 / 人間味 / 敬体ミックス / AI臭 / 一人称・基本）
- **箇所 B**: `sandalphon.md` L36-44, L66-82 — 5軸評価（レポート化 / 人間味 / 敬体ミックス / AI臭（声の観点） / 一人称）
- **影響**: 同じチェックを 2 段階で実施。意図的な二重チェックか、SANDALPHON が voice_checker の進化版か、不明
- **推奨対応**: SANDALPHON 自身が `voice_checker の進化版` と書いている（L4）。voice_checker を廃止 or SANDALPHON へ統合、または役割分担を明示（voice_checker = 修正、SANDALPHON = 判定のみ等）

### M-17: knowledge-base 書き込みルールが分散

- **種別**: 散在
- **箇所 A**: `CLAUDE.md` 「knowledge-base/ への書き込みルール」セクション — 詳細ルール
- **箇所 B**: 各 agent.md 末尾の「重要な制約」 — `knowledge-base/` 書き込み禁止を個別に記述（writer.md / voice_checker.md / editor.md / publisher.md / enricher.md / calibrator.md / SANDALPHON / MICHAEL / METATRON / MELCHIOR / BALTHASAR / CASPAR / AZRAEL）
- **影響**: CLAUDE.md が SSoT のはずだが、各 agent でルールが繰り返される。CLAUDE.md を更新しても各 agent 同期漏れリスク
- **推奨対応**: 各 agent では「CLAUDE.md の knowledge-base/ 書き込みルールに従う」の 1 行参照に圧縮。例外（knowledge-base/note/ 配下の書き込み許可 area が agent ごとに異なる）のみ個別記述

---

## Low（表現揺れレベル）

### L-01: 「note でやらないこと」と STYLE_GUIDE 禁止表現の重複

- **種別**: 重複
- **箇所 A**: `NOTE_CONCEPT.md` L61-66 — noteでやらないこと（ですます調 / 技術解説記事 / AIすごい系 / 絵文字多用 / 体験していないこと）
- **箇所 B**: `STYLE_GUIDE.md` 各所 — 同等内容が散在
- **推奨対応**: NOTE_CONCEPT を SSoT として「方向性」、STYLE_GUIDE を SSoT として「文体」、と役割を分離

### L-02: writer.md L96 「全体の文字数が型の目安に収まっているか」と M-08 の重複指摘

- **種別**: 重複（M-08 と統合可能）

### L-03: editor.md と publisher.md の H1/H2 説明型禁止の重複

- **種別**: 重複
- **箇所 A**: `editor.md` L68 Pass 4
- **箇所 B**: `publisher.md` L29-34 Step 1
- **影響**: 軽微。両方でチェックされるのは保険として機能
- **推奨対応**: 現状維持で OK、ただし feedback_no_explanatory_titles を SSoT として参照する形にすると保守容易

### L-04: voice_checker.md / SANDALPHON.md の保護ルールが似て非なる

- **種別**: 散在
- **箇所 A**: `voice_checker.md` L137-145 保護ルール（voice_profile / 構想の生の語り / メタナラティブ）
- **箇所 B**: `sandalphon.md` L83-89 保護ルール（メタナラティブ / 思考の跳躍 / 自覚的誇張 / 生の一言）
- **推奨対応**: M-16 と統合対応

### L-05: enricher.md / BALTHASAR.md の Obsidian CLI 言及

- **種別**: 散在
- **箇所 A**: `enricher.md` L19-22 (C-04 と重複)
- **箇所 B**: `balthasar.md` L22 — 「Bash（obsidian CLI 用 — なければGrep/Globで代替）」
- **影響**: BALTHASAR は fallback の明示があるので運用上は問題なし。enricher だけが「使う前提」で書かれている

### L-06: AZRAEL 起動時の「ファイルパスを渡さない」運用ルール

- **種別**: 場所違い
- **箇所 A**: `azrael.md` L52-59 — 「親エージェントは記事本文を Read で取得 → テキストを直接埋め込む」
- **箇所 B**: パイプライン実行を担う `.claude/commands/publish.md` (本監査では未読) — 不明
- **影響**: 起動を担う側の知識が AZRAEL.md にしか書かれていない。新規 agent 起動コードを書く人がこのルールを見落とす可能性
- **推奨対応**: publish.md（コマンド本体）でも明示

### L-07: 「過去記事の OK/NG 参照」が agent によって粒度が違う

- **種別**: 散在
- voice_checker / SANDALPHON / CASPAR — Calibration/ok 推奨参照
- METATRON — Calibration/ok 推奨参照
- editor — ng-examples/ 参照
- writer — Calibration 言及なし（voice-patterns のみ）
- **推奨対応**: 各 agent の OK/NG 参照ポリシーを統合または明示分離

### L-08: モデル指定の表記揺れ

- **種別**: 低
- 「Claude Opus」 / 「Opus」 / 「Claude Sonnet」 / 「Sonnet」 が混在
- **推奨対応**: 統一（軽微）

### L-09: handoff README の対応リポ表記揺れ

- **種別**: 低
- L14-17 「v0.3 対応リポ」と書きつつ実態は v0.6（最終版）
- **推奨対応**: ヘッダーを更新

### L-10: 「3つの仕事」 vs 「4 パス」等の構造的命名が独立して進化

- **種別**: 場所違い
- enricher = 3つの仕事、editor = 4 パス、publisher = 11 ステップ、SANDALPHON = 5 軸
- **影響**: 軽微、各 agent 独自の流儀

### L-11: handoff README v0.6 が「v0.3 対応リポ」と冒頭で言及

- **種別**: 低
- 改訂履歴で v0.6 まで進んでいるが冒頭の対応リポ節は v0.3 のまま
- **推奨対応**: 冒頭を最新版に同期

### L-12: 「メタナラティブ」の扱いが writer / voice_checker / SANDALPHON で微妙に違う

- **種別**: 散在
- writer L53 — 「実際の執筆過程を反映する場合のみ許容」
- voice_checker L143 — 削除禁止対象
- SANDALPHON L85 — 保護対象
- **推奨対応**: 役割分離は OK だが、SSoT 明示はあると親切

---

## カテゴリ別ダブり MAP

### 一人称（「自分」）

- `voice_profile.md` L9-11
- `STYLE_GUIDE.md` L8-10
- `voice_checker.md` L77, L80（SSoT 参照のみ）
- **推奨 SSoT**: STYLE_GUIDE
- **削除候補**: voice_profile L9-11 → 1 行参照に圧縮

### 文体（常体/敬体ミックス）

- `STYLE_GUIDE.md` L3-7
- `voice_profile.md` L13-17
- `NOTE_CONCEPT.md` L62
- **推奨 SSoT**: STYLE_GUIDE
- **削除候補**: voice_profile L13-17 をカジュアル語尾の具体例に絞る、NOTE_CONCEPT は参照のみ

### 短文・1文の長さ

- `STYLE_GUIDE.md` L13
- `voice_profile.md` L16
- `publisher.md` L51（60 字目安）
- **推奨 SSoT**: STYLE_GUIDE（60 字目安を吸収）
- **削除候補**: voice_profile / publisher で重複削除

### 体言止め・名詞止め

- `STYLE_GUIDE.md` L14 — 「積極的に使う」（**C-01 矛盾**）
- `voice-patterns.md` P-005 — 「多用禁止、動詞型は連発禁止」
- **推奨 SSoT**: voice-patterns P-005 + STYLE_GUIDE の修正同期
- **削除候補**: STYLE_GUIDE L14 を P-005 と同方針に書き換え

### 自虐 / ユーモア

- `STYLE_GUIDE.md` L28-32
- `voice_profile.md` L38-42
- `NOTE_CONCEPT.md` L13
- **推奨 SSoT**: voice_profile（4 型の具体化を残す）
- **削除候補**: STYLE_GUIDE と NOTE_CONCEPT を 1 行参照に

### 絵文字

- `STYLE_GUIDE.md` L53-54
- `voice_profile.md` L45
- `NOTE_CONCEPT.md` L65
- **推奨 SSoT**: STYLE_GUIDE
- **削除候補**: voice_profile / NOTE_CONCEPT 削除

### 禁止フレーズ

- `STYLE_GUIDE.md` L40-50
- `voice_profile.md` L45-47
- `voice-patterns.md` P-004〜P-029
- 各 agent.md（writer/voice_checker/MICHAEL）— 参照のみ
- **推奨 SSoT**: STYLE_GUIDE（形式的禁止） + voice-patterns（観察パターン）
- **役割分離を明示**: STYLE_GUIDE = 不変の禁止表現、voice-patterns = 観察・確定パターン

### タイトルフォーマット（シリーズ別）

- `STYLE_GUIDE.md` L56-68
- `CLAUDE.md` シリーズ現状セクション
- **推奨 SSoT**: STYLE_GUIDE
- **削除候補**: CLAUDE.md は 1 行参照に圧縮

### H1 = title 一致 + オーナー編集保護（P-012）

- `STYLE_GUIDE.md` L66-68（一致必須）
- `voice-patterns.md` P-012 確定（オーナー編集優先）
- `feedback_h1_owner_edit_protection.md`（オーナー編集優先、L18-22 で「一致は必須ではない」緩和）
- writer.md L159, L171 — 説明型 H1 禁止のみ
- voice_checker.md — **保護ルールに H1 明示なし**
- editor.md L68 — 説明型 H2 チェックのみ
- publisher.md L29-34 — 説明型タイトル禁止のみ
- **推奨 SSoT**: P-012 + feedback ファイル
- **修正候補**: STYLE_GUIDE L66-68 に P-012 例外節を追加、voice_checker.md 保護ルールに H1 を追加

### Claude Code 略禁止（P-019）

- `MEMORY.md` 索引
- `feedback_no_claude_code_abbreviation.md` 全文
- `voice-patterns.md` P-019
- writer.md / voice_checker.md / editor.md / publisher.md — **言及なし**
- **推奨 SSoT**: feedback ファイル
- **修正候補**: 4 つの agent prompt に明示追加

### 章N 呼称排除

- `MEMORY.md` 索引
- `feedback_no_chapter_number_in_conversation.md` 全文
- `handoff-magi-2026-05-13` 注意事項 2
- writer.md / voice_checker.md / editor.md / publisher.md — **言及なし**
- voice-patterns に P-XXX として登録されていない
- **推奨 SSoT**: feedback ファイル
- **修正候補**: 4 つの agent prompt に明示追加、voice-patterns に P-XXX として登録

### 具体日付・時刻禁止（P-018）

- `MEMORY.md` 索引
- `feedback_no_concrete_datetime.md` 全文
- `voice-patterns.md` P-018
- writer.md / voice_checker.md / editor.md / publisher.md — **言及なし**
- **推奨 SSoT**: feedback ファイル
- **修正候補**: 4 つの agent prompt に明示追加

### 章タイトル説明型禁止

- `feedback_no_explanatory_titles.md` 全文
- writer.md L159, L171（反映済）
- editor.md L68（反映済）
- publisher.md L29-34（反映済）
- voice_checker.md — **言及なし**
- **推奨 SSoT**: feedback
- **修正候補**: voice_checker.md にも追加

### 文字数オーバー警告廃止

- `feedback_word_count_policy.md` 全文
- writer.md L96 — 「目安に収まっているか」（**ルール違反残存**）
- editor.md L27 Pass 1 — 「目安に収まっているか」（**ルール違反残存**）
- **推奨 SSoT**: feedback
- **修正候補**: writer / editor から該当チェック削除

### snapshot 出力

- writer.md L130-147（実装済）
- voice_checker.md / editor.md / publisher.md — 言及なし
- **検討事項**: 後段 agent でオーナー編集後の snapshot が必要か運用判断

### knowledge-base/ 書き込みルール

- `CLAUDE.md` 「knowledge-base/ への書き込みルール」セクション
- 各 agent.md「重要な制約」（13 体すべて）
- **推奨 SSoT**: CLAUDE.md
- **削除候補**: 各 agent では 1 行参照に圧縮

### Obsidian CLI 使用前提

- `CLAUDE.md` 補助ツール（「未導入」明記）
- `enricher.md` L19-22, L95-97 — **使う前提で記述（aspirational 違反）**
- `balthasar.md` L22 — fallback 明示あり
- **修正候補**: enricher.md を Grep/Glob ベースに書き換え（C-04）

### NotePublishing/ vs knowledge-base/note/

- agent.md 全般 — `NotePublishing/` 旧 path 残存
- `CLAUDE.md` — `knowledge-base/note/` で統一
- **推奨 SSoT**: CLAUDE.md（新 path）
- **修正候補**: 全 agent prompt の path 一括置換

---

## 提案: SSoT 再配置案

| テーマ | 推奨 SSoT | 現状の散在 | 削除・修正候補 |
|---|---|---|---|
| 一人称 | STYLE_GUIDE | voice_profile + STYLE_GUIDE | voice_profile を 1 行参照に |
| 文体（常体/敬体） | STYLE_GUIDE | STYLE_GUIDE + voice_profile + NOTE_CONCEPT | voice_profile / NOTE_CONCEPT を参照に |
| 短文・1文長 | STYLE_GUIDE | STYLE_GUIDE + voice_profile + publisher | publisher 60 字を STYLE_GUIDE に吸収 |
| 体言止め | voice-patterns P-005 + STYLE_GUIDE | STYLE_GUIDE と P-005 が**矛盾** | **STYLE_GUIDE L14 を P-005 同方針に修正** |
| 自虐・ユーモア | voice_profile | STYLE_GUIDE + voice_profile + NOTE_CONCEPT | STYLE_GUIDE / NOTE_CONCEPT を参照に |
| 絵文字 | STYLE_GUIDE | 3 箇所 | voice_profile / NOTE_CONCEPT 削除 |
| 禁止フレーズ | STYLE_GUIDE（形式） + voice-patterns（観察） | 4 箇所 | 役割分離を明示 |
| タイトルフォーマット | STYLE_GUIDE | STYLE_GUIDE + CLAUDE.md | CLAUDE.md を参照に |
| H1 = title 一致 + 編集保護 | feedback_h1_owner_edit_protection + P-012 | STYLE_GUIDE / 4 feedback / 4 agent / voice-patterns | STYLE_GUIDE に例外節追加、voice_checker.md 保護ルールに H1 追加 |
| Claude Code 略禁止 | feedback ファイル | memory / voice-patterns | **4 agent prompt に展開**（writer/voice_checker/editor/publisher） |
| 章N 呼称排除 | feedback ファイル | memory / handoff | **4 agent prompt に展開** + voice-patterns に P-XXX 登録 |
| 具体日付・時刻禁止 | feedback ファイル | memory / voice-patterns | **4 agent prompt に展開** |
| 章タイトル説明型禁止 | feedback ファイル | 3 agent 反映済 | voice_checker.md に追加 |
| 文字数オーバー警告廃止 | feedback ファイル | writer / editor に違反残存 | writer.md L96 / editor.md L27 のチェック削除 |
| snapshot 出力工程 | writer.md | writer のみ | 必要なら voice_checker / editor / publisher へ展開 |
| knowledge-base/ 書込ルール | CLAUDE.md | CLAUDE.md + 13 agent | 各 agent で 1 行参照に |
| Obsidian CLI 運用 | CLAUDE.md（未導入明記） | CLAUDE.md + enricher 矛盾 | **enricher.md を Grep/Glob ベースに書き換え** |
| パス（NotePublishing/ → knowledge-base/note/） | CLAUDE.md | agent 全般に旧 path 残存 | 全 agent 一括置換 |

---

## 緊急修正候補（今すぐ、3-5 個に絞る）

優先度高い順:

### 1. **STYLE_GUIDE.md L14 の「体言止め積極推奨」を P-005 同方針に書き換え**（C-01）
- 直接矛盾、再発リスク最大
- writer が STYLE_GUIDE を素直に読むほど誤動作
- 修正コスト: 1 行書き換え

### 2. **enricher.md の Obsidian CLI 構文を Grep/Glob ベースに書き換え**（C-04）
- CLAUDE.md の「aspirational 書かない」ルールに直接違反
- 過去事故例として CLAUDE.md 自身が言及している件の取り残し
- 修正コスト: enricher.md L19-22, L95-97 を Grep/Glob 構文に置換

### 3. **章N 呼称排除を writer/voice_checker/editor/publisher の prompt に展開**（C-05）
- feedback で「agent prompt に明示」と書かれているのに 4 agent すべて漏れ
- オーナー手作業負荷が毎回発生
- 修正コスト: 各 agent prompt に 1 文追加 × 4

### 4. **Claude Code 略禁止（P-019）を 4 agent prompt + publisher 自動置換に展開**（C-06）
- 章4 公開版で AI 出力に 10+ 箇所残存実績
- voice-patterns 確定昇格候補
- 修正コスト: 各 agent prompt に 1 文追加 × 4 + publisher 自動置換手順追加

### 5. **voice_checker.md 保護ルールに H1 を追加 + STYLE_GUIDE に P-012 例外節を追加**（C-02, M-09）
- 章2 で実際に発生したバグの再発防止
- 修正コスト: voice_checker.md L137-145 + STYLE_GUIDE L66-68 に各 1-2 行追加

---

## 中期整理候補（10-20 個）

1. NotePublishing/ → knowledge-base/note/ の全 agent path 置換（M-10）
2. 具体日付・時刻禁止（P-018）を 4 agent prompt に展開（C-07）
3. 章タイトル説明型禁止を voice_checker.md にも展開（C-08）
4. 文字数オーバー警告チェックを writer.md L96 / editor.md L27 から削除（M-08）
5. Ideas メタ指示の本文化禁止（P-007）を writer.md に明示（M-13）
6. voice_checker と SANDALPHON の役割分離を明示、または統合（M-16）
7. snapshot 工程を voice_checker / editor / publisher にも検討（C-09）
8. knowledge-base/ 書込ルールを各 agent で 1 行参照に圧縮（M-17）
9. 一人称ルールの SSoT を STYLE_GUIDE に統一、voice_profile を圧縮（M-01）
10. 文体ルールの SSoT を STYLE_GUIDE に統一（M-02）
11. 自虐・ユーモアの 4 型を voice_profile に集約（M-04）
12. 絵文字ルールの SSoT を STYLE_GUIDE に統一（M-05）
13. タイトルフォーマットを STYLE_GUIDE に集約、CLAUDE.md は参照（M-07）
14. publisher.md L26 の 32 字制限を「主題部分のみ」に緩和（M-15）
15. voice_profile_guide.md の存在確認 → 不要なら参照削除（M-14）
16. handoff README ヘッダーを最新版に同期（L-09, L-11）
17. 「再構成 agent」の prompt template 化（C-08）
18. 過去記事 OK/NG 参照ポリシーを全 agent で統一（L-07）
19. agent モデル表記（Opus / Sonnet）の統一（L-08）
20. 写真・看板を象徴化しない（P-006）を writer.md「やってはいけないこと」に追加検討（M-12）
