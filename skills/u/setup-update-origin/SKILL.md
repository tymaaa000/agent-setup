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
- Every changed file listed in a table with repo label
- Every unpushed commit with its message

### Dynamic Question Construction

**Q1: 上传哪些文件？**

List EVERY changed file as an option with its function description:
- Read each file and extract a one-line Chinese summary of what it does
- For SKILL.md: read the `description` field from frontmatter
- For other files: read first meaningful comment or header line
- Format: `B) [repo] file/path — 功能描述`

Plus "A) 全部" at the top.

If zero changed files → report "无改动" and STOP.

**Q2: 提交信息用什么？**

```
2. 提交信息用什么？
   A) [自动生成: feat(xxx): xxx]
   B) 自定义
```

**Q3: 确认推送？**

```
3. 确认推送？
   A) 是
   B) 否
```

### Answer Format Hint — ALWAYS SHOW THIS

**MUST append this hint after EVERY question block, no exceptions:**

> 💡 多选用逗号分隔（如 `B,C`），题间用空格（如 `B,C A A`）。跳过用 `-`。

### Parse Rules

- Split by spaces to get per-question answers
- Q1: split by comma to get file selection. `A` = all, `B,C` = specific files, `-` = skip
- Q2: single letter, A=auto B=custom. If B, ask for message separately
- Q3: single letter, A=yes B=no
- If too few answers, ask for remaining questions only

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
