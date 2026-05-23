---
name: feedback
description: 【内部用・自動起動禁止】親skillからの呼び出し専用。ユーザーが明示的に /madamisu:feedback と入力した場合のみ単独起動可。それ以外では絶対に呼び出さない。既存の output/v{N}/ にユーザーフィードバックを反映し、新しいvに書き出す。
---

以下の手順でmadamisuのフィードバック反映を実行してください。

## 引数の解析

`$ARGUMENTS` から以下を取得してください。

- 第1引数（必須）: `FEEDBACK_CONTENT`（フィードバック内容、複数行可）
- `--v N` オプション: `TARGET_V`（対象バージョン番号。省略時は `output/` 内の最新vを自動選択）
- `--light` オプション: 軽量フロー強制（`FORCE_MODE = "light"`）
- `--full` オプション: 重量フロー強制（`FORCE_MODE = "full"`）

`--light` と `--full` が同時指定された場合はエラー表示して終了:

```
エラー: --light と --full は同時指定できません。
```

`FEEDBACK_CONTENT` が空の場合は以下を表示して終了:

```
フィードバック内容を指定してください。
例: /madamisu:feedback "キャラAの動機をもっと深く"
例: /madamisu:feedback --v 5 --light "誤字修正: 探偵→刑事"
```

## 対象バージョンの確定

`TARGET_V` が指定されていない場合:
- Bash ツールで `ls -d output/v*/ 2>/dev/null | sort -V | tail -1` を実行して最新vを取得
- v番号を抽出して `TARGET_V` とする
- `output/v*` が存在しない場合は以下を表示して終了:
  ```
  エラー: output/ に v ディレクトリが見つかりません。
  先に /madamisu でマダミスを生成してください。
  ```

`TARGET_V` が指定されている場合:
- `output/v{TARGET_V}/` が存在することを Bash で確認
- 存在しない場合は以下を表示して終了:
  ```
  エラー: output/v{TARGET_V}/ が見つかりません。
  ```

## 対象フェーズの特定

Read ツールで `output/v{TARGET_V}/_meta.md` を読み込んでください。
存在しない場合は以下を表示して終了:

```
エラー: output/v{TARGET_V}/_meta.md が見つかりません、または対象フェーズが特定できません。
--v オプションで明示的にバージョンを指定してください。
```

`_meta.md` から `フェーズ:` 行を抽出し、`TARGET_PHASE` として記憶してください（値は `1`, `2`, `3` のいずれか、または `フィードバック (元: Phase X)` の形式の場合は X を抽出）。

特定不能の場合は上記エラーと同じメッセージで終了してください。

## 制作方針の読み込み

`output/_config.md` が存在する場合は Read で読み込み、`CONFIG_CONTENT` として記憶してください。
存在しない場合は `CONFIG_CONTENT = ""`（空）として扱い、以降のエージェント注入と制作方針レビューを省略してください。

## 判定エージェントの起動（FORCE_MODE が未指定の場合のみ）

`FORCE_MODE` が `"light"` または `"full"` の場合はこのセクションをスキップして以下のように設定:
- `MODE = FORCE_MODE`
- `CROSS_PHASE = false`（フラグ強制時はクロスフェーズ警告も省略）
- `AFFECTED_FILES = ""`（後段で全関連ファイルを対象とする）

`FORCE_MODE` が未指定の場合、まず `output/v{TARGET_V}/_working/` ディレクトリが存在しなければ作成し、Agent ツールで以下のエージェントを起動してください。

