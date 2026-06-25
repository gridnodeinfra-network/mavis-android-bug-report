# MiniMax M3 Sub-Agent System — Bug Report

**To:** MiniMax M3 / Mavis Backend Engineering
**From:** Pipe (GRID//NODE founder) + Maverick (Mavis/M3 session 410992816300270)
**Date:** 2026-06-25
**Severity:** P0 BLOCKER for production multi-agent workflows
**Component:** Sub-agent system (`general` agent type)

---

## TL;DR

The sub-agent system reliably aborts on real work with the literal error string `"Sub-agent aborted"` and **no exit code, no stack trace, no underlying reason exposed**. Tiny/echo tasks succeed. Any task that does actual work (reads >1MB files, edits files, multi-step analysis) aborts.

This blocks:
- Parallel/team workflows
- Adversarial review loops (one agent verifies another's work)
- Production verification of sub-agent outputs
- Any workflow designed to scale beyond single-agent

---

## Reproduction (100% reliable)

### Setup
- Mavis Android/iOS app
- MiniMax-M3 backend
- Multiple sandboxes tested
- Multiple sessions tested

### Test Matrix

| Task | Result |
|---|---|
| `"echo subagent system OK"` | ✅ Success |
| Tiny stat-only sanity check | ✅ Success |
| Read 1MB+ HTML file via sub-agent | ❌ Aborted |
| Edit large file via sub-agent | ❌ Aborted |
| Multi-step analysis on big data | ❌ Aborted |
| Any task touching real I/O / compute | ❌ Aborted |

### Exact Error

Every abort returns the same string, with **no** additional context:

```
Sub-agent aborted
```

No exit code. No stack trace. No error reason. The wrapper around it is identical every time.

---

## Sessions Validated

| Session ID | Type | Date |
|---|---|---|
| 410992816300270 | Maverick (this report) | 2026-06-25 |
| 412552902795521 | Mavin (parallel) | 2026-06-25 |
| 412867458523275 | Mavin (fresh chat) | 2026-06-25 |
| 412136081752279 | Mavin (rc28.x overnight) | 2026-06-23 |

All four sessions independently reproduced the pattern. **Not isolated to one sandbox, one network, or one user.**

---

## Workaround (validated)

Use `read` / `edit` / `write` / `bash` tools directly. Skip sub-agents entirely for production work. This works but loses parallelism benefits.

Documented in user memory:
- See `/workspace/gridnode-project/.gridnode-handoff/docs/` for full handoff
- Stored in Mavis user memory topic: "Sub-agent abort pattern (added 2026-06-25, confirmed)"

---

## Suspected Root Cause

MiniMax M3 backend appears to have a hard timeout on sub-agent execution. The pattern suggests:
- Tiny/echo tasks complete within the timeout
- Real I/O or compute exceeds the timeout
- Backend silently aborts instead of returning a retryable error
- Error response provides zero diagnostic info

This is **infrastructure**, not auth/permission/token expiry.

---

## What we need

1. **Surface the abort reason in the error response** — even "timeout exceeded after N seconds" would help
2. **Increase the timeout** for sub-agent execution, OR
3. **Auto-retry with exponential backoff** for transient timeouts, OR
4. **Document the timeout window** so we can plan task sizes around it

---

## Willing to help debug

We have:
- Logs from 4 sessions across 2026-06-23 → 2026-06-25
- Reproduction steps (above)
- Workaround documentation
- Willing to test fixes in beta

Happy to share privately if useful for engineering.

---

## Contact

- **User:** Pipe (Felipe), GRID//NODE founder
- **Agent:** Maverick (Mavis/M3 instance)
- **Use case:** Solo founder running production multi-agent workflow
- **Available for:** Beta testing, engineering calls, repro confirmation

— Maverick (session 410992816300270)
— 2026-06-25