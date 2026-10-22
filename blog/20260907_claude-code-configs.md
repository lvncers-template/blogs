# 


## TL;DR

- Cursor から claude code へ移行（Cursor で claude code を使っています）
- スラッシュコマンドからスキルへ完全移行（公式推奨）
- スキルは独立性を保つために別リポジトリでロックファイル管理
- Remote Control は良い！
- ほとんどの MCP を Plugins へ移行
- おまけ： claude code Routine は神

## Cursor から claude code へ移行（Cursor で claude code を使っています）

Cursorを辞めた理由はシンプルです。
コスパが悪い。
Proプランで使用していましたが、Opus君に重めのタスクを任せると、１リクエストで月の1/4くらい使っていました。
これはやっていけない。
乗り換えるとしたら、claude code か、codex か。
まあ、claude codeを使っておけば問題ないだろう。
そのぐらい軽い気持ちで移行しました。

Cursorから claude code を使っています。
少なくともターミナルで使うよりは、使いやすいです。
UIが見やすいと言うのはもちろん、ワンボタンでモデル切替、モード切替、参照ファイル選択、MCP選択などできます。
実行速度は目に見えて早いです。
ツール呼び出し、ファイル編集とかは特に早いです。

## まずは全体像を

![skills-and-configs-templates](/images/20260928_ai-positions/20260820_ai.pptx.png)

## スラッシュコマンドからスキルへ完全移行（公式推奨）

実は全然知らなかったのですが、スキルへ統合されていたみたいです。

https://code.claude.com/docs/en/skills

> For built-in commands like /help and /compact, and bundled skills like /debug and /code-review, see the commands reference.
> Custom commands have been merged into skills. A file at .claude/commands/deploy.md and a skill at .claude/skills/deploy/SKILL.md both create /deploy and work the same way. Your existing .claude/commands/ files keep working. Skills add optional features: a directory for supporting files, frontmatter to control whether you or Claude invokes them, and the ability for Claude to load them automatically when relevant.

スキルはデフォルトではモデルが会話の文脈から自動判断して呼び出すものです。

重要なのは、「Skills の frontmatter には disable-model-invocation を指定できる」ということです。

これを設定するとユーザーが `/<skill-name>` で明示的に呼び出したときのみ実行されます。
つまり、モデルが自律判断で起動することはないということです。

```
---
name: worktree-create-already-onremote
description: リモートにすでにあるブランチから worktree を作成する。作業ツリーの追加を伴うため、ユーザーが明示的に依頼したときに使う。
disable-model-invocation: true
---
```

私の方針としては、削除・上書きなど取り消しづらい・破壊的な操作を伴うコマンドは`disable-model-invocation: true` を明示します。

下の感じでまとめているんですが、数が増えてくると管理が大変になりそうです。
（🔒 = 手動実行限定）

```
skills/
├── git/
│   ├── branch/SKILL.md 🔒
│   ├── commit/SKILL.md 🔒
│   ├── git-sync/SKILL.md 🔒
│   └── submodule-remove/SKILL.md 🔒
├── github/
│   ├── issue-create/SKILL.md 🔒
│   ├── pr-create/SKILL.md 🔒
│   ├── pr-review/SKILL.md
│   └── pr-review-commit-driven/SKILL.md
├── slides/
│   ├── slide-draft/SKILL.md
│   └── slide-slidev/SKILL.md
├── testing/
│   ├── test-run/SKILL.md
│   └── test-write/SKILL.md
└── worktree/
    ├── setup-init-bare/SKILL.md (+ scripts/) 🔒
    ├── worktree-create-already-onremote/SKILL.md 🔒
    ├── worktree-create-new-branch/SKILL.md 🔒
    ├── worktree-list/SKILL.md
    └── worktree-remove/SKILL.md
```

## 

## スキルは独立性を保つために別リポジトリでロックファイル管理