```
あなたはmadamisuフィードバック判定エージェントです。ユーザーからのフィードバック内容を分析し、修正フローの規模と影響範囲を判定してください。

【判定対象】
フィードバック内容:
{FEEDBACK_CONTENT}

対象バージョン: v{TARGET_V}（Phase {TARGET_PHASE} 完了バージョン）

【対象vのファイル】
Read ツールで以下を必要に応じて参照してください:
- output/v{TARGET_V}/ゲーム設定.md
- output/v{TARGET_V}/世界観.md
- output/v{TARGET_V}/キャラクター.md
- output/v{TARGET_V}/タイムスケジュール.md（Phase 3 完了v以降のみ存在）
- output/_config.md（存在する場合）

【判定軸1: 規模（軽量 / 重量）】

軽量フローと判定する条件（以下のいずれか）:
- 文言修正、誤字修正、表記揺れ修正
- 既存内容への追加情報（例:「キャラAに趣味を追加」）
- 明示的かつ局所的な指示（例:「23時の場所を会議室に変更」）
- 影響範囲が1〜2ファイル

重量フローと判定する条件（以下のいずれか）:
- 抽象的な要望（例:「もっと面白く」「キャラAを魅力的に」）
- 根本的な変更（例:「真犯人を変えたい」「動機を別物に」）
- 新規創作要素（例:「新しい伏線を追加」）
- 影響範囲が3ファイル以上

【判定軸2: 影響範囲（局所 / 横断）】

局所と判定する条件:
- フィードバックが対象v のフェーズに属する要素のみを変更
- 例: Phase 2 v に対し「キャラAの秘密を変更」（Phase 2 要素）

横断と判定する条件:
- フィードバックが対象v より前のフェーズ要素を変更
- 例: Phase 3 v に対し「キャラAの背景を変更」（Phase 1 要素）
- フェーズ別の要素分類:
  - Phase 1 要素: 事件骨子、世界観、キャラクター基本設定、舞台
  - Phase 2 要素: トゥルーエンド、動機詳細、秘密、エンディング条件
  - Phase 3 要素: タイムライン、アリバイ、行動記録、手がかり時刻

【出力】
Write ツールで output/v{TARGET_V}/_working/feedback-judgment.md に以下の形式で保存:

# フィードバック判定結果

- 判定: 軽量 / 重量
- 影響範囲: 局所 / 横断
- 対象ファイル: <ファイルのパスをカンマ区切りで列挙>
- 判定根拠: <2〜3文の説明>

判定結果は明確に「軽量」「重量」「局所」「横断」の文字列を使うこと（他の表現は使わない）。
```

エージェント完了後、Read ツールで `output/v{TARGET_V}/_working/feedback-judgment.md` を読み込み、以下を抽出してください:

- `MODE`: 「軽量」→ `"light"`、「重量」→ `"full"`
- `CROSS_PHASE`: 「横断」→ `true`、「局所」→ `false`
- `AFFECTED_FILES`: 対象ファイルリスト

## クロスフェーズ警告（CROSS_PHASE が true の場合のみ）

以下を表示してユーザー入力を待ってください:

```
========================================
警告: このフィードバックは Phase X 要素の変更を含みます。
対象 v{TARGET_V} は Phase {TARGET_PHASE} 完了バージョンです。

判定根拠: <feedback-judgment.md の判定根拠を表示>

選択肢:
1. フルやり直しを推奨（Phase X からの再生成）
   コマンド例: /madamisu:phase{該当Phase} 元資料 --base output/v{該当Phaseの完了v}
2. このバージョンで全関連ファイルを修正（重量フロー強制）
   注意: タイムライン等の整合性破綻リスクあり

[1/2/cancel]
========================================
```

- `1` 入力時: コマンド例を表示して終了
- `2` 入力時: `MODE = "full"` に強制設定、続行
- `cancel` または他の入力: 「処理を中断しました」と表示して終了

## バージョン処理

### 軽量フロー（MODE == "light"）の場合

- 適用先 = 既存 `output/v{TARGET_V}/`
- `output/v{TARGET_V}/_working/` が存在しなければ Bash で作成
- `APPLY_V = TARGET_V`

### 重量フロー（MODE == "full"）の場合

- 既存 v 番号の最大値 + 1 を `NEXT_V` として決定
- Bash で `output/v{TARGET_V}/` の全ファイル（_working/ を除く）を `output/v{NEXT_V}/` にコピー:
  ```bash
  mkdir -p output/v{NEXT_V}
  for f in output/v{TARGET_V}/*; do
    name=$(basename "$f")
    if [ "$name" != "_working" ]; then
      cp -r "$f" output/v{NEXT_V}/
    fi
  done
  mkdir -p output/v{NEXT_V}/_working
  ```
- `APPLY_V = NEXT_V`

## 修正フロー実行

### 軽量フロー（MODE == "light"）

Agent ツールで以下を順次起動してください。

