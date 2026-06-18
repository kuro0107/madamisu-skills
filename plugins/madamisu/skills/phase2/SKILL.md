---
name: phase2
description: 【内部用・自動起動禁止】親skillからの呼び出し専用。ユーザーが明示的に /madamisu:phase2 と入力した場合のみ単独起動可。それ以外では絶対に呼び出さない。マダミス生成のPhase 2（プロット精錬）を実行し、Phase 1の出力を基にプロットを練り込む。
---

以下の手順でマダミスの **Phase 2: プロット精錬** を実行してください。

## 引数の解析

`$ARGUMENTS` から以下を取得してください。
- 第1引数: `SOURCE`（元資料のパス。フォルダまたはファイル）
- `--base <path>` オプション: `BASE`（ベースとなるバージョンのフォルダパス。必須推奨）
- `--loops N` オプション: `MAX_LOOPS`（最大ループ回数。省略時デフォルト: 4）
- `--from-master` オプション: `FROM_MASTER`（指定時 true。scenario 経由時に付与される。省略時 false ＝単体実行）

## 元資料の読み込み

Readツールで `SOURCE` を読み込んでください。
- フォルダの場合: フォルダ内のすべての `.md` ファイルを読み込む
- ファイルの場合: そのファイルを読み込む

## ベースバージョンの確認

`BASE` が指定されている場合:
- `BASE` フォルダの全ファイルをReadツールで読み込む
- `BASE_CONTENT` として記憶する

`BASE` が指定されていない場合:
- `output/` フォルダ内の最新 `v{N}` を探して `BASE` とする

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

1. `output/` フォルダ内の既存 `v{N}` を確認し、次の番号 `NEXT_V` を決定する
2. `BASE` の全ファイルを `output/v{NEXT_V}/` にコピーする
3. `output/v{NEXT_V}/_working/` ディレクトリを作成する

## ループ実行（最大{MAX_LOOPS}回）

以下を最大{MAX_LOOPS}回繰り返す。**停止条件**: 🔴致命的矛盾 = 0件 OR {MAX_LOOPS}回完了。

ループ変数:
- `LOOP_NUM`: 現在のループ番号（1から開始）
- `IS_FIRST`: LOOP_NUM == 1 かどうか
- `PREV_V`: NEXT_V - 1
- `FOCUS`: ループ1→「初回・プロット構築」、ループ2→「証拠の配置と情報開示の順序」、ループ3→「各キャラクターの見せ場」、ループ4→「エンディングルートの多様性」

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

**エージェント1: ゲーム性アイデア出し（プロット特化）**

プロンプト（SOURCE_CONTENT・BASE_CONTENT を実際の内容で置換すること）:

```
あなたはマーダーミステリーの脚本家です。プレイヤーが公平に真相へ辿り着けるプロット設計を最優先にしてください。

【元資料】
{SOURCE_CONTENTの全文}

【Phase 1の骨子】
{BASE_CONTENTのゲーム設定.md・キャラクター.md の内容}

【タスク - IS_FIRST が true の場合】
以下を詳細に設計してください:
1. トゥルーエンド: 真犯人・動機・手段・アリバイ崩しの完全なストーリー
2. 各キャラクターの秘密と嘘をつく理由（真犯人以外も全員）
3. 証拠・手がかりの種類と配布タイミング
4. 複数エンディングの条件（トゥルーエンド以外も1〜2案）

【タスク - IS_FIRST が false の場合】
前回出力の弱点（フォーカス: {FOCUS}）を補強する改善案を生成してください。
ゼロから作り直さず、前回出力をベースに弱点箇所のみ改善すること。

【前回の出力（IS_FIRST が false の場合のみ）】
{output/v{PREV_V}/ゲーム設定.md の内容}
{output/v{PREV_V}/キャラクター.md の内容}

【出力】
Writeツールで output/v{NEXT_V}/_working/game-ideas.md に保存してください。
```

**エージェント2: 世界観アイデア出し（キャラクター深掘り特化）**

プロンプト（SOURCE_CONTENT・BASE_CONTENT を実際の内容で置換すること）:

```
あなたはマーダーミステリーのキャラクター・ドラマ設計の専門家です。各キャラクターの人間的魅力と複雑な動機を深掘りしてください。

【元資料】
{SOURCE_CONTENTの全文}

【Phase 1の骨子】
{BASE_CONTENTの世界観.md・キャラクター.md の内容}

【タスク - IS_FIRST が true の場合】
各キャラクターについて以下を詳細化してください:
- 事件前の人間関係の詳細（誰と何があったか）
- 隠している秘密と、それを隠す動機
- ゲーム内での行動パターン（何を守るために何を隠すか）
- プレイヤーへの見せ場（このキャラクターならではの魅力的なシーン）

【タスク - IS_FIRST が false の場合】
前回出力の弱点（フォーカス: {FOCUS}）を補強する改善案を生成してください。
ゼロから作り直さず、前回出力をベースに弱点箇所のみ改善すること。

【前回の出力（IS_FIRST が false の場合のみ）】
{output/v{PREV_V}/キャラクター.md の内容}

【出力】
Writeツールで output/v{NEXT_V}/_working/world-ideas.md に保存してください。
```

