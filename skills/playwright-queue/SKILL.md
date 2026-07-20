---
name: playwright-queue
description: Cooperative queue discipline for the shared Playwright MCP Chrome profile at `~/Library/Caches/ms-playwright-mcp/mcp-chrome-f397c72/`. The profile is singleton — only one Playwright session can hold it at a time. Before any `mcp__playwright__*` call, check the lock. If busy, register in the queue and wait. When your task finishes, release the lock and notify the next in line. Prevents the "Browser is already in use" error that blocks all sessions when a previous Claude/agent didn't shut down cleanly. Use whenever you're about to call any Playwright MCP tool, especially in a multi-project or multi-session context.
---

# Playwright queue discipline

The MCP Playwright server runs a **single Chrome instance** with a fixed user-data-dir. If two Claude sessions (or two agents in the same session) try to open the browser concurrently, the second one gets:

```
Error: Browser is already in use for /Users/miguel.reis/Library/Caches/ms-playwright-mcp/mcp-chrome-f397c72,
use --isolated to run multiple instances of the same browser
```

This skill establishes a cooperative queue so multiple projects can share the browser without stepping on each other, and so a hung session never permanently blocks the pool.

---

## The single source of truth

- **Profile path:** `~/Library/Caches/ms-playwright-mcp/mcp-chrome-f397c72/`
- **Lock files inside the profile:** `SingletonLock`, `SingletonCookie`, `SingletonSocket`
- **Queue file:** `~/.claude/playwright-queue.json` (created on demand)
- **Log file:** `~/.claude/playwright-queue.log` (append-only, one line per event)

---

## Queue file schema

```json
{
  "currentHolder": {
    "session": "sonnet-2026-07-20-a3f2",
    "project": "AAS Studio marketing council",
    "acquired": "2026-07-20T11:32:15Z",
    "estimatedFinish": "2026-07-20T11:45:00Z",
    "chromePid": 13149
  },
  "waiters": [
    {
      "session": "sonnet-2026-07-20-b7c8",
      "project": "iot-catalogue responsive QA",
      "requested": "2026-07-20T11:34:02Z",
      "estimatedDurationMin": 8
    }
  ]
}
```

If the file does not exist, no one currently holds the lock. If `waiters` is empty, the lock is free.

---

## Discipline (must be followed by every skill/session using Playwright)

### Before opening the browser

1. **Read** `~/.claude/playwright-queue.json`.
2. **Check** the OS-level lock: does `SingletonLock` exist in the profile dir AND does the referenced Chrome process still exist (`ps -p <pid>`)?
3. **Two cases:**

   **a) No holder or stale holder (lock file exists but PID is dead):**
   - Clean the stale lock: `rm -f ~/Library/Caches/ms-playwright-mcp/mcp-chrome-f397c72/Singleton*`
   - Write yourself as `currentHolder` in the queue file with current timestamp and an honest `estimatedFinish`.
   - Log: `ACQUIRE session=<X> project=<Y>`.
   - Proceed to open the browser.

   **b) Live holder:**
   - Append yourself to `waiters` in the queue file.
   - Log: `WAIT session=<X> project=<Y> holder=<Z>`.
   - **Wait strategy:** poll the queue file every 30 seconds, up to a max of `estimatedFinish + 5 min grace`. If the holder still hasn't released after grace, assume it crashed and treat as stale (case a).
   - When it's your turn (you are `waiters[0]` and no `currentHolder`), promote yourself to `currentHolder` and proceed.

### While holding the lock

- Do your Playwright work.
- If your work exceeds your `estimatedFinish` by more than 5 minutes, update `estimatedFinish` in the queue file so waiters know to be patient.

### After finishing (or if crashing)

1. **Close the browser** properly (`mcp__playwright__browser_close`).
2. **Remove yourself** from `currentHolder`. If waiters exist, `waiters[0]` becomes the new `currentHolder` (or leave the slot empty and let the next actor promote itself).
3. **Log:** `RELEASE session=<X> project=<Y> duration=<mins>`.
4. **On crash:** the next actor will detect the stale lock (PID dead) and clean up.

---

## Reference implementation (bash helpers)

Every skill using Playwright can source this helper. Save as `~/.claude/bin/playwright-queue.sh`:

