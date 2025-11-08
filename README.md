# Claude Code Custom Commands

カスタムの Claude Code コマンド集です。生産性を向上させる便利なコマンドを提供します。

## 📦 インストール方法

### 方法1: プラグインマーケットプレイスから（推奨）

Claude Code でプラグインとしてインストールする最も簡単な方法です：

```bash
# マーケットプレイスを追加
/plugin marketplace add kanketsu-jp/claudecode-custom-commandes

# プラグインをインストール
/plugin install boost-commands@kanketsu-jp/claudecode-custom-commandes
```

インストール後、すぐに以下のコマンドが使用可能になります：
- `/boost`
- `/boost-list`
- `/boost-clean`

### 方法2: 手動インストール

1. このリポジトリをダウンロード：

```bash
git clone https://github.com/kanketsu-jp/claudecode-custom-commandes.git
cd claudecode-custom-commandes
```

2. コマンドをユーザーの `.claude` ディレクトリにコピー：

```bash
# ホームディレクトリの .claude/commands にコピー
cp -r commands/* ~/.claude/commands/
```

3. Claude Code を再起動すると、コマンドが利用可能になります

### 方法3: プロジェクト単位でインストール

特定のプロジェクトでのみ使用したい場合：

```bash
# プロジェクトルートで実行
mkdir -p .claude/commands
cp -r /path/to/claudecode-custom-commandes/commands/* .claude/commands/
```

### 確認方法

Claude Code で以下のコマンドが使えることを確認：

```bash
/boost "テスト"
/boost-list
/boost-clean
```

## 📚 利用可能なコマンド

### 🚀 Boost コマンド

Git Worktree を使用して複数のタスクを並列実行するコマンドグループです。

#### `/boost` - タスク並列実行

複数のタスクを独立した環境で同時実行します。

**使用例：**
```bash
/boost "ログイン機能を実装"
/boost "フロントエンド開発" "バックエンド開発"
```

**機能：**
- Git Worktree で独立した作業環境を自動作成
- 各タスクに自動でポート番号を割り当て
- 環境変数ファイル（.env.boost）を自動生成

#### `/boost-list` - Worktree一覧表示

実行中のWorktreeを一覧表示します。

**使用例：**
```bash
/boost-list
```

**表示内容：**
- タスク番号と説明
- ディレクトリパス
- 割り当てられたポート番号
- ステータス（実行中/完了）

#### `/boost-clean` - クリーンアップ

完了したWorktreeを削除します。

**使用例：**
```bash
/boost-clean           # 完了したWorktreeのみ削除
/boost-clean --all     # すべてのWorktreeを強制削除
```

**注意：**
- 重要な変更は事前にコミット推奨
- mainブランチへのマージが必要な場合は事前に実行

## 🛠️ 開発者向け情報

### プロジェクト構造

```
claudecode-custom-commands/
├── .claude-plugin/
│   └── plugin.json          # プラグインメタデータ
├── commands/                # コマンド格納ディレクトリ
│   ├── boost.md            # /boost コマンド
│   ├── boost-list.md       # /boost-list コマンド
│   └── boost-clean.md      # /boost-clean コマンド
├── .gitignore
└── README.md
```

### 新しいコマンドの追加

1. `commands/` ディレクトリにコマンドファイルを作成（`.md` 形式）
2. frontmatter を追加：

```markdown
---
description: コマンドの説明
argument-hint: [引数のヒント]
allowed-tools:
  - Bash
  - Read
  - Write
model: claude-sonnet-4-20250514
---

# コマンドの内容
```

3. `.claude-plugin/plugin.json` に追加：

```json
{
  "commands": [
    "./commands/your-command.md"
  ]
}
```

**コマンド呼び出し方法:**
- `commands/mycommand.md` → `/mycommand` として呼び出し可能
- `commands/my-command.md` → `/my-command` として呼び出し可能

## 🤝 コントリビューション

コントリビューションを歓迎します！

### セキュリティガイドライン

このリポジトリは公開リポジトリです。以下を厳守してください：

- ❌ 個人情報（ユーザー名、パスなど）を含めない
- ❌ 特定プロジェクト構造への依存を避ける
- ✅ すべてのパスは相対パス表記
- ✅ 汎用的で再利用可能なコマンドのみ

### プルリクエスト手順

1. フォーク
2. 新しいブランチを作成（`git checkout -b feature/amazing-command`）
3. 変更をコミット（`git commit -m 'Add amazing command'`）
4. ブランチにプッシュ（`git push origin feature/amazing-command`）
5. プルリクエストを作成

## 📄 ライセンス

MIT License

## 📞 サポート

問題や質問がある場合は、[Issues](https://github.com/kanketsu-jp/claudecode-custom-commandes/issues) を作成してください。

---


# おすすめのMCP for Claude code
確認: `claude mcp list`

## [NextJS](https://github.com/vercel/next-devtools-mcp)
```
claude mcp add next-devtools npx next-devtools-mcp@latest
```

## [chrome-devtools-mcp](https://github.com/ChromeDevTools/chrome-devtools-mcp)
```
claude mcp add chrome-devtools npx chrome-devtools-mcp@latest
```

## [playwright](https://github.com/microsoft/playwright-mcp)
```
claude mcp add playwright npx @playwright/mcp@latest
```

## Gemin CLI
```
claude mcp add gemini-cli -- npx -y gemini-mcp-tool
```

## [figma-remote-server](https://developers.figma.com/docs/figma-mcp-server/remote-server-installation/#claude-code)
```
claude mcp add --transport http figma-remote-mcp
```

## [excel](https://github.com/haris-musa/excel-mcp-server)
```
claude mcp add-json excel-server '{"name":"excel-server","command":"uvx","args":["excel-mcp-server","stdio"]}'
```

> Excel MCP Serverのグローバルインストール
> `npm install -g excel-mcp-server`

## [Grep](https://vercel.com/blog/grep-a-million-github-repositories-via-mcp)
```
claude mcp add --transport http grep https://mcp.grep.app
```

## Context7
```
claude mcp add --transport sse context7 https://mcp.context7.com/sse
```

# Tips

## Selenaでブラウザに飛ばされるのを止める
```
vim ~/.serena/serena_config.yml
```

```diff
- web_dashboard: true
+ web_dashboard: false
```
