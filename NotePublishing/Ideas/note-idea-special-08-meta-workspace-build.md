---
created: 2026-04-23
tags:
  - note-candidate
status: flash
source_session: session-summary-2026-04-23-夜
---

# Meta-Workspace構想と大工事の記録

## 主題

2リポジトリ運用の限界から Meta-Workspace という概念に到達し、3者合議を経て実装に至るまでのプロセスドキュメンタリー。「設計 → 議論 → 実装」の全過程を時系列で追体験できる記事。

## 骨子

- 出発点
  - Offmap-Dev リポジトリの肥大化
  - 「両方に紐づく雑タスク」の居場所問題
  - 純度を保てないストレス
- Meta-Workspace という着想
  - メタレベル作業の居場所として新設
  - ハブアンドスポーク構造
  - 情報相互保持の実現手段として
- Chat側構想の最初のバージョン
  - 全部 Meta-Workspace に集約する案
  - 理想論としての全体像
- Code側からの反論
  - 既存の _handoffs/ 機構が稼働している事実
  - ゼロから作るのは車輪の再発明
  - Meta-Workspace のスコープ絞り込み提案
- v2 → v3 の進化
  - launchd駐在スクリプトによる digest 生成
  - outbound/inbound 分割による将来余地確保
  - LLMPJ umbrella 統一の先行実施
- magi 合流による追加論点
  - M1: Chat→magi 経路併存ルール
  - M2: 卒業時命名規則
  - M3: パイプライン副産物の回収
- 実装（大工事）
  - LLMPJ 移行
  - Meta-Workspace 新設
  - _handoffs/ 機構の拡張
  - digest 駐在スクリプト（予定）
- 完成したアーキテクチャの姿

## 主要素材

- 今回の全セッションログ
- investigation-report v1/v2/v3
- chat-to-code-response v1/v2
- handoff-magi-2026-04-22-reply
- decision-2026-04-22-chat-route-coexistence
- Meta-Workspace アーキテクチャ図（4/22 生成 HTML）

## 想定分量

6000〜8000字（長めの記事、前後編も検討）

## 備考

技術詳細に埋もれないよう、「決断の瞬間」「気づきの瞬間」を軸に物語化する。アーキテクチャ図をふんだんに使い、視覚的に理解しやすくする。前後編にする場合は「構想編 / 実装編」で切るのが自然。
