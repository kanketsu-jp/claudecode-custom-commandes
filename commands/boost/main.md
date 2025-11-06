---
description: タスクを並列実行（Git Worktreeで独立環境を作成）
argument-hint: "タスク1" "タスク2" "タスク3"
allowed-tools:
  - Bash
  - Read
  - Write
model: claude-sonnet-4-20250514
---

# 🚀 Boost - タスク並列実行

複数のタスクを独立したGit Worktree環境で並列実行します。

## 📖 使い方

```bash
/boost "タスク1の説明" "タスク2の説明" "タスク3の説明"
```

**例**:
```bash
/boost "ログイン機能を実装"
/boost "フロントエンド開発" "バックエンド開発"
```

## 🛠️ 実行手順

以下のシェルスクリプトを実行してください：

```bash
#!/bin/bash
set -e

# 設定
BASE_DIR="../boost-worktrees"
BASE_PORT=3000
BOOST_DIR=".boost/worktrees"

# ディレクトリ作成
mkdir -p "$BOOST_DIR" "$BASE_DIR"

# 引数チェック
if [ -z "$ARGUMENTS" ]; then
    echo "❌ エラー: タスクが指定されていません"
    echo "使い方: /boost \"タスク1\" \"タスク2\""
    exit 1
fi

# タスクを配列に変換（スペース区切り）
IFS=' ' read -ra TASKS <<< "$ARGUMENTS"

echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo "✨ Boost - タスク並列実行"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo ""

# 既存のWorktree数を取得
EXISTING_COUNT=$(ls -1 "$BOOST_DIR"/*.json 2>/dev/null | wc -l | tr -d ' ')

# 各タスクを処理
TASK_INDEX=1
for TASK in "${TASKS[@]}"; do
    # クォートを削除
    TASK=$(echo "$TASK" | sed 's/^"//;s/"$//')

    WORKTREE_INDEX=$((EXISTING_COUNT + TASK_INDEX))
    TIMESTAMP=$(date +%Y%m%d-%H%M%S)

    # タスク名をスラッグ化
    TASK_SLUG=$(echo "$TASK" | sed 's/[^a-zA-Z0-9]/-/g' | tr '[:upper:]' '[:lower:]' | cut -c1-30)

    # ブランチ名とディレクトリ
    BRANCH="boost/task-${WORKTREE_INDEX}-${TASK_SLUG}-${TIMESTAMP}"
    WORKTREE_DIR="$BASE_DIR/boost-${WORKTREE_INDEX}"
    PORT=$((BASE_PORT + (WORKTREE_INDEX * 10)))

    echo "📋 タスク ${TASK_INDEX}: ${TASK}"

    # Git Worktreeを作成
    git branch "$BRANCH" 2>/dev/null || true
    git worktree add "$WORKTREE_DIR" "$BRANCH"

    # 環境変数ファイルを作成
    cat > "$WORKTREE_DIR/.env.boost" << EOF
BOOST_WORKTREE_INDEX=${WORKTREE_INDEX}
BOOST_SERVICE_PORT=${PORT}
BOOST_TASK_DESC="${TASK}"
BOOST_BRANCH_NAME="${BRANCH}"
EOF

    # メタデータを保存
    cat > "$BOOST_DIR/worktree-${WORKTREE_INDEX}.json" << EOF
{
  "index": ${WORKTREE_INDEX},
  "task": "${TASK}",
  "branch": "${BRANCH}",
  "directory": "${WORKTREE_DIR}",
  "port": ${PORT},
  "created": "$(date -u +"%Y-%m-%dT%H:%M:%SZ")",
  "status": "running"
}
EOF

    echo "   ✅ Worktree作成: ${WORKTREE_DIR}"
    echo "   🔌 ポート: ${PORT}"
    echo ""

    TASK_INDEX=$((TASK_INDEX + 1))
done

echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo "✅ ${#TASKS[@]}個のWorktreeを作成しました"
echo ""
echo "📝 次のステップ:"
echo "  1. cd ${BASE_DIR}/boost-X で各Worktreeに移動"
echo "  2. source .env.boost で環境変数を読み込み"
echo "  3. /boost-list で一覧確認"
echo "  4. /boost-clean で完了したWorktreeをクリーンアップ"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
```

## 💡 Tips

- **環境変数**: 各Worktree内で `source .env.boost` を実行すると、タスク情報を取得できます
- **ポート番号**: 自動で割り当てられるので、複数サービスを同時起動可能
- **一覧確認**: `/boost-list` で実行中のWorktreeを確認
- **クリーンアップ**: `/boost-clean` で完了したWorktreeを削除

## ⚠️ 注意事項

- 作業完了後は必ず `/boost-clean` でクリーンアップしてください
- 重要な変更はWorktree内でコミットしてからクリーンアップ
- mainブランチへのマージが必要な場合は、クリーンアップ前に実行
