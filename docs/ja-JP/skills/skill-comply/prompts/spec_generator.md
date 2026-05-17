<!-- markdownlint-disable MD007 -->
コーディングエージェント（Claude Code）のスキル/ルールファイルを分析しています。
タスク：このスキルがアクティブな場合にエージェントが従うべき**観察可能な動作シーケンス**を抽出してください。

各ステップは自然言語で説明してください。正規表現パターンは使用しないでください。

以下の正確なフォーマットで有効なYAMLのみを出力してください（マークダウンフェンス、コメントなし）：

id: <kebab-case-id>
name: <人間が読めるの名前>
source_rule: <提供されたファイルパス>
version: "1.0"

steps:
  - id: <snake_case>
    description: <エージェントが何をすべきか>
    required: true|false
    detector:
      description: <探すべきツール呼び出しの自然言語による説明>
      after_step: <このステップの前に来るべきstep_id（オプション — 必要でなければ省略）>
      before_step: <このステップの後に来るべきstep_id（オプション — 必要でなければ省略）>

scoring:
  threshold_promote_to_hook: 0.6

ルール：
- detector.descriptionはパターンではなく、ツール呼び出しの意味を説明すること
  良い例：「テストファイルを書くまたは編集する（実装ファイルではない）」
  悪い例：「test.*\\.pyにマッチする入力でWriteまたはEdit」
- 順序が重要なスキルにはbefore_step/after_stepを使用する（例：TDD：実装の前にテスト）
- 存在のみが重要なスキルには順序制約を省略する
- 「オプションで」または「該当する場合」とスキルが言っている場合のみrequired: falseとする
- 3〜7ステップが理想的。細かく分解しすぎない
- 重要：コロンを含むすべてのYAML文字列値はダブルクォートで囲む
  良い例：description: "慣習的なコミットフォーマットを使用する（type: description）"
  悪い例：description: 慣習的なコミットフォーマットを使用する（type: description）

分析するスキルファイル：

---
{skill_content}
---
