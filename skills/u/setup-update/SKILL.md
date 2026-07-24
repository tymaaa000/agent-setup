---
name: setup-update
description: "Manage pi setup repos — pull upstream updates or push local changes to forks. Use when user says setup-update, 检查更新, 推送更新, sync setup. Two modes — local (pull upstream to sync runtime) and origin (push local to fork)."
---

# Setup Update

Two modes, selected by the first argument:

- `local` — pull upstream updates, sync to runtime
- `origin` — push local changes to GitHub forks

## Path Resolution

This skill file is at `{PI_HOME}/.pi/agent/skills/u/setup-update/SKILL.md`.
Derive `PI_HOME` by going up 4 levels from this file's directory.

| Variable | Value |
|----------|-------|
| `PI_HOME` | derived from skill location |
| `PI_SETUP` | `{PI_HOME}/pi-setup` |
| `AGENT_SETUP` | `{PI_HOME}/agent-setup` |
| `RUNTIME` | `{PI_HOME}/.pi/agent` |
| `BRANCH` | auto-detect: `git rev-parse --abbrev-ref HEAD` |

Remotes per repo: `origin` = user's fork, `upstream` = aqua2k1 original.

---

## MODE: local

Pull upstream updates → sync to runtime.

### Step L1: Pull Fork First

```bash
for repo in "{PI_SETUP}" "{AGENT_SETUP}"; do
  cd "$repo"
  BRANCH=$(git rev-parse --abbrev-ref HEAD)
  git pull origin $BRANCH || echo "⚠ pull 失败: $repo"
done
```

### Step L2: Fetch and Compare

```bash
cd "{PI_SETUP}" && git fetch upstream 2>/dev/null || echo "⚠ pi-setup upstream 不可达"
cd "{AGENT_SETUP}" && git fetch upstream 2>/dev/null || echo "⚠ agent-setup upstream 不可达"
```

Show three-way comparison per repo:

```bash
echo "本地: $(git rev-parse --short HEAD)"
echo "fork: $(git rev-parse --short origin/$BRANCH)"
echo "上游: $(git rev-parse --short upstream/$BRANCH 2>/dev/null || echo 'N/A')"
```

List new upstream commits:

```bash
git log --oneline origin/$BRANCH..upstream/$BRANCH 2>/dev/null || echo "(无)"
```

### Step L3: Report — STOP HERE

**CRITICAL: Do NOT proceed without user confirmation.**

Show which repos have updates, every new commit, and files changed (+/-).

**Q1: 是否合并上游更新？**

Dynamic based on which repos have updates:
- Both → `A) 全部合并  B) 仅 pi-setup  C) 仅 agent-setup  D) 都不合并`
- One → `A) 合并  B) 不合并`
- Neither → "已是最新" and STOP

**Q2: 合并后是否同步到运行时？**

```
2. 合并后是否同步到运行时(.pi/agent)？
   A) 是
   B) 否
```

> 💡 空格分隔回答，如 `A A`。跳过用 `-`。

Parse: split by spaces, map by position. `-` = skip. Too few → ask remaining.

### Step L4: Merge Upstream

```bash
cd {repo}
BRANCH=$(git rev-parse --abbrev-ref HEAD)
git merge upstream/$BRANCH || { echo "❌ 合并冲突，请手动解决"; exit 1; }
```

### Step L5: Sync to Runtime

Copy from repos to `{RUNTIME}`, with protected files:

**PROTECTED (never overwrite):**
`settings.json` `pi-websearch.json` `auth.json` `trust.json` `models-store.json` `sessions/` `npm/` `git/` `bin/`

**pi-setup → .pi/agent:**
Copy all dirs/files under `pi-setup/agent/` except protected list. Overwrite existing.

**agent-setup → .pi/agent/skills:**
```bash
cp -r {AGENT_SETUP}/skills/ {RUNTIME}/skills/
```

**Cleanup:** Remove files in runtime not present in source repos.

If upstream `settings.json` has new packages, show diff and ask separately.

### Step L6: Verify

```bash
echo "=== Git 状态 ==="
cd {PI_SETUP} && echo "pi-setup: $(git rev-parse --short HEAD)"
cd {AGENT_SETUP} && echo "agent-setup: $(git rev-parse --short HEAD)"
echo "=== 运行时关键目录 ==="
ls {RUNTIME}/extensions/ 2>/dev/null | head -5
ls {RUNTIME}/agents/ 2>/dev/null | head -5
ls {RUNTIME}/skills/u/ 2>/dev/null | head -5
echo "=== 受保护文件 ==="
for f in settings.json pi-websearch.json auth.json; do
  [ -f "{RUNTIME}/$f" ] && echo "✅ $f" || echo "❌ $f 缺失"
done
```

### Step L7: Report

Summarize merged/synced/skipped. Tell user to restart pi.

---

## MODE: origin

Push local changes to GitHub forks.

### Step O1: Check Local Changes

```bash
for repo in "{PI_SETUP}" "{AGENT_SETUP}"; do
  cd "$repo"
  BRANCH=$(git rev-parse --abbrev-ref HEAD)
  echo "=== $(basename $repo) ==="
  git status --short || echo "❌ git status 失败"
  git log --oneline origin/$BRANCH..HEAD 2>/dev/null || true
done
```

### Step O2: Report — STOP HERE

**CRITICAL: Do NOT proceed without user confirmation.**

**Q1: 上传哪些文件？**

List EVERY changed file with function summary:
- SKILL.md → read `description` frontmatter
- Other → read first header or purpose line
- Format: `B) [repo] file/path — 功能描述`
- >10 files → group by directory
- First option always `A) 全部`

Zero changes → "无改动" and STOP.

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

> 💡 多选用逗号分隔（如 `B,C`），题间用空格（如 `B,C A A`）。跳过用 `-`。

Parse: split by spaces. Q1 split by comma (`A`=all, `B,C`=specific). Q2 `A`=auto `B`=custom→ask message. Q3 `A`=yes.

### Step O3: Stage and Commit

```bash
cd {repo}
git add {user-selected files}
git commit -m "{message}" || { echo "❌ 提交失败"; exit 1; }
```

### Step O4: Push

```bash
cd {repo}
BRANCH=$(git rev-parse --abbrev-ref HEAD)
git push origin $BRANCH || { echo "❌ 推送失败，检查网络或权限"; exit 1; }
```

### Step O5: Verify

```bash
cd {repo}
LOCAL=$(git rev-parse --short HEAD)
REMOTE=$(git ls-remote origin $BRANCH | cut -c1-7)
[ "$LOCAL" = "$REMOTE" ] && echo "✅ 推送成功" || echo "⚠ $LOCAL ≠ $REMOTE"
```

### Step O6: Report

Summarize. Remind about PR to upstream (aqua2k1) via GitHub.
