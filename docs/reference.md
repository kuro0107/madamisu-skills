# madamisu-skills リファレンス

## 目次

1. [skill一覧](#skill一覧)
2. [出力ファイル構造](#出力ファイル構造)
3. [output/_config.md の構造](#output_configmd-の構造)
4. [rubric一覧](#rubric一覧)
5. [agents 一覧](#agents-一覧)
6. [よくある使い方パターン](#よくある使い方パターン)

---

## skill一覧

### `/madamisu:scenario` — メインスキル（全フェーズ一括）

```
/madamisu:scenario <元資料のパス> [--loops N] [--p1-loops N] [--p2-loops N] [--p3-loops N]
```

| 引数 | 必須 | 説明 |
|---|---|---|
| `<元資料のパス>` | ✅ | フォルダまたはMarkdownファイルのパス |
| `--loops N` | — | 全フェーズのデフォルトループ上限を N に統一 |
| `--p1-loops N` | — | Phase 1 のみ上限 N に上書き |
| `--p2-loops N` | — | Phase 2 のみ上限 N に上書き |
| `--p3-loops N` | — | Phase 3 のみ上限 N に上書き |

優先順位: `--p{N}-loops` > `--loops` > フェーズ別デフォルト（P1:3, P2:4, P3:3）

---

### `/madamisu:phase1` — Phase 1（骨子確立）

```
/madamisu:phase1 <元資料のパス> [--base <バージョン>] [--loops N]
```

| 引数 | 必須 | 説明 |
|---|---|---|
| `<元資料のパス>` | ✅ | 元資料のパス |
| `--base <バージョン>` | — | 引き継ぎ元バージョン（例: `output/v3`）。省略時は新規スタート |
| `--loops N` | — | ループ上限上書き |

出力: `世界観.md`, `キャラクター.md`, `ゲーム設定.md`, `_meta.md`

---

### `/madamisu:phase2` — Phase 2（プロット精錬）

```
/madamisu:phase2 <元資料のパス> [--base <バージョン>] [--loops N]
```

引数仕様は Phase 1 と同一。

出力: `キャラクター.md`（更新）, `ゲーム設定.md`（更新）, `_meta.md`（更新）

---

### `/madamisu:phase3` — Phase 3（タイムスケジュール構築）

```
/madamisu:phase3 <元資料のパス> [--base <バージョン>] [--loops N]
```

引数仕様は Phase 1 と同一。

出力: `タイムスケジュール.md`, `_meta.md`（更新）

---

### `/madamisu:feedback` — フィードバック反映

```
/madamisu:feedback [--v N] [--light|--full] "<フィードバック内容>"
```

| 引数 | 必須 | 説明 |
|---|---|---|
| `"<フィードバック内容>"` | ✅ | 反映したい指摘・要望（引用符で括る） |
| `--v N` | — | 対象バージョン番号。省略時は最新v |
| `--light` | — | 軽量フロー強制（文言修正・追加情報向け） |
| `--full` | — | 重量フロー強制（根本変更・新規創作向け） |

フロー自動判定:
- **軽量**: 文言修正・追加情報・明示的な局所指示 → まとめ → 簡易整合性 → 校正（同一v上書き）
- **重量**: 抽象要望・根本変更・新規創作 → アイデア出し → レビュー3並列 → 整合性 → まとめ → 校正（新v作成）

---

### `/madamisu:clean` — 中間ファイル削除

```
/madamisu:clean
```

`output/v*/` 配下のすべての `_working/` を削除する。確認プロンプトで `y` を入力すると実行。最終納品物（.md）は残る。

---

### `/madamisu:publish` — プレイ用資料作成（全工程一括）

```
/madamisu:publish [<入力ディレクトリ>]
```

| 引数 | 必須 | 説明 |
|---|---|---|
| `<入力ディレクトリ>` | — | 省略時は `output/v*/` の最新vを自動検出 |

工程0〜7を順次実行。各工程はサブスキルに委譲。

---

### `/madamisu:publish-step0` 〜 `/madamisu:publish-step7` — 工程別スキル

| スキル | 工程 | 内容 |
|---|---|---|
| `step0` | 前提確認 | 入力資料の存在・形式チェック |
| `step1` | ヒアリング | 難易度・プレイ時間・書き口・テーマ等の確認 |
| `step2` | ルール設計 | フェーズ構成・情報開示マトリクス設計 |
| `step3` | 設計確認 | 設計案をユーザー確認 `[y/n/f]` |
| `step4` | 資料作成 | GM資料・ハンドアウト・証拠・イントロ・ルール・エンディング等を並列生成 |
| `step5` | 四段階レビュー | プレイアビリティ・整合性/網羅性・ストレステスト・校正 |
| `step6` | 修正反映 | レビュー指摘を反映（軽微/資料は自動、設計はユーザー確認） |
| `step7` | 最終確認・納品 | スコアサマリー表示・最終確認 |

各スキルの `$ARGUMENTS`:

```
/madamisu:publish-step0 <入力ディレクトリ> --v <publishバージョン>
/madamisu:publish-step1 <入力ディレクトリ> --v <publishバージョン>
/madamisu:publish-step2 <入力ディレクトリ> --v <publishバージョン>
/madamisu:publish-step3 <入力ディレクトリ> --v <publishバージョン>
/madamisu:publish-step4 <入力ディレクトリ> --v <publishバージョン>
/madamisu:publish-step5 <入力ディレクトリ> --v <publishバージョン>
/madamisu:publish-step6 <入力ディレクトリ> --v <publishバージョン>
/madamisu:publish-step7 <入力ディレクトリ> --v <publishバージョン>
```

---

## 出力ファイル構造

### `output/` — 設定生成フェーズの出力

```
output/
  _config.md              制作方針（ヒアリング結果）
  v1/                     Phase 1 ループ1回目完了
    ゲーム設定.md
    世界観.md
    キャラクター.md
    _meta.md
    _working/             中間ファイル（自動削除対象）
  v2/                     Phase 1 ループ2回目完了
    ...
  v3/                     Phase 1 最終（ユーザー確認時点）
    ...
  v4/                     Phase 2 開始（v3をベースにコピー）
    ゲーム設定.md
    世界観.md
    キャラクター.md
    _meta.md
  ...
  vN/                     Phase 3 完了
    ゲーム設定.md
    世界観.md
    キャラクター.md
    タイムスケジュール.md
    _meta.md
```

| ファイル | 生成フェーズ | 内容 |
|---|---|---|
| `ゲーム設定.md` | Phase 1〜2 | タイトル・あらすじ・事件概要・ゲーム構成 |
| `世界観.md` | Phase 1 | 舞台・時代・雰囲気・ロケーション詳細 |
| `キャラクター.md` | Phase 1〜2 | 全PC/NPCのプロフィール・動機・秘密 |
| `タイムスケジュール.md` | Phase 3 | 事件前後の全キャラ行動・アリバイ・タイムライン |
| `_meta.md` | 全フェーズ | ループ回数・各スコア・変更サマリー |

### `publish/` — プレイ資料作成フェーズの出力

```
publish/
  v1/
    01_requirements.md         ヒアリング結果
    02_game_design.md          ゲームデザイン設計書
    02_info_disclosure_matrix.md  情報開示マトリクス
    04_gm.md                   GM（ゲームマスター）資料
    04_handout_<キャラ名>.md   プレイヤーハンドアウト（キャラ数分）
    04_evidence_<連番>.md      証拠資料（証拠数分）
    04_intro.md                イントロダクション
    04_rules.md                ゲームルール
    04_endings.md              エンディング集
    04_postgame.md             ポストゲーム（解説・種明かし）
    05_review_report.md        レビュー報告書
    _meta.md                   メタ情報
```

---

## output/_config.md の構造

Phase 0.5 のヒアリング結果が保存される。全フェーズ共通の制作方針として参照される。

```markdown
# 制作方針

## 基本設定
- プレイ人数: N人
- プレイ時間: N時間
- ゲームタイプ: PvP / 協力型 / 半協力型
- ...

## 品質閾値
- ゲーム性スコア閾値: 6
- 世界観スコア閾値: 6
- 制作方針スコア閾値: 8

## ジャンル・雰囲気
...
```

**編集可能な品質閾値:**

| 項目 | デフォルト | 説明 |
|---|---|---|
| ゲーム性スコア閾値 | 6 | この値以上でゲーム性OKと判定（0〜10） |
| 世界観スコア閾値 | 6 | この値以上で世界観OKと判定（0〜10） |
| 制作方針スコア閾値 | 8 | この値以上で制作方針準拠と判定（0〜10） |

閾値を上げると品質基準が厳しくなりループ回数が増える。`--loops 1` との併用でトークンをコントロールできる。

---

## rubric一覧

`plugins/madamisu/rubrics/` 配下のファイルがエージェントの評価基準として使われる。ファイルを編集することでプロジェクト固有の評価軸を追加・調整できる。

### 設定生成フェーズ用

| ファイル | 参照エージェント | 役割 |
|---|---|---|
| `phase1-game.md` | ゲーム性レビュー（Phase 1） | 骨子のゲームとしての面白さチェック観点 |
| `phase1-world.md` | 世界観レビュー（Phase 1） | 世界観・キャラ造形の魅力チェック観点 |
| `phase1-config.md` | 制作方針レビュー（Phase 1） | _config.md準拠・人数/キャラ数整合チェック観点 |
| `phase2-game.md` | ゲーム性レビュー（Phase 2） | プロット・動機・秘密のゲーム性チェック観点 |
| `phase2-world.md` | 世界観レビュー（Phase 2） | キャラ深度・関係性の世界観チェック観点 |
| `phase2-config.md` | 制作方針レビュー（Phase 2） | Phase 2での制作方針準拠チェック観点 |
| `phase3-game.md` | ゲーム性レビュー（Phase 3） | タイムスケジュールのゲーム性チェック観点 |
| `phase3-world.md` | 世界観レビュー（Phase 3） | タイムスケジュールの世界観チェック観点 |
| `phase3-config.md` | 制作方針レビュー（Phase 3） | Phase 3での制作方針準拠チェック観点 |
| `consistency-phase1.md` | 整合性チェック（Phase 1） | 🔴/🟡/🔵の矛盾分類観点（Phase 1用） |
| `consistency-phase2.md` | 整合性チェック（Phase 2） | 🔴/🟡/🔵の矛盾分類観点（Phase 2用） |
| `consistency-phase3.md` | 整合性チェック（Phase 3） | 🔴/🟡/🔵の矛盾分類観点（Phase 3用） |

### プレイ資料作成フェーズ用

| ファイル | 参照エージェント | 役割 |
|---|---|---|
| `publish-difficulty.md` | 全工程共通 | 難易度別指針（フェーズ数・手がかり数・ミスリード密度・時間配分等） |
| `publish-playability.md` | step5プレイアビリティレビュー | 視点別シミュレーションの評価観点 |
| `publish-consistency.md` | step5整合性/網羅性レビュー | 論理整合・情報網羅の評価観点 |
| `publish-coverage.md` | step5整合性/網羅性レビュー | ゲーム情報の網羅度評価観点 |
| `publish-stress-test.md` | step5ストレステスト | メタ読み・暴走・消極的プレイの堅牢性評価観点 |
| `publish-proofread.md` | step5校正 | 誤字・表記揺れ・書き口統一の評価観点 |

**矛盾の重大度:**
- 🔴 致命的: ゲームが成立しない矛盾。解消必須
- 🟡 軽微: 気になるが許容範囲内
- 🔵 要確認: 意図的か誤りか判断できないもの

---

## agents 一覧

`plugins/madamisu/agents/` 配下のエージェント定義ファイル。各スキルから必要に応じて呼び出される。

| ファイル | 役割 |
|---|---|
| `game-idea.md` | ゲーム性アイデア出し |
| `world-idea.md` | 世界観アイデア出し |
| `game-review.md` | ゲーム性レビュー（rubric `RUBRIC_PATH` で動的指定） |
| `world-review.md` | 世界観レビュー（rubric `RUBRIC_PATH` で動的指定） |
| `config-review.md` | 制作方針レビュー（rubric `RUBRIC_PATH` で動的指定） |
| `consistency.md` | 整合性チェック（rubric `RUBRIC_PATH` で動的指定） |
| `synthesis.md` | 資料まとめ |
| `proofread.md` | 校正 |

---

## よくある使い方パターン

### トークンを節約したい

```
/madamisu:scenario 設定資料/ --loops 1
```

各フェーズ最大1ループで完了。品質は下がるがトークンを大幅節約。

### Phase 2だけ再実行してブラッシュアップ

```
/madamisu:phase2 設定資料/ --base output/v5
```

`output/v5` を引き継いで Phase 2 のみ再実行。

### フィードバックを軽く反映（文言修正等）

```
/madamisu:feedback --light "PC2の職業を医師から弁護士に変更"
```

軽量フロー強制。同一バージョンを上書き更新。

### フィードバックで大幅変更

```
/madamisu:feedback --full "犯人の動機を復讐から金銭目的に変えて全体を見直したい"
```

重量フロー強制。新バージョンを生成。

### Phase 1完了後に止めてプレイ資料を作る

```
/madamisu:scenario 設定資料/ --p1-loops 2 --p2-loops 0 --p3-loops 0
```

Phase 1のみ2ループで実行。Phase 2・3はスキップ（`--loops 0` で即停止）。

### プレイ資料だけを単独で作る（既存の設定資料から）

```
/madamisu:publish 自作設定資料/
```

madamisu の出力に依存せず、任意のディレクトリを入力として使える。
