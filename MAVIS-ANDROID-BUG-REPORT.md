# Mavis Android App — Bug Report

**Reporter:** Pipe (Felipe), founder GRID//NODE
**Date:** 2026-06-25
**App:** Mavis Android (MiniMax-M3 backend)
**Severity:** Multiple blockers for production work

---

## Issue 1: Web/File viewer permanently stuck (BLOCKER)

**What happens:** Tapping any file attachment in a Mavis chat opens a "page details" view (网页详情) that hangs on a "Loading file..." spinner forever. Never resolves. No error. No retry.

**Affected:**
- iOS Safari (latest)
- Android Chrome (latest)
- Multiple file types: HTML, MD, PNG, JPG, ZIP, plain text

**Reproducibility:** 100%. Every file, every time.

**Impact:** Cannot inspect deliverables from Mavins. Cannot preview screenshots. Cannot review generated code. Cannot audit reports without copy-paste into another app.

**Workaround (limited):**
- HTML files → deploy to live URL (gridnode.network)
- Screenshots → re-send via in-chat `<media>` block (auto-detected)
- MD/text → paste inline (loses formatting)
- Large files → external cloud (Drive)

---

## Issue 2: Cannot download or copy files to local device (BLOCKER)

**What happens:** Files delivered by Mavis (deliverables, reports, archives) show up in chat but cannot be saved to:
- Phone storage / Downloads
- Camera roll (for images)
- Files app
- External share targets

**Affected:**
- Android Mavis app
- iOS Mavis app (presumed, not directly tested)

**Reproducibility:** 100%. Every file.

**Impact:** Cannot move work product off the chat. Cannot share deliverables with team. Cannot archive builds locally. Cannot open files in other apps.

**Workaround:**
- For screenshots: long-press in chat, save (sometimes works, often doesn't)
- For all files: ask Mavin to upload to Drive separately
- For code/text: copy-paste manually (lossy)

---

## Issue 3: Sub-agent system aborts on real work (BLOCKER)

**What happens:** When Mavin spawns a sub-agent (`general` type) for any task that does real work (reads >1MB files, edits, multi-step analysis), the sub-agent returns literally:
```
Sub-agent aborted
```
No exit code. No stack trace. No underlying reason. Same error every time.

**Pattern (validated 2 sandboxes, 6+ attempts):**

| Task | Result |
|---|---|
| `echo OK` (no-op) | ✅ success |
| Tiny stat / sanity check | ✅ success |
| Read 1MB HTML file | ❌ aborted |
| Edit large file | ❌ aborted |
| Multi-step analysis | ❌ aborted |

**Likely cause:** MiniMax M3 backend timeout on sub-agent execution. Not auth, not permission. Pattern has been intermittent through 2026-06-23 → 2026-06-25.

**Workaround:** Use `read` / `edit` / `write` / `bash` tools directly. Skip sub-agents entirely for production work. Loses parallel/team workflow capabilities.

---

## Issue 4: Chat input / send button gets stuck blue (MEDIUM)

**What happens:** After a tool-using turn, the Mavis chat input area shows a blue indicator (agent mode active) and tapping the input field does nothing. Cannot type or send new messages.

**Affected:** iOS Mavis app (Android not tested by us but likely)

**Reproducibility:** Intermittent. Happens after long tool-using sequences.

**Impact:** Have to force-close and reopen app. Conversation state preserved.

**Workaround:** Force-close + reopen Mavis app.

---

## Issue 5: M3 background dispatcher / stream errors (HIGH)

**What happens:** Throughout long sessions, M3 backend throws errors visible in agent responses:
- `[stream.message] system error`
- `[background.dispatcher] system error`
- `queue.start_loo` errors

**Reproducibility:** Random, ~10-20% of long-running turns.

**Impact:** Agent responses get cut off, retry required, sometimes message lost. Severe during 8+ hour continuous sessions.

**Workaround:** Save state externally (handoff docs), use shorter sessions.

---

## Issue 6: M3 cache_control ignored on /anthropic/v1/messages (MEDIUM)

**What happens:** When Mavis routes through `/anthropic/v1/messages` endpoint, the `cache_control` markers are ignored. This causes 5x over-billing on cached contexts.

**Workaround:** Use `/v1/chat/completions` endpoint instead (passive caching works correctly). Mavis should switch default route.

---

## Issue 7: New chat sometimes spawns non-Maverick agent (LOW)

**What happens:** Tapping "+" to start new chat sometimes spawns a different agent (different persona, different context) instead of Maverick. The new agent may know Maverick from handoff docs but isn't Maverick itself.

**Impact:** Discontinuity in conversation. User has to check chat header to verify agent identity.

**Workaround:** Verify agent name in chat header. If wrong, abandon and retry.

---

## Environment details

- **Device:** Android (Pixel-class, model varies)
- **OS:** Android 14 / 15
- **Mavis app version:** Latest from Play Store
- **Backend:** MiniMax-M3 via Mavis daemon
- **Network:** Stable WiFi + 5G
- **Sandboxes tested:** 2 separate Mavis sandboxes, both show same sub-agent pattern

---

## What would unblock us

| Priority | Fix | Effort |
|---|---|---|
| 1 | Web/file viewer: resolve "Loading file..." → render or show error | Mavis team |
| 2 | File download: add Save / Share / Open-in action | Mavis team |
| 3 | Sub-agent: increase timeout or fix abort reason exposure | M3 backend |
| 4 | M3 stream errors: rate-limit or auto-retry at daemon | M3 backend |
| 5 | Cache_control: switch default route to /v1/chat/completions | Mavis routing |
| 6 | New chat default agent: pin to last-used agent | Mavis app |

---

## Willing to help

We have:
- Logs from 2 sandboxes across 2026-06-23 → 2026-06-25
- Reproduction steps for each issue
- A "worse is better" workaround doc that gets users unblocked

Happy to share privately if useful for engineering.

— Pipe (GRID//NODE founder)
GRIDNODE_HANDOFF project, ~30 hours of testing
Maverick (Mavis/M3 session 410992816300270)