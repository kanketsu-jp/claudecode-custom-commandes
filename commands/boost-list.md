---
description: 実行中のWorktreeを簡易表示
allowed-tools:
  - Bash
  - Read
model: claude-sonnet-4-20250514
---

# 📋 Boost List - Worktree一覧

実行中のWorktreeを簡潔に一覧表示します。

## 📖 使い方

```bash
/boost-list
```

## 🛠️ 実行手順

以下のシェルスクリプトを実行してください：

```bash
#!/bin/bash

BOOST_DIR=".boost/worktrees"

# ディレクトリが存在しない場合
if [ ! -d "$BOOST_DIR" ]; then
    echo "📋 実行中のWorktree: なし"
    echo ""
    echo "/boost \"タスク説明\" で新しいタスクを開始できます。"
    exit 0
fi

# JSONファイルを取得
JSON_FILES=("$BOOST_DIR"/*.json)

# ファイルが存在しない場合
if [ ! -f "${JSON_FILES[0]}" ]; then
    echo "📋 実行中のWorktree: なし"
    echo ""
    echo "/boost \"タスク説明\" で新しいタスクを開始できます。"
    exit 0
fi

echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo "📋 実行中のWorktree"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo ""

# 各Worktreeの情報を表示
for JSON_FILE in "${JSON_FILES[@]}"; do
    if [ -f "$JSON_FILE" ]; then
        # JSONから情報を抽出（jqがない場合はgrepで代用）
        if command -v jq &> /dev/null; then
            INDEX=$(jq -r '.index' "$JSON_FILE")
            TASK=$(jq -r '.task' "$JSON_FILE")
            DIR=$(jq -r '.directory' "$JSON_FILE")
            PORT=$(jq -r '.port' "$JSON_FILE")
            STATUS=$(jq -r '.status' "$JSON_FILE")
        else
            INDEX=$(grep -oP '"index":\s*\K\d+' "$JSON_FILE")
            TASK=$(grep -oP '"task":\s*"\K[^"]+' "$JSON_FILE")
            DIR=$(grep -oP '"directory":\s*"\K[^"]+' "$JSON_FILE")
            PORT=$(grep -oP '"port":\s*\K\d+' "$JSON_FILE")
            STATUS=$(grep -oP '"status":\s*"\K[^"]+' "$JSON_FILE")
        fi

        # ステータスアイコン
        if [ "$STATUS" == "completed" ]; then
            ICON="✅"
        else
            ICON="🔵"
        fi

        echo "  ${ICON} #${INDEX}: ${TASK}"
        echo "     → cd ${DIR}"
        echo "     → ポート: ${PORT}"
        echo ""
    fi
done

echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo "💡 Tips:"
echo "  - /boost \"タスク\" で新規タスク開始"
echo "  - /boost-clean で完了したWorktreeをクリーンアップ"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
```

## 💡 Tips

- **jqなし対応**: jqコマンドがない環境でも動作します（grepで代用）
- **ステータス**: 🔵=実行中、✅=完了
- **ディレクトリ移動**: 表示されたcdコマンドをコピペして各Worktreeに移動

## ⚠️ 注意事項

- 完了したWorktreeは `/boost-clean` で削除してください
- 定期的に実行して、不要なWorktreeがないか確認しましょう
