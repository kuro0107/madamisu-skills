---
name: madamisu-publish
description: 生成済みマダミスシナリオ（output/v{N}/ または独自入力ディレクトリ）を入力に、プレイ用配布資料（GMマニュアル・プレイヤーハンドアウト・証拠資料・エンディング演出等）を publish/v{N}/ に作成する。ユーザーが「公開用資料作って」「配布資料生成」「ハンドアウト作成」と言ったとき、または /madamisu-publish 起動時に使う。
---

以下の手順でマダミスプレイ用資料を作成してください。

## 引数の解析

`$ARGUMENTS` から以下を取得してください:
- 第1引数（省略可）: `INPUT_DIR`（入力ディレクトリのパス）

`INPUT_DIR` が省略された場合:
- Bash で `ls -d output/v*/ 2>/dev/null | sort -V | tail -1` を実行
- 該当が見つかれば `INPUT_DIR = <最新v>`
- 見つからない場合は以下を表示して終了:
  ```
  エラー: 入力が指定されておらず、output/v*/ も見つかりません。
  引数で入力ディレクトリを指定してください。
  例: /madamisu-publish 設定資料/
  ```

`INPUT_DIR` が指定されている場合: そのパスを使用。Bash で存在チェック、なければエラー終了。

## バージョン番号の決定

1. `publish/` フォルダ内の既存 `v{N}` フォルダを確認し、次の番号 `NEXT_V` を決定する
2. `publish/v{NEXT_V}/_working/` ディレクトリを Bash で作成する

## 工程0: 前提資料の存在確認

Skillツールで `madamisu-publish-step0` skillを起動してください。
- `args` に渡す文字列: `"{INPUT_DIR} --v {PUBLISH_V}"`
（`PUBLISH_V` は前ステップで決定した `NEXT_V`）

工程0の結果で不足があれば、ユーザーに具体的に指摘し補完を求めて停止してください。

## 工程1: ヒアリング

Skillツールで `madamisu-publish-step1` skillを起動してください。
- `args` に渡す文字列: `"{INPUT_DIR} --v {PUBLISH_V}"`

## 工程2: ルール設計

Skillツールで `madamisu-publish-step2` skillを起動してください。
- 通常時の `args`: `"{INPUT_DIR} --v {PUBLISH_V}"`
- 工程3/工程6からの再呼出時の `args`: `"{INPUT_DIR} --v {PUBLISH_V} --feedback \"{FEEDBACK_CONTENT}\""`

## 工程3: 設計案ユーザー確認

Skillツールで `madamisu-publish-step3` skillを起動してください。
- `args` に渡す文字列: `"--v {PUBLISH_V}"`

ユーザー入力が:
- `y`: 工程4へ進む
- `n`: 中断し終了
- `f`: フィードバック入力を受付け → step2 を `--feedback "{内容}"` で再実行 → 工程3再表示

## 工程4: 資料作成

Skillツールで `madamisu-publish-step4` skillを起動してください。
- `args` に渡す文字列: `"{INPUT_DIR} --v {PUBLISH_V}"`

## 工程5: 四段階レビュー

Skillツールで `madamisu-publish-step5` skillを起動してください。
- `args` に渡す文字列: `"--v {PUBLISH_V}"`

## 工程6: 修正反映

Skillツールで `madamisu-publish-step6` skillを起動してください。
- `args` に渡す文字列: `"{INPUT_DIR} --v {PUBLISH_V}"`

設計レベル修正が発生した場合は工程2に戻り、工程3 → 工程4 → 工程5 → 工程6 ループを最大3回繰り返してください。

## 工程7: 最終確認・納品

Skillツールで `madamisu-publish-step7` skillを起動してください。
- `args` に渡す文字列: `"--v {PUBLISH_V}"`

## 全工程完了

工程7でユーザー承認後、以下を表示してください:

```
========================================
madamisu-publish 全工程完了！

納品物: publish/v{PUBLISH_V}/
使い方ガイド: publish/v{PUBLISH_V}/_meta.md

【各工程別スキルで再実行する場合】
工程N: /madamisu-publish-stepN <引数>
========================================
```