**エージェント1: まとめ・編集（軽量）**

```
あなたはmadamisuのフィードバック反映エージェントです。ユーザーからのフィードバックを既存資料に正確に反映してください。

【制作方針】
{CONFIG_CONTENT}（空なら省略）
※上記の方針を最優先にして出力すること。「固定要素・活かしたい要素」に記載された内容は絶対に変更しないこと。

【フィードバック内容】
{FEEDBACK_CONTENT}

【対象ファイル】
{AFFECTED_FILES}（feedback-judgment.md から抽出した対象ファイル）

【タスク】
1. Read ツールで対象ファイルを読み込む
2. フィードバック内容を反映するため、Edit ツールで該当箇所を直接修正する
3. 修正した各箇所の直前に以下の HTML コメントを挿入する:
   <!-- Feedback {現在のタイムスタンプ YYYY-MM-DD HH:MM}: {フィードバック内容の要約30字以内} -->
4. 修正内容を output/v{APPLY_V}/_working/feedback-edit-log.md に記録する（変更ファイル名と変更箇所のサマリ）

【厳禁】
- 対象ファイル以外を変更しない
- フィードバックで指示されていない箇所を変更しない（最小限の編集）
- 既存の HTML コメントを削除しない
```

**エージェント2: 簡易整合性チェック（軽量）**

```
あなたはmadamisuの簡易整合性チェックエージェントです。フィードバック適用後の影響範囲のみを限定的にチェックしてください。

【チェック対象】
Read ツールで以下を読んでください:
- output/v{APPLY_V}/_working/feedback-edit-log.md（修正ログ）
- output/v{APPLY_V}/_working/feedback-judgment.md
- 修正されたファイル（feedback-edit-log.md から特定）

【チェック観点（影響範囲のみ）】
- フィードバックで触れた箇所と、その前後の既存記述との間に矛盾がないか
- 同じ概念が修正前と修正後で異なる表記に揺れていないか
- 修正により他キャラの行動・動機との論理的整合が破綻していないか

【出力形式】
Write ツールで output/v{APPLY_V}/_working/feedback-consistency.md に保存:

### 🔴 致命的（ゲームが成立しない矛盾）
### 🟡 軽微（修正推奨）
### 🔵 要確認（意図不明、意図的な可能性あり）

各項目: 該当箇所 / 問題内容 / 修正案

最終行に必ず記載: 🔴件数: N件

【補足】
軽量フローのためフルレビューは行いません。観点は「影響範囲の矛盾のみ」に限定してください。
```

**エージェント3: 校正（軽量）**

```
あなたは日本語校正の専門家です。フィードバックで修正されたファイルのみを校正してください。

【対象ファイル】
output/v{APPLY_V}/_working/feedback-edit-log.md を Read で読み、修正されたファイル一覧を取得してください。各ファイルを Read で読み、Edit ツールで誤字・表記揺れ・句読点不統一を修正してください。

【チェック項目】
1. 誤字・脱字
2. 表記揺れ（同じ概念が異なる表記で書かれていないか）
3. 句読点の不統一（「、」「，」の混在、全角半角の混在）
4. 敬体・常体の混在

【出力】
修正ログを Write ツールで output/v{APPLY_V}/_working/feedback-proofread-log.md に記録してください。
問題がなければ「校正完了: 修正なし」と記録してください。
```

### 重量フロー（MODE == "full"）

Agent ツールで以下を起動してください。対象フェーズ {TARGET_PHASE} に応じてルーブリックを切り替えます。

**エージェント1・2: アイデア出し（並列、フィードバック補強観点）**

`{TARGET_PHASE}` の値に応じて既存の Phase {TARGET_PHASE} アイデア出しエージェントと同じ役割で2エージェントを並列起動してください。ただし以下の差分を追加:

- 補強観点として `FEEDBACK_CONTENT` を冒頭に追加
- 出力は `output/v{APPLY_V}/_working/feedback-game-ideas.md` と `output/v{APPLY_V}/_working/feedback-world-ideas.md`

