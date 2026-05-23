---
name: publish-step5
description: 【内部用・自動起動禁止】親skillからの呼び出し専用。ユーザーが明示的に /madamisu:publish-step5 と入力した場合のみ単独起動可。それ以外では絶対に呼び出さない。madamisu:publishの工程5（四段階レビュー）を実行する。
---

以下の手順でmadamisu-publishの工程5（四段階レビュー）を実行してください。

## 引数の解析

- `--v N` オプション: `PUBLISH_V`

## 入力ファイル読み込み

Read ツールで:
- `publish/v{PUBLISH_V}/04_*.md`（工程4の全資料）
- `publish/v{PUBLISH_V}/02_info_disclosure_matrix.md`

## 5A: プレイアビリティレビュー

### 5A.1 視点別コンテキスト生成

`02_info_disclosure_matrix.md` を参照し、各キャラ × 各フェーズの「その時点で見える情報のみ」を抽出。

キャラクター名リストとフェーズ番号リストを抽出後、Bash で:

```bash
mkdir -p publish/v{PUBLISH_V}/_working
```

各キャラ × 各フェーズについて Write で `publish/v{PUBLISH_V}/_working/play_{キャラ名}_phase{番号}.md` を生成。
内容は「そのキャラがそのフェーズ時点で見える情報のみ」。他キャラのハンドアウトや未取得証拠は含めない。

### 5A.2 視点別シミュレーション（並列実行）

並列上限5。プレイヤー数 + GM が5を超える場合はバッチ分割。

各キャラ視点エージェントを起動（GMありの場合はGM視点も加える）:

```
あなたはマーダーミステリーのキャラ「{キャラ名}」を演じるプレイヤーです。

【厳命】
渡される情報は publish/v{PUBLISH_V}/_working/play_{キャラ名}_phase{番号}.md シリーズのみ。
他キャラのハンドアウトや未取得証拠は絶対に参照しない。

【観点】
Read ツールで ${CLAUDE_PLUGIN_ROOT}/rubrics/publish-playability.md を読み、キャラ視点エージェント用の観点に従って評価。

【処理】
フェーズ1から順に、play_{キャラ名}_phase{N}.md を Read で読み込み、各フェーズで以下を出力:
- このフェーズで自分は何を知っているか
- 何に疑問を持つか
- 次フェーズで何ができるか
- 手詰まりの兆候

最終フェーズ後:
- 真相到達可否（根拠付き）
- 自分の勝利条件達成可否
- 退屈な時間の有無
- 見せ場の有無

【出力】
Write ツールで publish/v{PUBLISH_V}/_working/05a_playability_{キャラ名}.md に保存。
```

GM視点エージェント（GMありのみ、全資料アクセス可）:
```
あなたはマーダーミステリーのGMです。全資料を読み、進行手順の漏れ、想定外質問への対応材料、判定基準の明確さを評価してください。

【入力】
publish/v{PUBLISH_V}/04_*.md 全ファイル
${CLAUDE_PLUGIN_ROOT}/rubrics/publish-playability.md のGM視点

【出力】
Write ツールで publish/v{PUBLISH_V}/_working/05a_playability_gm.md に保存。
```

## 5B: 整合性・網羅性レビュー（並列2）

**整合性レビュー**:
```
あなたはマーダーミステリーの整合性チェック専門家です。

【入力】
publish/v{PUBLISH_V}/04_*.md 全ファイル
${CLAUDE_PLUGIN_ROOT}/rubrics/publish-consistency.md

【出力】
Write ツールで publish/v{PUBLISH_V}/_working/05b_consistency.md に保存。
ルーブリック出力フォーマットに従う（🔴致命的 / 🟡軽微 / 🔵要確認）。
```

