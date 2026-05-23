# バージョニング規約

madamisu-skills は [Semantic Versioning 2.0.0](https://semver.org/lang/ja/) に従います。

## バージョン番号の意味

```
MAJOR.MINOR.PATCH
```

| セグメント | 変更タイミング |
|---|---|
| **MAJOR** | 後方互換性のない変更（スキル名変更・廃止、引数仕様の破壊的変更） |
| **MINOR** | 後方互換性のある機能追加（新スキル追加、新 agent 追加、出力フォーマット拡充） |
| **PATCH** | バグ修正・表現修正・rubric の微調整（既存動作に影響しない変更） |

## リリース手順

1. `plugins/madamisu/` の変更を private リポジトリに commit
2. `plugin.json` の `version` を更新
3. `/madamisu:sync-public` を実行し、`public/plugins/madamisu/` に同期
4. `public/CHANGELOG.md` に変更内容を追記
5. `public/` で `git commit` → `git tag vX.Y.Z` → `git push` → `git push --tags`

## ブランチ戦略

- `main` ブランチが常に最新リリース
- 開発中の破壊的変更は private リポジトリで完結させてから public に反映
- リリースタグは `vMAJOR.MINOR.PATCH` 形式（例: `v0.1.0`）

## 後方互換性の定義

以下は **破壊的変更（MAJOR バンプ必須）**:
- スキル名の変更・削除（`/madamisu:scenario` → `/madamisu:generate` など）
- agent 名の変更・削除
- 必須入力フォーマットの変更

以下は **非破壊的変更（MINOR/PATCH で可）**:
- 新スキル・新 agent の追加
- rubric の内容改善
- 出力品質の向上
- ドキュメントの更新
