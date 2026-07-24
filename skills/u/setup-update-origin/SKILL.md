---
name: setup-update-origin
description: Push local changes in pi-setup and agent-setup repos to the user's GitHub forks. Use when user says "推送更新", "push to origin", "upload my changes", "setup-update-origin", or wants to save their customizations to their forks.
---

# Setup Update Origin Skill

Push local changes in the two config repos to the user's GitHub forks.

## Repo Layout

| Repo | Local path | Remote fork |
|------|-----------|-------------|
| pi-setup | `D:/Program Files/piagent/pi-setup/` | `git@github.com:tymaaa000/pi-setup.git` |
| agent-setup | `D:/Program Files/piagent/agent-setup/` | `git@github.com:tymaaa000/agent-setup.git` |

## Step 1: Check Local Changes

For each repo, check if there's anything to commit or push:

```bash
cd <repo_path>
git status --short
git log --oneline origin/main..HEAD
```

## Step 2: Report to User — STOP HERE

**CRITICAL: Do NOT proceed beyond this step without explicit user confirmation.**

Present a clear summary:
- Which repo has changes (pi-setup / agent-setup / both)
- Every changed file listed in a table
- Every unpushed commit with its message

### Question Construction Rules

**DYNAMIC OPTIONS — never show dead options.**

**Q1: 上传哪些文件？**
- Both repos have changes → A) 全部  B) 仅 pi-setup  C) 仅 agent-setup
- Only pi-setup has changes → A) 是（仅 pi-setup）  B) 否
- Only agent-setup has changes → A) 是（仅 agent-setup）  B) 否
- Neither has changes → report "无改动" and STOP, do not ask questions

**Q2: 提交信息用什么？**
- Always: A) 自动生成（写清具体内容，如 "feat: add xxx"）  B) 自定义

**Q3: 确认推送？**
- Always: A) 是  B) 否

### Answer Format

At the end of the questions, always append this hint:

> 💡 按顺序空格分隔回答，如 `A A A`。跳过用 `-`。

Parse rules:
- Split by spaces, map by position: first → Q1, second → Q2, third → Q3
- `-` means skip
- Fewer answers than questions → ask remaining
- Q2=B → ask commit message separately

## Step 3: Stage and Commit

If there are unstaged changes that the user approved:

```bash
cd <repo_path>
git add -A
git commit -m "<user's message>"
```

## Step 4: Push

```bash
cd <repo_path>
git push origin main
```

## Step 5: Verify

Confirm push succeeded by checking remote:

```bash
git ls-remote --heads origin main
```

Compare with local HEAD to confirm they match.

## Step 6: Report

Summarize what was pushed to which fork. Remind the user that if they want to contribute these changes back to upstream (aqua2k1), they can open a Pull Request on GitHub.
