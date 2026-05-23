---
name: phase1
description: 【内部用・自動起動禁止】親skillからの呼び出し専用。ユーザーが明示的に /madamisu:phase1 と入力した場合のみ単独起動可。それ以外では絶対に呼び出さない。マダミス生成のPhase 1（骨子確立）を実行し、ゲーム設定・世界観・キャラクターの初版を output/v{N}/ に生成する。
---

以下の手順でマダミスの **Phase 1: 骨子確立** を実行してください。

## 引数の解析

`$ARGUMENTS` から以下を取得してください。
- 第1引数: `SOURCE`（元資料のパス。フォルダまたはファイル）
- `--base <path>` オプション: `BASE`（ベースバージョンのフォルダパス。省略可）
- `--loops N` オプション: `MAX_LOOPS`（最大ループ回数。省略時デフォルト: 3）

## 元資料の読み込み

Readツールで `SOURCE` を読み込んでください。
- フォルダの場合: フォルダ内のすべての `.md` ファイルを読み込む
- ファイルの場合: そのファイルを読み込む

読み込んだ内容を `SOURCE_CONTENT` として記憶してください。

## 制作方針と品質閾値の読み込み

`output/_config.md` が存在する場合は Readツールで読み込み、以下を記憶してください:
- `CONFIG_CONTENT`: ファイル全体の内容（「品質閾値」セクション除く制作方針部分）
- `THRESHOLD_GAME`: 品質閾値セクションの「ゲーム性スコア閾値」（デフォルト: 6）
- `THRESHOLD_WORLD`: 「世界観スコア閾値」（デフォルト: 6）
- `THRESHOLD_CONFIG`: 「制作方針スコア閾値」（デフォルト: 8）

存在しない場合:
- `CONFIG_CONTENT = ""`（空、以降の注入とレビュー追加を省略）
- 閾値はデフォルト値を使用

## バージョン番号の決定

1. `output/` フォルダ内の既存 `v{N}` フォルダを確認し、次の番号 `NEXT_V` を決定する
   - 例: v1〜v3 が存在 → `NEXT_V = 4`
   - `output/` が存在しない → `NEXT_V = 1`
2. `BASE` が指定されている場合: `BASE` の全ファイルを `output/v{NEXT_V}/` にコピーする
3. `output/v{NEXT_V}/_working/` ディレクトリを作成する

## ループ実行（最大{MAX_LOOPS}回）

以下を最大{MAX_LOOPS}回繰り返す。**停止条件**: 🔴致命的矛盾 = 0件 OR {MAX_LOOPS}回完了。

ループ変数:
- `LOOP_NUM`: 現在のループ番号（1から開始）
- `IS_FIRST`: LOOP_NUM == 1 かどうか
- `PREV_V`: NEXT_V - 1（前のバージョン番号）
- `FOCUS`: ループ1→「初回・発散（複数案生成）」、ループ2→「動機の説得力」、ループ3→「トリックの物理的成立性」

---

> **制作方針の注入**: `CONFIG_CONTENT` が空でない場合、以降の全エージェントのプロンプト冒頭に必ず以下を追加してください。`CONFIG_CONTENT` が空の場合はこの追加を省略してください。
>
> ```
> 【制作方針】
> {CONFIG_CONTENTの内容}
> ※上記の方針を最優先にして出力すること。「固定要素・活かしたい要素」に記載された内容は絶対に変更しないこと。
> ```

---

### ステップA: アイデア出し（2エージェントを並列起動）

Agentツールで以下の2エージェントを**同時に**起動してください。

**エージェント1: ゲーム性アイデア出し**

Agentツールで起動:
- subagent_type: `madamisu:game-idea`
- prompt（以下を埋めて渡す）:

  ```
  【動的変数】
  - SOURCE_CONTENT: {元資料の実内容}
  - IS_FIRST: {IS_FIRST}
  - NEXT_V: {NEXT_V}
  - PREV_V: {PREV_V}（IS_FIRST=false 時のみ）
  - FOCUS: {FOCUS}（IS_FIRST=false 時のみ）
  - CONFIG_CONTENT: {CONFIG_CONTENT}（空でない場合のみ）

  【出力先】
  output/v{NEXT_V}/_working/game-ideas.md
  ```

agent 定義: `${CLAUDE_PLUGIN_ROOT}/agents/game-idea.md`

**エージェント2: 世界観アイデア出し**

