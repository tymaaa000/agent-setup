---
name: setup-update-local
description: Check two pi config repos (pi-setup and agent-setup) for upstream updates, show what changed, sync to .pi/agent/ runtime directory, and verify. Use when user says "检查更新", "update pi", "sync setup", "setup-update-local", or wants to check if their pi setup repos are behind upstream.
---

# Setup Update Skill

## Repo Layout

The user has two config repos under `D:/Program Files/piagent/`:

| Repo | Git path | Runtime target |
|------|----------|---------------|
| pi-setup | `D:/Program Files/piagent/pi-setup/` | `D:/Program Files/piagent/.pi/agent/` |
| agent-setup | `D:/Program Files/piagent/agent-setup/` | `D:/Program Files/piagent/.pi/agent/skills/` |

Each repo has:
- `origin`: the user's fork on GitHub (`git@github.com:tymaaa000/<name>.git`)
- `upstream`: the original repo (`git@github.com:aqua2k1/<name>.git`)

## Step 1: Fetch and Compare

For each repo, run in its directory:

```bash
git fetch origin && git fetch upstream
```

Then show the three-way comparison:

```bash
echo "本地: $(git rev-parse --short HEAD)"
echo "fork: $(git rev-parse --short origin/main)"
echo "上游: $(git rev-parse --short upstream/main)"
```

And list commits that exist upstream but not in the local fork:

```bash
git log --oneline origin/main..upstream/main
```

## Step 2: If Local is Behind Fork

Pull from fork first:

```bash
git pull origin main
```

## Step 3: Report to User — STOP HERE

**CRITICAL: Do NOT proceed beyond this step without explicit user confirmation.**

Present a clear summary:
- Which repo has upstream updates (pi-setup / agent-setup / both)
- Every new commit listed with hash and Chinese description
- Every file that will be changed, with `+` for new and `-` for removed

Then ask ALL questions and wait for answers:
1. "是否合并这些更新？" — user may say yes to all, or pick specific commits
2. "确认同步到 .pi/agent/？" — final go/no-go before touching runtime files

Only after the user explicitly says yes, move to Step 4.

## Step 4: Sync to .pi/agent (runtime)

After user confirms, copy files from the repo to the `.pi/agent/` runtime directory.

### pi-setup sync rules:

Copy these directories completely (overwrite):
- `pi-setup/agent/extensions/` → `.pi/agent/extensions/`
- `pi-setup/agent/agents/` → `.pi/agent/agents/`
- `pi-setup/agent/prompts/` → `.pi/agent/prompts/`
- `pi-setup/agent/APPEND_SYSTEM.md` → `.pi/agent/APPEND_SYSTEM.md`

**NEVER overwrite these user-custom files:**
- `.pi/agent/settings.json` — user has custom provider/model/packages
- `.pi/agent/pi-websearch.json` — user has custom web search config
- `.pi/agent/auth.json` — user credentials
- `.pi/agent/trust.json` — trust settings

If `settings.json` has new packages in the pi-setup version, show the diff and let the user decide.

### agent-setup sync rules:

Copy the skills directory:
- `agent-setup/skills/` → `.pi/agent/skills/` (overwrite)

### After syncing:

```bash
# Clean up any files that were removed upstream
# For pi-setup: remove files in .pi/agent/{extensions,agents,prompts} that don't exist in pi-setup counterpart
diff -rq "D:/Program Files/piagent/pi-setup/agent/agents/" "D:/Program Files/piagent/.pi/agent/agents/" 2>&1 | grep "Only in $DST" || true
```

## Step 5: Verify

Check that key files exist and are accessible:

```bash
ls -la "D:/Program Files/piagent/.pi/agent/extensions/open.ts"
grep "ppt-master" "D:/Program Files/piagent/.pi/agent/settings.json" || echo "(未启用 ppt-master)"
```

## Step 6: Report

Summarize what was synced, what was skipped, and tell the user to restart pi.