**網羅性レビュー**:
```
あなたはマーダーミステリーの網羅性チェック専門家です。

【入力】
publish/v{PUBLISH_V}/04_*.md 全ファイル
publish/v{PUBLISH_V}/02_info_disclosure_matrix.md
${CLAUDE_PLUGIN_ROOT}/rubrics/publish-coverage.md

【出力】
Write ツールで publish/v{PUBLISH_V}/_working/05b_coverage.md に保存。
ルーブリック出力フォーマットに従う。
```

## 5C: ストレステスト(並列3)

### 5C.1 メタ読み事前処理

Agent または直接処理で `publish/v{PUBLISH_V}/_working/05c_meta_summary.md` を生成:
- 各 04_handout_<キャラ名>.md の文字数・章立て
- 各キャラの初期所持証拠数（マトリクスから集計）
- 各キャラの設定の複雑さ
- 公開資料に登場する各キャラ名の言及頻度

本文は含めない。

### 5C.2 ストレステスト3並列

**メタ読みエージェント**:
```
【厳命】
publish/v{PUBLISH_V}/_working/05c_meta_summary.md のみ参照可。本文は見ない。

【観点】
Read で ${CLAUDE_PLUGIN_ROOT}/rubrics/publish-stress-test.md のメタ読み観点を読み、メタ情報から犯人・トゥルーエンド条件・キャラ重要度ランキングを推測（確信度付き）。

【出力】
publish/v{PUBLISH_V}/_working/05c_meta.md に保存。
```

**暴走エージェント**:
```
【入力】
publish/v{PUBLISH_V}/04_*.md 全ファイル
${CLAUDE_PLUGIN_ROOT}/rubrics/publish-stress-test.md の暴走観点

【シナリオ】
早期暴露・役割放棄・嘘つき・無関係発言 を 犯人/重要キャラ/キーアイテム保持者 の3パターンで処理。

【出力】
publish/v{PUBLISH_V}/_working/05c_disruption.md に保存。
```

**消極的エージェント**:
```
【入力】
publish/v{PUBLISH_V}/04_*.md 全ファイル
${CLAUDE_PLUGIN_ROOT}/rubrics/publish-stress-test.md の消極的観点

【パターン】
犯人沈黙・キーアイテム保持者沈黙 の2パターン処理。

【出力】
publish/v{PUBLISH_V}/_working/05c_passive.md に保存。
```

## 5D: 校正レビュー

```
あなたは日本語校正の専門家です。

【入力】
publish/v{PUBLISH_V}/04_*.md 全ファイル
publish/v{PUBLISH_V}/01_requirements.md（書き口指定）
${CLAUDE_PLUGIN_ROOT}/rubrics/publish-proofread.md

【出力】
publish/v{PUBLISH_V}/_working/05d_proofread.md に保存。
ルーブリック出力フォーマットに従う。
```

## 統合レポート生成

Agent で以下を起動:

```
【入力】
Read で publish/v{PUBLISH_V}/_working/05* の全ファイルを読む。

【タスク】
4段階のレビュー結果を統合し、publish/v{PUBLISH_V}/05_review_report.md を生成。

【フォーマット】
# 工程5 レビュー結果

## サマリ
- プレイアビリティ問題: N件
- 整合性問題: N件
- 網羅性問題: N件
- メタ読みリスク: 確信度N%（犯人）/ N%（トゥルー条件）
- 暴走耐性: 完全破綻N件 / 一部破綻M件
- 消極的耐性: 完全詰みN件 / 一部目標不能M件
- 校正問題: N件

## プレイアビリティ問題
### キャラA視点
...

## 整合性問題
...

## 網羅性問題
...

## ストレステスト結果
### メタ読み(確信度別)
- 犯人推定: N%
- トゥルー条件: N%

### 暴走シナリオ
...

### 消極的シナリオ
...

## 校正問題
...
```

## 完了表示

```
========================================
工程5 完了

レビューレポート: publish/v{PUBLISH_V}/05_review_report.md
========================================
```

## 完了条件

4段階レビュー完了、統合レポート出力済。