Agentツールで起動:
- subagent_type: `madamisu:world-idea`
- prompt（以下を埋めて渡す）:

  ```
  【動的変数】
  - SOURCE_CONTENT: {元資料の実内容}
  - IS_FIRST: {IS_FIRST}
  - NEXT_V: {NEXT_V}
  - PREV_V: {PREV_V}（IS_FIRST=false 時のみ）
  - FOCUS: {FOCUS}（IS_FIRST=false 時のみ）
  - CONFIG_CONTENT: {CONFIG_CONTENT}（空でない場合のみ）

  【出力先】
  output/v{NEXT_V}/_working/world-ideas.md
  ```

agent 定義: `${CLAUDE_PLUGIN_ROOT}/agents/world-idea.md`

---

### ステップB: レビュー（3エージェントを並列起動）

Agentツールで以下の3エージェントを**同時に**起動してください。
`CONFIG_CONTENT` が空の場合は **制作方針レビュー（エージェント5）をスキップ** してください（2並列で実行）。

**エージェント3: ゲーム性レビュー**

Agentツールで起動:
- subagent_type: `madamisu:game-review`
- prompt（以下を埋めて渡す）:

  ```
  【動的変数】
  - NEXT_V: {NEXT_V}
  - RUBRIC_PATH: ${CLAUDE_PLUGIN_ROOT}/rubrics/phase1-game.md
  - CONFIG_CONTENT: {CONFIG_CONTENT}（空でない場合のみ）
  - レビュー対象パス: output/v{NEXT_V}/_working/game-ideas.md

  【出力先】
  output/v{NEXT_V}/_working/game-review.md
  ```

agent 定義: `${CLAUDE_PLUGIN_ROOT}/agents/game-review.md`

**エージェント4: 世界観レビュー**

Agentツールで起動:
- subagent_type: `madamisu:world-review`
- prompt（以下を埋めて渡す）:

  ```
  【動的変数】
  - NEXT_V: {NEXT_V}
  - RUBRIC_PATH: ${CLAUDE_PLUGIN_ROOT}/rubrics/phase1-world.md
  - CONFIG_CONTENT: {CONFIG_CONTENT}（空でない場合のみ）
  - レビュー対象パス: output/v{NEXT_V}/_working/world-ideas.md

  【出力先】
  output/v{NEXT_V}/_working/world-review.md
  ```

agent 定義: `${CLAUDE_PLUGIN_ROOT}/agents/world-review.md`

**エージェント5: 制作方針レビュー（CONFIG_CONTENT が空でない場合のみ）**

Agentツールで起動:
- subagent_type: `madamisu:config-review`
- prompt（以下を埋めて渡す）:

  ```
  【動的変数】
  - NEXT_V: {NEXT_V}
  - RUBRIC_PATH: ${CLAUDE_PLUGIN_ROOT}/rubrics/phase1-config.md
  - CONFIG_CONTENT: {CONFIG_CONTENTの実内容}
  - レビュー対象パス:
    - output/v{NEXT_V}/_working/game-ideas.md
    - output/v{NEXT_V}/_working/world-ideas.md
    - output/_config.md

  【出力先】
  output/v{NEXT_V}/_working/config-review.md
  ```

agent 定義: `${CLAUDE_PLUGIN_ROOT}/agents/config-review.md`

---

### ステップC: 整合性チェック

Agentツールで以下のエージェントを起動してください。

Agentツールで起動:
- subagent_type: `madamisu:consistency`
- prompt（以下を埋めて渡す）:

  ```
  【動的変数】
  - NEXT_V: {NEXT_V}
  - RUBRIC_PATH: ${CLAUDE_PLUGIN_ROOT}/rubrics/consistency-phase1.md
  - PREV_V: {PREV_V}（前バージョンが存在する場合のみ）
  - CONFIG_CONTENT: {CONFIG_CONTENT}（空でない場合のみ）
  - レビュー対象パス:
    - output/v{NEXT_V}/_working/game-ideas.md
    - output/v{NEXT_V}/_working/world-ideas.md
    - output/v{NEXT_V}/_working/game-review.md
    - output/v{NEXT_V}/_working/world-review.md
    - output/v{NEXT_V}/_working/config-review.md（存在する場合）
    - output/_config.md（存在する場合）
    - output/v{PREV_V}/ゲーム設定.md、世界観.md、キャラクター.md（前バージョンが存在する場合）

  【出力先】
  output/v{NEXT_V}/_working/consistency.md
  ```

agent 定義: `${CLAUDE_PLUGIN_ROOT}/agents/consistency.md`

---

### ステップD: 資料まとめ

Agentツールで以下のエージェントを起動してください。

