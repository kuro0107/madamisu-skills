---
name: madamisu-clean
description: 【内部用・自動起動禁止】親skillからの呼び出し専用。ユーザーが明示的に /madamisu-clean と入力した場合のみ単独起動可。それ以外では絶対に呼び出さない。output/ 配下の全 _working/ ディレクトリを削除する。
---

以下の手順で `output/` 配下のすべての `_working/` ディレクトリを削除します。

## 削除対象の列挙

Bash ツールで以下を実行し、削除対象のディレクトリを列挙してください:

```bash
TARGETS=$(find output -type d -name "_working" 2>/dev/null)
echo "$TARGETS"
COUNT=$(echo "$TARGETS" | grep -c "_working" 2>/dev/null || echo 0)
```

ターゲットが0件の場合は以下を表示して終了:

```
削除対象の _working/ ディレクトリは見つかりませんでした。
```

## ユーザー確認

ターゲットが1件以上ある場合、以下を表示して確認を求めてください:

```
削除対象（N個の _working/ ディレクトリ）:
output/v1/_working
output/v2/_working
...

これらを削除します。続行しますか？ [y/n]
```

`y` 以外が入力された場合は何もせず終了してください。

## 削除実行

`y` が入力された場合、Bash ツールで以下を実行してください:

```bash
find output -type d -name "_working" -exec rm -rf {} + 2>/dev/null
echo "削除完了"
```

削除エラーが発生してもメッセージのみ表示し処理を継続してください（権限エラー等は想定範囲）。

## 完了メッセージ

```
========================================
クリーンアップ完了！
N個の _working/ ディレクトリを削除しました。

最終納品物は output/v{最終番号}/ に残っています:
- ゲーム設定.md
- 世界観.md
- キャラクター.md
- タイムスケジュール.md
========================================
```
