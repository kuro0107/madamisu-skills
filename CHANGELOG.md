# CHANGELOG

## [0.1.1] - 2026-05-24

### Added
- `docs/versioning.md`: バージョニング規約ドキュメント
- `CHANGELOG.md`: 変更履歴の導入
- `examples/sample-input/mountain-villa/`: サンプル入力（山荘クローズドサークル）

## [0.1.0] - 2026-05-22

### Added
- Claude Code plugin 形式への対応（`/madamisu:<skill>` 名前空間）
- plugin agent 8 種（`game-idea`, `world-idea`, `game-review`, `world-review`, `config-review`, `consistency`, `synthesis`, `proofread`）
- `dev.ps1` によるローカル開発起動ラッパー
- `marketplace.json` によるセルフホスト型マーケットプレース対応

### Changed
- スキル名から `madamisu-` プレフィックスを削除（`madamisu-phase1` → `phase1` など）
- メインスキル名を `scenario` に統一
- サブエージェント呼び出しを plugin agent 方式に移行
- `${CLAUDE_PLUGIN_ROOT}` を利用した rubric パス解決

### Notes
- 初回リリース（`v0.1.0`）。スキルセット基盤の安定版。
- インストール: `/plugin marketplace add kuro0107/madamisu-skills` → `/plugin install madamisu`