Agentツールで起動:
- subagent_type: `madamisu:synthesis`
- prompt（以下を埋めて渡す）:

  ```
  【動的変数】
  - IS_FIRST: {IS_FIRST}
  - NEXT_V: {NEXT_V}
  - PREV_V: {PREV_V}（IS_FIRST=false 時のみ）
  - LOOP_NUM: {LOOP_NUM}
  - FOCUS: {FOCUS}（IS_FIRST=false 時のみ）
  - CONFIG_CONTENT: {CONFIG_CONTENT}（空でない場合のみ）
  - 読み込み対象パス:
    - output/v{NEXT_V}/_working/game-ideas.md
    - output/v{NEXT_V}/_working/world-ideas.md
    - output/v{NEXT_V}/_working/game-review.md
    - output/v{NEXT_V}/_working/world-review.md
    - output/v{NEXT_V}/_working/consistency.md
    - output/v{NEXT_V}/_working/config-review.md（存在する場合）
    - output/_config.md（存在する場合）
    - output/v{PREV_V}/ゲーム設定.md、世界観.md、キャラクター.md（前バージョンが存在する場合）

  【出力先】
  - output/v{NEXT_V}/ゲーム設定.md
  - output/v{NEXT_V}/世界観.md
  - output/v{NEXT_V}/キャラクター.md
  - output/v{NEXT_V}/_meta.md
  ```

agent 定義: `${CLAUDE_PLUGIN_ROOT}/agents/synthesis.md`

---

### ステップE: 校正

Agentツールで以下のエージェントを起動してください。

Agentツールで起動:
- subagent_type: `madamisu:proofread`
- prompt（以下を埋めて渡す）:

  ```
  【動的変数】
  - NEXT_V: {NEXT_V}
  - 校正対象パス:
    - output/v{NEXT_V}/ゲーム設定.md
    - output/v{NEXT_V}/世界観.md
    - output/v{NEXT_V}/キャラクター.md

  【出力先】
  output/v{NEXT_V}/_working/proofread-log.md
  ```

agent 定義: `${CLAUDE_PLUGIN_ROOT}/agents/proofread.md`

---

### ステップF: ループ継続判定

以下を確認してください:
- `output/v{NEXT_V}/_working/consistency.md` の最終行から🔴件数を抽出
- `output/v{NEXT_V}/_working/game-review.md` の総合スコア
- `output/v{NEXT_V}/_working/world-review.md` の総合スコア
- `output/v{NEXT_V}/_working/config-review.md` の総合スコア（存在する場合）

判定ロジック:
1. 🔴件数 > 0 → **ループ継続**（致命的矛盾の解消が最優先）
2. 🔴件数 = 0 AND ゲーム性 >= {THRESHOLD_GAME} AND 世界観 >= {THRESHOLD_WORLD} AND（制作方針レビュー実施時のみ）制作方針 >= {THRESHOLD_CONFIG}
   → **ループ停止**（品質OK）
3. LOOP_NUM >= {MAX_LOOPS} → **ループ強制停止**
4. それ以外（🔴=0 だが一部閾値未達）→ **ループ継続**
   - 補強観点（次の `FOCUS`）: 閾値未達のレビュー軸のうち、スコアが最も低い観点の「最低スコア項目」を採用

ループ継続時: NEXT_V を +1、LOOP_NUM を +1 してステップA に戻る。

## ユーザー確認

ループ終了後、以下を表示してユーザーに確認を求めてください。

```
========================================
Phase 1 完了（output/v{NEXT_V}/）

整合性スコア: 🔴N件 / 🟡N件 / 🔵N件
ゲーム性: N/10 | 世界観: N/10 | 制作方針: N/10（未実施なら「未評価」）
実行ループ数: {LOOP_NUM}回
閾値（参考）: ゲーム性>={THRESHOLD_GAME}, 世界観>={THRESHOLD_WORLD}, 制作方針>={THRESHOLD_CONFIG}

output/v{NEXT_V}/ を確認してください。
Phase 2（プロット精錬）へ進みますか？ [y/n/f]
  y: 次フェーズへ
  n: 中断
  f: フィードバックを反映する
========================================
```

ユーザー入力後の分岐:
- 「y」: 次フェーズへ進む（マスタースキル経由の場合は呼び出し元の判断に委ねる、単体実行時はそのまま終了）
- 「n」: 「処理を中断しました。」と表示して終了
- 「f」:
  1. 「フィードバック内容を入力してください:」と表示してユーザー入力を受け付ける
  2. ユーザー入力を `FEEDBACK_INPUT` として記憶する
  3. Skillツールで `madamisu:feedback` skillを起動。引数として `--v {NEXT_V} "{FEEDBACK_INPUT}"` を渡して実行する
  4. フィードバック適用完了後、適用先 v 番号で `NEXT_V` を更新する
  5. このユーザー確認プロンプト（[y/n/f]）に戻る
