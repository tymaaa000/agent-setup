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
- Every changed file listed line by line
- Every unpushed commit with its message

Then ask ALL three questions and wait for answers:
1. "哪些文件需要上传？" — let user pick specific files or all
2. "提交信息用什么？" — let user write their own message
3. "确认推送吗？" — final go/no-go

Only after the user explicitly says yes, move to Step 3.

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