---

### ステップB: レビュー（3エージェントを並列起動）

Agentツールで以下の3エージェントを**同時に**起動してください。
`CONFIG_CONTENT` が空の場合は **制作方針レビュー（エージェント5）をスキップ** してください（2並列で実行）。

**エージェント3: ゲーム性レビュー（プロット）**

Agentツールで起動:
- subagent_type: `madamisu:game-review`
- prompt（以下を埋めて渡す）:

  ```
  【フェーズ固有指示】
  あなたはマーダーミステリーのプロットレビュアーです。真相到達ルートの完全性を検証してください。プレイヤーとして実際にプレイするつもりで評価すること。

  【動的変数】
  - NEXT_V: {NEXT_V}
  - RUBRIC_PATH: ${CLAUDE_PLUGIN_ROOT}/rubrics/phase2-game.md
  - CONFIG_CONTENT: {CONFIG_CONTENT}（空でない場合のみ）
  - レビュー対象パス: output/v{NEXT_V}/_working/game-ideas.md

  【出力先】
  output/v{NEXT_V}/_working/game-review.md
  ```

agent 定義: `${CLAUDE_PLUGIN_ROOT}/agents/game-review.md`

**エージェント4: 世界観レビュー（キャラクター）**

Agentツールで起動:
- subagent_type: `madamisu:world-review`
- prompt（以下を埋めて渡す）:

  ```
  【フェーズ固有指示】
  あなたはマーダーミステリーのキャラクターレビュアーです。各キャラクターの人間的魅力と動機の深さを評価してください。

  【動的変数】
  - NEXT_V: {NEXT_V}
  - RUBRIC_PATH: ${CLAUDE_PLUGIN_ROOT}/rubrics/phase2-world.md
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
  【フェーズ固有指示】
  あなたはマーダーミステリー制作方針レビュアーです。Phase 2で詳細化されたプロットが制作方針（_config.md）に忠実かを評価してください。

  特に厳しく評価すべき項目:
  - プレイ人数とキャラクター数の一致
  - PvP/協力指定とプロットの整合
  - エンディング構成指定と詳細化されたエンディング数の一致
  - Q12 固定要素の維持
  - 元資料準拠度指定（あれば）への整合

  【動的変数】
  - NEXT_V: {NEXT_V}
  - RUBRIC_PATH: ${CLAUDE_PLUGIN_ROOT}/rubrics/phase2-config.md
  - CONFIG_CONTENT: {CONFIG_CONTENTの実内容}
  - レビュー対象パス:
    - output/v{NEXT_V}/_working/game-ideas.md
    - output/v{NEXT_V}/_working/world-ideas.md
    - output/_config.md
    - output/v{PREV_V}/ゲーム設定.md（存在する場合）
    - output/v{PREV_V}/キャラクター.md（存在する場合）

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
  【フェーズ固有指示】
  あなたはマーダーミステリーの整合性チェック専門家です。プロットレベルの矛盾を全て指摘してください。

  【動的変数】
  - NEXT_V: {NEXT_V}
  - RUBRIC_PATH: ${CLAUDE_PLUGIN_ROOT}/rubrics/consistency-phase2.md
  - PREV_V: {PREV_V}（前バージョンが存在する場合のみ）
  - CONFIG_CONTENT: {CONFIG_CONTENT}（空でない場合のみ）
  - レビュー対象パス:
    - output/v{NEXT_V}/_working/game-ideas.md
    - output/v{NEXT_V}/_working/world-ideas.md
    - output/v{NEXT_V}/_working/game-review.md
    - output/v{NEXT_V}/_working/world-review.md
    - output/v{NEXT_V}/_working/config-review.md（存在する場合）
    - output/_config.md（存在する場合）
    - output/v{PREV_V}/ゲーム設定.md、output/v{PREV_V}/キャラクター.md（前バージョンが存在する場合）

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
  【フェーズ固有指示】
  あなたはマーダーミステリーの設定資料ライターです。Phase 2の成果をゲーム設定.mdとキャラクター.mdに統合してください。

  前バージョンをベースに以下を追記・更新してください。
  consistency.mdの🔴矛盾は必ず解消すること。
  変更箇所に HTML コメント <!-- Phase2 Loop{LOOP_NUM}修正: [修正内容の短い説明] --> を付けること。
  複数案統合時は最高総合スコア案を優先し、同点の場合はゲーム性・世界観のバランスを優先して判断してください。

  更新内容:
  - ゲーム設定.md: 「真相（GMのみ）」セクション（真犯人・動機・手段・証拠の完全版）、「エンディング条件」セクション（トゥルーエンド＋他エンディングの条件）を前バージョンをベースに詳細化
  - キャラクター.md: 各キャラクターの「秘密」「目的」を詳細化、「嘘をつく理由」「守りたいもの」「事件当日の行動方針」を追加
  - 世界観.md: Phase 1から変更なければ前バージョンをそのままコピー

  _meta.md のフェーズ番号は 2 とすること。

  【動的変数】
  - IS_FIRST: {IS_FIRST}
  - NEXT_V: {NEXT_V}
  - PREV_V: {PREV_V}（IS_FIRST=false 時のみ）
  - LOOP_NUM: {LOOP_NUM}
  - FOCUS: {FOCUS}（IS_FIRST=false 時のみ）
  - CONFIG_CONTENT: {CONFIG_CONTENT}（空でない場合のみ）
  - 読み込み対象パス:
    - output/v{NEXT_V}/_working/ 以下の全ファイル
    - output/v{PREV_V}/ゲーム設定.md、キャラクター.md、世界観.md（前バージョンが存在する場合）
    - output/_config.md（存在する場合）

  【出力先】
  - output/v{NEXT_V}/ゲーム設定.md
  - output/v{NEXT_V}/キャラクター.md
  - output/v{NEXT_V}/世界観.md
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
1. 🔴件数 > 0 → **ループ継続**
2. 🔴件数 = 0 AND ゲーム性 >= {THRESHOLD_GAME} AND 世界観 >= {THRESHOLD_WORLD} AND（制作方針レビュー実施時のみ）制作方針 >= {THRESHOLD_CONFIG}
   → **ループ停止**
3. LOOP_NUM >= {MAX_LOOPS} → **ループ強制停止**
4. それ以外 → **ループ継続**
   - 補強観点（次の `FOCUS`）: 閾値未達のレビュー軸のうち、スコアが最も低い観点の「最低スコア項目」を採用

ループ継続時: NEXT_V を +1、LOOP_NUM を +1 してステップA に戻る。

## ユーザー確認

ループ終了後、以下のサマリを表示してください。

```
========================================
Phase 2 完了（output/v{NEXT_V}/）

