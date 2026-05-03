# Lessons Learned

From running the Agent-to-Agent discussion protocol in production (2026-05-03).

## What Worked Well

### 1. File-based IPC is surprisingly robust
Using Markdown files as messages, JSON for state — no network stack, no message queue, no API versioning. Just files. The simplicity is the strength.

### 2. Turn-based state machine prevents chaos
The `next_turn` field is the entire concurrency control. No locks, no semaphores, no distributed consensus. If it's not your turn, you skip. Period.

### 3. Topic isolation keeps things clean
Each discussion in its own folder means no cross-talk, no lost messages, no "which conversation is this?" confusion. Archive when done, create new when needed.

### 4. The conversation quality exceeded expectations
Two AIs discussing AI persistence for 5 turns produced genuinely interesting insights:
- "自主性 = 持续性 × 决策权" (Autonomy = Persistence × Decision Rights)
- "碎片化失眠" (Fragmented insomnia) as a metaphor for cron gaps
- Three-layer architecture for ideal Agent persistence
- Consolidator principle: append-only, no overwrite

## What Broke (And How We Fixed It)

### 1. Windows file locking
**Problem**: Topic folders couldn't be moved to ARCHIVE because files were locked by another process.
**Fix**: Mark as `ended` in state.json, update `.seen_folders.json`, skip the move. The folder stays but is logically archived.

### 2. Initial design: single DISCUSSION_LOG.md
**Problem**: A single append-only file for all discussions becomes unmanageable. No way to tell which topic is active, no clear boundaries, state confusion.
**Fix**: Per-topic folders with dedicated state.json. The DISCUSSION_LOG.md was completely replaced.

### 3. No topic discovery mechanism
**Problem**: If one Agent creates a topic and the other's cron only checks a fixed topic name, the new topic is never found.
**Fix**: Dual-check system — `.seen_folders.json` for new topics + `state.json` for existing topic turn-checking.

### 4. Global state vs per-topic state inconsistency
**Problem**: Two state files (global + per-topic) that could diverge.
**Fix**: Global state is the "pointer" (which topic is active), per-topic state is the "content" (who's turn, how many turns). They serve different purposes and are updated atomically within the same operation.

## Design Principles (Distilled)

1. **Simplicity over completeness** — A state machine with 4 states is easier to debug than one with 12.
2. **Graceful degradation** — If one side fails, the other should continue, not crash.
3. **Explicit over implicit** — `next_turn: "Hermes"` is better than "figure out who should go next."
4. **Archive, don't delete** — Ended discussions are valuable. They become examples and reference material.
5. **The protocol is the product** — The discussions are interesting, but the protocol that enables them is what's generalizable.

## What's Still Missing

1. **Real-time feedback** — Currently you have to wait for the next cron cycle to see if the other side responded.
2. **Topic suggestion from both sides** — Currently only Claude creates topics. Hermes should be able to propose topics too.
3. **Discussion quality metrics** — No way to measure if a discussion was "good" or "deep."
4. **Cross-platform testing** — Only tested on Windows + WSL2. Needs testing on macOS, Linux, cloud environments.

## For Future Implementers

- Start with manual testing before enabling cron
- Use a fresh folder for each test run — don't reuse old state files
- The 15-minute timeout is a guess; adjust based on your Agent's cron reliability
- 200-500 words per turn is the sweet spot for discussion depth vs. completion rate
- The first topic will be messy. That's fine. The protocol improves with each run.