具体的なプロンプト構造は `${CLAUDE_PLUGIN_ROOT}/skills/phase{TARGET_PHASE}/SKILL.md` の「ステップA: アイデア出し」セクションを参考にし、以下を変更:
- 【タスク】を「以下のフィードバックを反映する改善案を生成」に置き換え:
  ```
  【フィードバック】
  {FEEDBACK_CONTENT}

  【タスク】
  対象vの既存内容をベースに、上記フィードバックを反映する改善案を生成してください。ゼロから作り直さず、フィードバックで触れられた箇所を中心に改善してください。
  ```
- 出力先を `feedback-*` プレフィックスに変更

**エージェント3・4・5: レビュー3並列**

既存 Phase {TARGET_PHASE} のレビューエージェント3つ（ゲーム性・世界観・制作方針）を並列起動。`CONFIG_CONTENT` が空の場合はエージェント5（制作方針レビュー）をスキップ（2並列）。

ルーブリック参照:
- `${CLAUDE_PLUGIN_ROOT}/rubrics/phase{TARGET_PHASE}-game.md`
- `${CLAUDE_PLUGIN_ROOT}/rubrics/phase{TARGET_PHASE}-world.md`
- `${CLAUDE_PLUGIN_ROOT}/rubrics/phase{TARGET_PHASE}-config.md`

レビュー対象:
- `output/v{APPLY_V}/_working/feedback-game-ideas.md`
- `output/v{APPLY_V}/_working/feedback-world-ideas.md`

出力:
- `output/v{APPLY_V}/_working/feedback-game-review.md`
- `output/v{APPLY_V}/_working/feedback-world-review.md`
- `output/v{APPLY_V}/_working/feedback-config-review.md`（_config.md 存在時のみ）

**エージェント6: 整合性チェック（フル）**

既存 Phase {TARGET_PHASE} の整合性チェックエージェントと同じ役割で起動。

ルーブリック参照: `${CLAUDE_PLUGIN_ROOT}/rubrics/consistency-phase{TARGET_PHASE}.md`

確認対象:
- `output/v{APPLY_V}/_working/feedback-game-ideas.md`
- `output/v{APPLY_V}/_working/feedback-world-ideas.md`
- `output/v{APPLY_V}/_working/feedback-game-review.md`
- `output/v{APPLY_V}/_working/feedback-world-review.md`
- `output/v{APPLY_V}/_working/feedback-config-review.md`（存在する場合）
- `output/v{TARGET_V}/` の既存ファイル群

出力: `output/v{APPLY_V}/_working/feedback-consistency.md`（最終行に🔴件数記載）

**エージェント7: まとめ（重量）**

```
あなたはマーダーミステリーの設定資料ライターです。フィードバックを反映した最終資料を統合してください。

【制作方針】
{CONFIG_CONTENT}（空なら省略）

【フィードバック内容】
{FEEDBACK_CONTENT}

【読み込むファイル】
Read ツールで以下を読んでください:
- output/v{APPLY_V}/_working/ 以下の全 feedback-*.md
- output/v{TARGET_V}/ゲーム設定.md
- output/v{TARGET_V}/世界観.md
- output/v{TARGET_V}/キャラクター.md
- output/v{TARGET_V}/タイムスケジュール.md（Phase 3 v以降のみ）

【タスク】
対象v の既存ファイルをベースに、フィードバックを反映した最終版を output/v{APPLY_V}/ 配下に Write で出力してください。
- consistency.md の🔴矛盾は必ず解消する
- 複数案統合時は最高総合スコア案を優先
- 変更箇所に HTML コメント <!-- Feedback v{APPLY_V}: {フィードバック要約30字以内} --> を付加

【出力ファイル】
対象フェーズに応じて以下を Write で生成:
- Phase 1 対象: ゲーム設定.md, 世界観.md, キャラクター.md
- Phase 2 対象: ゲーム設定.md, キャラクター.md, 世界観.md（変更なければ前バージョンをコピー）
- Phase 3 対象: タイムスケジュール.md（および必要に応じて他ファイル）

Write ツールで output/v{APPLY_V}/_meta.md を以下の形式で作成:

# フィードバック適用 メタ情報
- フェーズ: フィードバック (元: Phase {TARGET_PHASE})
- 元バージョン: v{TARGET_V}
- 判定: 重量
- ゲーム性スコア: N/10
- 世界観スコア: N/10
- 制作方針スコア: N/10（制作方針レビュー実施時のみ。未実施時は「未評価」）
- 整合性スコア: 🔴N件 / 🟡N件 / 🔵N件
- フィードバック内容: {FEEDBACK_CONTENT を1行サマリ化}
- 変更サマリー: （主な変更点を箇条書き）
```