整合性スコア: 🔴N件 / 🟡N件 / 🔵N件
ゲーム性: N/10 | 世界観: N/10 | 制作方針: N/10（未実施なら「未評価」）
実行ループ数: {LOOP_NUM}回
閾値（参考）: ゲーム性>={THRESHOLD_GAME}, 世界観>={THRESHOLD_WORLD}, 制作方針>={THRESHOLD_CONFIG}

output/v{NEXT_V}/ を確認してください。
========================================
```

続けて、`FROM_MASTER` に応じてメニューを表示する。

`FROM_MASTER = true`（scenario 経由）の場合:
```
Phase 3（タイムスケジュール）へ進みますか？
  [y] 次フェーズへ
  [n] 中断
  [f] フィードバックを反映する
  [l] もう1ループ追加実行する
```

`FROM_MASTER = false`（単体実行）の場合:
```
この出力で完了とします。どうしますか？
  [d] 完了（終了）
  [n] 中断
  [f] フィードバックを反映する
  [l] もう1ループ追加実行する
```

ユーザー入力後の分岐:
- 「y」（FROM_MASTER=true のみ）: 次フェーズへ進む（呼び出し元 scenario の判断に委ねる）。
- 「d」（FROM_MASTER=false のみ）: 「Phase 2 を完了しました。output/v{NEXT_V}/ に生成されています。」と表示して終了。
- 「n」: 「処理を中断しました。」と表示して終了。
- 「f」:
  1. 「フィードバック内容を入力してください:」と表示してユーザー入力を受け付ける。
  2. ユーザー入力を `FEEDBACK_INPUT` として記憶する。
  3. Skillツールで `madamisu:feedback` skill を起動。引数として `--v {NEXT_V} "{FEEDBACK_INPUT}"` を渡して実行する。
  4. フィードバック適用完了後、適用先 v 番号で `NEXT_V` を更新する。
  5. このユーザー確認メニューに戻る。
- 「l」（追加ループ）:
  1. 今回限りで loop を1回追加実行する。NEXT_V を +1、LOOP_NUM を +1 する。
  2. `FOCUS` を「ループ継続判定」と同じ規則で再算出する（閾値未達レビュー軸の最低スコア項目。全閾値達成済なら直近の FOCUS を再利用）。
  3. ステップA〜F を1回実行する（最新 v を base とする）。
  4. 実行後、このユーザー確認メニューに戻る。
