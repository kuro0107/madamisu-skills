---
name: scenario
description: マダミス（マーダーミステリー）シナリオの自動生成。元資料（フォルダまたは.md）を入力にPhase 0〜3を順次実行し、ゲーム設定・世界観・キャラクター・タイムスケジュールを output/v{N}/ に出力する。ユーザーが「マダミス作って」「シナリオ生成」「マーダーミステリー作成」と言ったとき、または /madamisu:scenario 起動時に使う。
---

以下の手順でマダミス自動生成システムの全フェーズを実行してください。

## 引数の解析

`$ARGUMENTS` から以下を取得してください。
- 第1引数: `SOURCE`（元資料のパス）
- `--loops N`: `LOOPS_DEFAULT`（全フェーズ統一最大ループ数。省略時は各フェーズのデフォルト維持）
- `--p1-loops N`: `P1_LOOPS`（Phase 1の最大ループ数。省略時は `LOOPS_DEFAULT` → Phase1デフォルト(3)）
- `--p2-loops N`: `P2_LOOPS`（Phase 2の最大ループ数。省略時は `LOOPS_DEFAULT` → Phase2デフォルト(4)）
- `--p3-loops N`: `P3_LOOPS`（Phase 3の最大ループ数。省略時は `LOOPS_DEFAULT` → Phase3デフォルト(3)）

優先順位: `--p{N}-loops` > `--loops` > フェーズ別デフォルト

`SOURCE` が指定されていない場合は以下を表示して終了してください:

```
元資料のパスを指定してください。
例: /madamisu:scenario 設定資料/
例: /madamisu:scenario 元ネタ.md
```

## Phase 0+0.5: 制作方針の決定

Skillツールで `madamisu:config` skill を起動してください。
- `args` に渡す文字列: `"{SOURCE}"`
- config skill が要件定義・ヒアリングを行い `output/_config.md` を生成します。
- 既存の `output/_config.md` がある場合、config skill 内で「そのまま使う/設定し直す」が選択されます。

config 完了後、`output/_config.md` が存在することを前提に Phase 1 へ進んでください。

## Phase 1: 骨子確立

Skillツールで `madamisu:phase1` skillを起動してください。
- `args` に渡す文字列: `"{SOURCE} --loops {P1_LOOPS} --from-master"`
- skill本文の `$ARGUMENTS` 解析で `SOURCE` と `MAX_LOOPS` が取得されます。

Phase 1 完了後のユーザー確認で:
- 「y」が入力された場合: Bash で以下を実行して `PHASE1_RESULT` を取得し、Phase 2 へ進む
  ```bash
  PHASE1_RESULT=$(ls -d output/v*/ 2>/dev/null | sort -t'v' -k2 -n | tail -1)
  ```
- 「n」が入力された場合: 以下を表示して終了する

```
処理を中断しました。
output/ フォルダに生成途中の資料が保存されています。
続きから再開する場合: /madamisu:phase2 {SOURCE} --base {PHASE1_RESULT}
```

- 「f」が入力された場合: 以下を実行する
  1. 「フィードバック内容を入力してください:」と表示してユーザー入力を受け付ける
  2. ユーザーが入力した内容を `FEEDBACK_INPUT` として記憶する
  3. Skillツールで `madamisu:feedback` skillを起動。`args` に渡す文字列: `"--v {Phase 1完了時の最新v番号} \"{FEEDBACK_INPUT}\""`
  4. フィードバック適用完了後、適用先v番号で `PHASE1_RESULT` を更新する
  5. Phase 1 完了確認プロンプト（[y/n/f/l]）に戻る

## Phase 2: プロット精錬

Skillツールで `madamisu:phase2` skillを起動してください。
- `args` に渡す文字列: `"{SOURCE} --base {PHASE1_RESULT} --loops {P2_LOOPS} --from-master"`

Phase 2 完了後のユーザー確認で:
- 「y」が入力された場合: Bash で以下を実行して `PHASE2_RESULT` を取得し、Phase 3 へ進む
  ```bash
  PHASE2_RESULT=$(ls -d output/v*/ 2>/dev/null | sort -t'v' -k2 -n | tail -1)
  ```
- 「n」が入力された場合: 以下を表示して終了する

```
処理を中断しました。
output/ フォルダに生成途中の資料が保存されています。
続きから再開する場合: /madamisu:phase3 {SOURCE} --base {PHASE2_RESULT}
```

- 「f」が入力された場合: 以下を実行する
  1. 「フィードバック内容を入力してください:」と表示してユーザー入力を受け付ける
  2. ユーザーが入力した内容を `FEEDBACK_INPUT` として記憶する
  3. Skillツールで `madamisu:feedback` skillを起動。`args` に渡す文字列: `"--v {Phase 2完了時の最新v番号} \"{FEEDBACK_INPUT}\""`
  4. フィードバック適用完了後、適用先v番号で `PHASE2_RESULT` を更新する
  5. Phase 2 完了確認プロンプト（[y/n/f/l]）に戻る

## Phase 3: タイムスケジュール構築

Skillツールで `madamisu:phase3` skillを起動してください。
- `args` に渡す文字列: `"{SOURCE} --base {PHASE2_RESULT} --loops {P3_LOOPS} --from-master"`

## 全フェーズ完了

### 中間ファイルの自動削除

最終バージョン番号を `FINAL_V` として記録してください。

Bash ツールで以下を実行し、最終 v 以外の `_working/` ディレクトリを削除してください:

```bash
for dir in output/v*/; do
  v_num=$(basename "$dir" | sed 's/v//')
  if [ "$v_num" != "{FINAL_V}" ] && [ -d "$dir/_working" ]; then
    rm -rf "$dir/_working"
    echo "Deleted: $dir/_working"
  fi
done
```

Windows 環境では PowerShell を経由する場合があるが、Git Bash であれば上記で動作する。
削除エラーが発生した場合はメッセージ表示のみで処理を継続してください。

### 完了メッセージ表示

以下を表示してください:

```
========================================
全フェーズ完了！

生成された資料は output/v{FINAL_V}/ に保存されています。
各バージョンの _meta.md でスコア推移を確認できます。

中間バージョンの _working/ は自動削除されました（最終 v{FINAL_V}/_working/ は残しています）。
完全削除する場合: /madamisu:clean

【各フェーズ別スキルで再実行する場合】
Phase 1のみ: /madamisu:phase1 {SOURCE}
Phase 2のみ: /madamisu:phase2 {SOURCE} --base output/v{N}
Phase 3のみ: /madamisu:phase3 {SOURCE} --base output/v{N}
========================================
```