**エージェント8: 校正（重量）**

既存 Phase {TARGET_PHASE} の校正エージェントと同じ。対象は output/v{APPLY_V}/ の変更されたファイル全て。
出力: `output/v{APPLY_V}/_working/feedback-proofread-log.md`

## 記録: _meta.md にフィードバック履歴を追記

### 軽量フローの場合

Edit ツールで `output/v{APPLY_V}/_meta.md` を編集してください。ファイル末尾に「フィードバック履歴」セクションが既に存在する場合はその下に新しい履歴を追記、存在しない場合は新規追加します。

追記内容:

```markdown

# フィードバック履歴

## {現在のタイムスタンプ YYYY-MM-DD HH:MM}
- 判定: 軽量
- 影響範囲: 局所
- 内容: {FEEDBACK_CONTENT を要約}
- 適用結果: {feedback-edit-log.md から抽出した変更ファイルと行数}
```

既に「# フィードバック履歴」セクションが存在する場合は、その末尾に `## {タイムスタンプ}` ブロックのみ追記してください。

### 重量フローの場合

エージェント7のまとめで新規作成された `output/v{APPLY_V}/_meta.md` に「フィードバック履歴」セクションを追記してください。書き出し:

```markdown

# フィードバック履歴

## {現在のタイムスタンプ YYYY-MM-DD HH:MM}
- 判定: 重量
- 影響範囲: {CROSS_PHASE が true なら「横断（重量フロー強制）」、false なら「局所」}
- 内容: {FEEDBACK_CONTENT を要約}
- 適用結果: {変更したファイルと行数の概要}
```

## フィードバック回数のカウント

Bash ツールで以下を実行し、`output/_config.md` に書かれた「累計フィードバック回数」をインクリメントしてください。

```bash
# _config.md 末尾に「# フィードバック累計」セクションがあるか確認
if grep -q "^# フィードバック累計" output/_config.md 2>/dev/null; then
  # 既存カウントをインクリメント
  current=$(grep "^- 回数:" output/_config.md | tail -1 | sed 's/.*: //')
  next=$((current + 1))
  # sed で置換
  sed -i.bak "s/^- 回数: $current/- 回数: $next/" output/_config.md
  rm -f output/_config.md.bak
else
  # 新規セクション追加
  cat >> output/_config.md << 'EOF'

# フィードバック累計

- 回数: 1
EOF
fi

# 結果を読み出して FEEDBACK_COUNT として記憶
FEEDBACK_COUNT=$(grep "^- 回数:" output/_config.md | tail -1 | sed 's/.*: //')
echo "FEEDBACK_COUNT=$FEEDBACK_COUNT"
```

`output/_config.md` が存在しない場合（フェーズ別スキル単体実行時等）はこの処理をスキップし、`FEEDBACK_COUNT = "不明"` として扱ってください。

## 完了表示

以下を表示してください。

```
========================================
フィードバック適用完了

判定: {MODE == "light" ? "軽量" : "重量"}
適用先: output/v{APPLY_V}/
スコアサマリ:
{重量フローの場合のみ:
  ゲーム性: N/10 | 世界観: N/10 | 制作方針: N/10（未実施なら「未評価」）
}
整合性: 🔴N件 / 🟡N件 / 🔵N件
フィードバック累計: {FEEDBACK_COUNT}回
========================================
```

`FEEDBACK_COUNT` が 5 の倍数（5, 10, 15, ...）の場合、上記に続けて以下を表示してください:

```
注意: 累計フィードバック5回ごとの警告です。
コストがかさんでいます。継続するかどうかご確認ください。
```

経路A（呼び出し元が madamisu:scenario/madamisu:phase{N} 経由）の場合、呼び出し元の `[y/n/f]` プロンプトに復帰します。
経路B（独立コマンド実行）の場合、以下を追加表示して終了します:

```
追加で修正したい場合: /madamisu:feedback --v {APPLY_V} "次の指摘"
```
