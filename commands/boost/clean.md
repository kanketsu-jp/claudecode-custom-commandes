---
description: 完了したWorktreeをクリーンアップ
argument-hint: [--all]
allowed-tools:
  - Bash
  - Read
model: claude-sonnet-4-20250514
---

# 🧹 Boost Clean - クリーンアップ

完了したWorktree、または全Worktreeをクリーンアップします。

## 📖 使い方

```bash
/boost-clean           # 完了したWorktreeのみ削除
/boost-clean --all     # 全Worktreeを強制削除
```

## 🛠️ 実行手順

以下のシェルスクリプトを実行してください：

```bash
#!/bin/bash
set -e

BOOST_DIR=".boost/worktrees"

# ディレクトリが存在しない場合
if [ ! -d "$BOOST_DIR" ]; then
    echo "✨ クリーンアップ対象はありません"
    exit 0
fi

# 引数チェック
CLEAN_ALL=false
if [[ "$ARGUMENTS" == *"--all"* ]]; then
    CLEAN_ALL=true
fi

echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
if [ "$CLEAN_ALL" = true ]; then
    echo "🧹 全Worktreeをクリーンアップ"
else
    echo "🧹 完了したWorktreeをクリーンアップ"
fi
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo ""

CLEANED_COUNT=0

# 各Worktreeを処理
for JSON_FILE in "$BOOST_DIR"/*.json; do
    if [ ! -f "$JSON_FILE" ]; then
        continue
    fi

    # JSONから情報を抽出
    if command -v jq &> /dev/null; then
        INDEX=$(jq -r '.index' "$JSON_FILE")
        TASK=$(jq -r '.task' "$JSON_FILE")
        DIR=$(jq -r '.directory' "$JSON_FILE")
        BRANCH=$(jq -r '.branch' "$JSON_FILE")
        STATUS=$(jq -r '.status' "$JSON_FILE")
    else
        INDEX=$(grep -oP '"index":\s*\K\d+' "$JSON_FILE")
        TASK=$(grep -oP '"task":\s*"\K[^"]+' "$JSON_FILE")
        DIR=$(grep -oP '"directory":\s*"\K[^"]+' "$JSON_FILE")
        BRANCH=$(grep -oP '"branch":\s*"\K[^"]+' "$JSON_FILE")
        STATUS=$(grep -oP '"status":\s*"\K[^"]+' "$JSON_FILE")
    fi

    # クリーンアップ対象かチェック
    SHOULD_CLEAN=false
    if [ "$CLEAN_ALL" = true ]; then
        SHOULD_CLEAN=true
    elif [ "$STATUS" = "completed" ]; then
        SHOULD_CLEAN=true
    fi

    if [ "$SHOULD_CLEAN" = true ]; then
        echo "🗑️  #${INDEX}: ${TASK}"

        # Worktreeディレクトリを削除
        if [ -d "$DIR" ]; then
            git worktree remove "$DIR" --force 2>/dev/null || rm -rf "$DIR"
            echo "   ✅ Worktree削除: ${DIR}"
        fi

        # ブランチを削除
        if git show-ref --verify --quiet "refs/heads/${BRANCH}"; then
            git branch -D "$BRANCH" 2>/dev/null
            echo "   ✅ ブランチ削除: ${BRANCH}"
        fi

        # メタデータファイルを削除
        rm -f "$JSON_FILE"
        echo "   ✅ メタデータ削除"
        echo ""

        CLEANED_COUNT=$((CLEANED_COUNT + 1))
    fi
done

# Git Worktree情報をクリーンアップ
git worktree prune 2>/dev/null || true

echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
if [ $CLEANED_COUNT -eq 0 ]; then
    echo "✨ クリーンアップ対象はありませんでした"
else
    echo "✅ ${CLEANED_COUNT}個のWorktreeをクリーンアップしました"
fi
echo ""
echo "💡 Tips:"
echo "  - /boost-list で残りのWorktreeを確認"
echo "  - /boost \"タスク\" で新しいタスクを開始"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
```

## 💡 Tips

- **完了マーク**: Worktree内で作業が完了したら、JSONファイルの`status`を`completed`に変更
- **強制削除**: `--all` オプションで全Worktreeを一括削除
- **安全性**: 重要な変更は必ずコミットしてからクリーンアップ

## ⚠️ 注意事項

- 削除前に重要な変更はコミットしておいてください
- `--all` オプションは全てのWorktreeを削除します（元に戻せません）
- mainブランチへのマージが必要な場合は、クリーンアップ前に実行してください

## 📝 ステータスの変更方法

Worktreeの作業が完了したら、ステータスを変更します：

```bash
# .boost/worktrees/worktree-1.json を編集
# "status": "running" → "status": "completed"
```

または、以下のコマンドで一括変更：

```bash
sed -i '' 's/"status": "running"/"status": "completed"/' .boost/worktrees/worktree-1.json
```
