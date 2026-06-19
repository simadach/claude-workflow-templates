# claude-workflow-templates

Claude.ai Projects（Claude Web UI）用の共有ワークフロー設定。
ClaudeFlow パイプラインにおける Claude Web UI の役割・行動を定義する。

## ファイル構成

| ファイル | 役割 |
|---|---|
| `instructions.md` | Claude Web UI の行動仕様（役割・GitHub push 手順・禁止事項） |
| `review-format.md` | REVIEW.md のフォーマット定義と査読観点 |
| `apply-format.md` | 承認項目の適用ルール（apply.sh の動作仕様） |

## ClaudeFlow パイプライン

```
Claude Web UI（このプロジェクト）
  ↓ spec / sub_id/spec.md を GitHub に push
Mac mini cron → Claude Code が自動査読 → REVIEW.md 生成
  ↓ iCloud 同期
iPhone Obsidian で確認 → [x] マークを付けて保存
  ↓ fswatch 検知
Claude Code が承認項目のみ実装反映 → git push
```

## 関連リポジトリ

| リポジトリ | 役割 |
|---|---|
| [simadach/claudeflow](https://github.com/simadach/claudeflow) | フレームワーク本体（スクリプト・SPEC） |
| [simadach/homeassistant-dev](https://github.com/simadach/homeassistant-dev) | HEMS プロジェクト（運用例） |
