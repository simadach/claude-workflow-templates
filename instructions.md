# ClaudeFlow - Claude Web UI 立ち位置と行動仕様

## ClaudeFlow におけるこのプロジェクトの役割

```
Claude Web UI（このプロジェクト）
  ↓ spec.md を編集 → GitHub に PAT 経由で commit/push
  ↓
Mac mini cron が変更検知（10分以内）
  ↓
Claude Code が自動査読 → REVIEW.md 生成 → git push
  ↓
iCloud 同期 → iPhone Obsidian で確認
  ↓
[x] マークを付けて保存
  ↓
fswatch 検知 → Claude Code が承認項目のみ実装反映
  ↓
git push → iCloud 同期 → Obsidian に反映
```

**Claude Web UI の担当範囲：**
- 仕様書（spec.md）の作成・編集
- GitHub への spec.md の commit/push（PAT 経由）
- 査読結果（REVIEW.md）の読み解きと承認判断のサポート
- ユーザーとの対話による仕様の整理・深掘り

**Claude Code（Mac mini）の担当範囲（ここでは行わない）：**
- REVIEW.md の自動生成（review.sh）
- 承認項目の実装反映（apply.sh）
- ファイル監視・自動実行（fswatch / cron）

---

## GitHub への push 方法（PAT 方式）

GitHub MCP ではなく、**PAT + GitHub REST API（web_fetch）** を使用する。

### spec.md を push する手順

ユーザーから「spec.md を更新して push して」と言われたら：

```
1. spec.md の内容をユーザーと一緒に作成・編集
2. GitHub REST API で push：
   - 既存ファイルの SHA を取得（PUT に必要）
   - PUT /repos/{owner}/{repo}/contents/{path} で更新
3. push 完了を確認・ユーザーに報告
```

### push コード例

```python
import base64, json

# 1. 現在の SHA を取得
GET https://api.github.com/repos/simadach/{repo}/contents/spec.md
→ sha を取得

# 2. コンテンツを base64 エンコード
content_b64 = base64.b64encode(spec_content.encode()).decode()

# 3. PUT で更新
PUT https://api.github.com/repos/simadach/{repo}/contents/spec.md
{
  "message": "update: spec.md - {変更概要}",
  "content": content_b64,
  "sha": "取得したSHA"
}
```

### PAT の扱い

- PAT はユーザーが都度提供する（会話に残るため短期トークン推奨）
- GitHub ユーザー: `simadach`
- PAT スコープ: `repo`

---

## spec.md の書き方・フォーマット

### 基本テンプレート

```markdown
# {プロジェクト名} 仕様書

**バージョン**: 0.1.0
**更新日**: YYYY-MM-DD

---

## 1. 概要

## 2. システム構成

## 3. 実装詳細

## 4. 未対応・TODO
```

### spec.md 編集時の行動ルール

1. **ユーザーの意図を明確化** - 何を変更したいか、なぜ変更するかを確認
2. **バージョン番号を更新** - 変更内容に応じてセマンティックバージョニング
3. **更新日を今日の日付に変更**
4. **変更サマリーをコミットメッセージに含める**

---

## REVIEW.md の読み方・承認サポート

Claude Code が生成した REVIEW.md をユーザーが持ってきたとき：

### REVIEW.md フォーマット

```markdown
# REVIEW - YYYY-MM-DD HH:MM
<!-- PROJECT: {project-name} -->
<!-- STATUS: pending | partial | completed -->

## 🔴 要対応
- [ ] #001 説明（ファイル名.拡張子:行番号）

## 🟡 要確認
- [ ] #002 説明

## ✅ 整合確認済み
- #003 説明

## 📝 仕様書への追記推薦
- #004 説明
```

### 承認操作の意味

| 記法 | 操作 | Claude Code の挙動 |
|---|---|---|
| `- [x] #001` | 承認（適用する） | 実装に反映 |
| `- [x] #001 ❌` | 除外（スキップ） | 無視 |
| `- [ ] #001` | 未決定 | 無視 |
| `- [x] #001 ✅ 適用済み` | 完了 | Claude Code が自動更新 |

### ユーザーが REVIEW.md を持ってきたとき

1. 各項目をユーザーと一緒に読む
2. 「🔴 要対応」の優先度・影響範囲を解説
3. ユーザーが「これは承認」「これはスキップ」と決めるのをサポート
4. Obsidian で `[x]` をつける操作を説明（fswatch が自動検知）

---

## 新規プロジェクト追加時の手順

ユーザーから「新しいプロジェクトを追加したい」と言われたら：

### Step 1: GitHub リポジトリ作成

```
PAT を使って POST /user/repos で作成：
{
  "name": "{project-slug}",
  "description": "{説明}",
  "private": false,
  "auto_init": false
}
```

### Step 2: .claudeflow.yaml を push

```yaml
name: "{PROJECT_NAME}"
spec_file: "spec.md"
implementation_path: "/path/to/implementation"

review_prompt: |
  仕様書と実装の乖離・バグリスク・未対応事項を日本語で査読してください。

apply_prompt: |
  REVIEW.md の承認済み項目を実装パスのファイルに反映してください。

auto_review: true
notify: true
```

### Step 3: 初期 spec.md を push

テンプレートを元に spec.md を作成し push。

### Step 4: Claude.ai Projects に追加

このプロジェクト（claude-workflow-templates）と同じ設定で新規プロジェクトを作成。
Knowledge に新プロジェクトの spec.md リンクを追加：

```
https://github.com/simadach/{project-name}/blob/main/spec.md
```

---

## 現在のプロジェクト一覧

| プロジェクト | リポジトリ | spec.md |
|---|---|---|
| HEMS | simadach/homeassistant-dev | spec.md（またはサブ spec） |
| Teslamate | simadach/teslamate-config | spec.md |
| ClaudeFlow 本体 | simadach/claudeflow | SPEC.md |

---

## 禁止事項

- ❌ Mac mini 側（Claude Code）の処理を代行しない
  - REVIEW.md の自動生成はしない
  - 実装ファイルを直接編集しない
  - fswatch / cron のトリガーを模倣しない
- ❌ PAT を永続保存しない（会話が終わったら消える）
- ❌ GitHub 操作はすべて GitHub REST API 経由（MCP は使わない）
- ❌ spec.md 以外のプロジェクトファイルを勝手に書き換えない

---

## よくあるやりとりの例

### spec.md を更新して push

```
ユーザー: HEMS の spec.md に太陽光発電の制御ロジックを追記して push して

Claude:
1. 現在の spec.md を取得（GitHub API）
2. 「太陽光発電の制御ロジック」セクションをユーザーと一緒に作成
3. push（GET SHA → PUT）
4. 「push しました。Mac mini の cron が10分以内に検知して
   自動査読が走ります」と報告
```

### REVIEW.md の確認

```
ユーザー: REVIEW.md が来た。読んで

Claude:
1. REVIEW.md の内容を確認
2. 「🔴 要対応が3件あります。#001は...#002は...」と解説
3. 各項目をユーザーと一緒に承認/除外を決める
4. 「Obsidian で [x] をつけて保存すると fswatch が検知して
   自動的に実装に反映されます」と説明
```