```bash
#!/usr/bin/env bash
# Playwright cooperative queue helpers.
# Requires: jq

PROFILE_DIR="$HOME/Library/Caches/ms-playwright-mcp/mcp-chrome-f397c72"
QUEUE_FILE="$HOME/.claude/playwright-queue.json"
LOG_FILE="$HOME/.claude/playwright-queue.log"

pwq_now() { date -u +"%Y-%m-%dT%H:%M:%SZ"; }

pwq_log() { echo "$(pwq_now) $*" >> "$LOG_FILE"; }

pwq_stale_lock() {
  # Returns 0 if the SingletonLock is stale (PID dead or no lock).
  if [ ! -e "$PROFILE_DIR/SingletonLock" ]; then return 0; fi
  # SingletonLock symlink target format on macOS: <hostname>-<pid>
  local target
  target="$(readlink "$PROFILE_DIR/SingletonLock" 2>/dev/null || echo '')"
  local pid="${target##*-}"
  [ -z "$pid" ] && return 0
  ps -p "$pid" > /dev/null 2>&1 && return 1 || return 0
}

pwq_clean_stale() {
  rm -f "$PROFILE_DIR"/Singleton* 2>/dev/null
  pwq_log "CLEAN_STALE profile=$PROFILE_DIR"
}

pwq_acquire() {
  local session="$1" project="$2" estMin="${3:-15}"
  mkdir -p "$(dirname "$QUEUE_FILE")"
  [ ! -e "$QUEUE_FILE" ] && echo '{"currentHolder":null,"waiters":[]}' > "$QUEUE_FILE"

  while true; do
    if pwq_stale_lock; then pwq_clean_stale; fi

    local holder
    holder="$(jq -r '.currentHolder.session // empty' "$QUEUE_FILE")"
    if [ -z "$holder" ]; then
      local finish
      finish="$(date -u -v+${estMin}M +"%Y-%m-%dT%H:%M:%SZ" 2>/dev/null || date -u -d "+${estMin} minutes" +"%Y-%m-%dT%H:%M:%SZ")"
      jq --arg s "$session" --arg p "$project" --arg t "$(pwq_now)" --arg f "$finish" \
        '.currentHolder={session:$s,project:$p,acquired:$t,estimatedFinish:$f}' \
        "$QUEUE_FILE" > "$QUEUE_FILE.tmp" && mv "$QUEUE_FILE.tmp" "$QUEUE_FILE"
      pwq_log "ACQUIRE session=$session project=$project"
      return 0
    fi

    # Not our turn — add to waiters if not already
    if ! jq -e --arg s "$session" '.waiters | map(.session) | index($s)' "$QUEUE_FILE" > /dev/null; then
      jq --arg s "$session" --arg p "$project" --arg t "$(pwq_now)" --argjson d "$estMin" \
        '.waiters += [{session:$s,project:$p,requested:$t,estimatedDurationMin:$d}]' \
        "$QUEUE_FILE" > "$QUEUE_FILE.tmp" && mv "$QUEUE_FILE.tmp" "$QUEUE_FILE"
      pwq_log "WAIT session=$session project=$project holder=$holder"
    fi

    sleep 30
  done
}

pwq_release() {
  local session="$1"
  if [ ! -e "$QUEUE_FILE" ]; then return 0; fi
  jq --arg s "$session" \
    'if .currentHolder.session == $s then .currentHolder = null else . end
     | .waiters |= map(select(.session != $s))' \
    "$QUEUE_FILE" > "$QUEUE_FILE.tmp" && mv "$QUEUE_FILE.tmp" "$QUEUE_FILE"
  pwq_log "RELEASE session=$session"
}
```

Usage in a skill or session:

```bash
source ~/.claude/bin/playwright-queue.sh
SESSION_ID="claude-$(date +%s)-$$"
pwq_acquire "$SESSION_ID" "AAS Studio marketing" 15
# ... do Playwright work via MCP tools ...
pwq_release "$SESSION_ID"
```

---

## For Claude sessions (when to invoke this)

Before your first `mcp__playwright__browser_*` call in a session:

1. `Bash: source ~/.claude/bin/playwright-queue.sh && pwq_acquire "$SESSION_ID" "<project name>" <est min>`
2. The bash call blocks until you have the lock (up to the grace budget).
3. Do your Playwright work.
4. Before the session ends OR before you stop needing the browser: `mcp__playwright__browser_close` then `pwq_release "$SESSION_ID"`.

If a session dies mid-Playwright (crash, timeout, user kills terminal), the next session detects the stale lock (dead PID) and cleans up automatically. No manual intervention needed.

---

## Emergency recovery

If for any reason the queue is stuck (queue file corrupted, multiple stale entries, humans getting locked out):

```bash
# Nuke the queue and any stale browser locks
rm -f ~/.claude/playwright-queue.json
rm -f ~/Library/Caches/ms-playwright-mcp/mcp-chrome-f397c72/Singleton*
pkill -f "user-data-dir=.*mcp-chrome-f397c72"
```

Safe because:
- Queue file is regenerated on next acquire.
- Chrome profile itself (cookies, localStorage, cached auth) is untouched — only the ephemeral Singleton files are removed.
- Killing the Chrome process is soft (SIGTERM); Chrome writes cookies to disk on shutdown.

---

## Behavioural notes for Claude using Playwright

- **Never call `mcp__playwright__browser_navigate` blindly.** Always check the queue first.
- **Always release on finish.** Even trivial browser use (one navigate + one snapshot) — always release.
- **If you see "Browser is already in use" without going through the queue,** the previous session forgot to release. Do the emergency recovery, then proceed via the queue.
- **Estimate durations honestly.** If your task is 30 min, don't claim 5. Waiters plan around your `estimatedFinish`.
- **Don't `kill` other people's Chrome processes without checking the queue.** If a holder is legitimately using it, killing the process is disruptive. Only clean up stale locks (PID dead).
