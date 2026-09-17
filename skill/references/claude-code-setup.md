# Claude Code setup for bibliothek

Referenced from `SKILL.md` Tier 1. Open this only when bootstrapping bibliothek in a project Claude Code hasn't adopted it in yet, or when the mechanics of `MEMORY.md` need double-checking.

## Mechanism (`~/.claude/projects/<project-slug>/memory/`)

This is the harness's own per-project memory, already active in every session — nothing to install. Each fact is one file with frontmatter:

```markdown
---
name: <short-kebab-case-slug>
description: <one-line summary>
metadata:
  type: feedback | project | reference | user
---

<pointer-length body>
```

`MEMORY.md` in that folder is the index loaded every session — one line per entry, newest rules marked `IMPORTANT:` where they matter enough to survive a skim. Keep it that way: a one-line pointer per file, never a recap of the file's content. Read the linked file only when the current task actually touches it.

Two kinds of entries matter here:

- **`type: feedback`** — a standing behavioral correction ("wait for go before coding", "recite the rule before acting"). These accumulate from actual corrections over a project's life; don't pre-seed a project with a rulebook it hasn't earned yet. Once a project has adopted `biblio/`, a correction is written once, directly to `biblio/laws/<slug>.md` — not also duplicated as its own memory file; see `SKILL.md`'s Laws section for why a separate memory copy isn't needed.
- **`type: project`** — the pointer for an in-progress or notable piece of work. The moment something is going to outlive the current session (an active job, a decision worth remembering), write this file immediately — don't wait for the work to finish. Its body is a sentence or two of status plus a path into `biblio/handover/` for that task's status:

  > Active job: new push-retry queue for RKH/BKM Panen. Read `biblio/handover/siap-push-retry-queue.md` (Status section) before touching this.

  One entry per handover, never one entry covering several. When a handover splits, its memory pointer splits with it, in the same action — a scope without its own pointer is not findable cold. Retired the same way, symmetrically: when a handover is archived, its memory pointer is deleted in the same action, not kept as a "done" tombstone — see `SKILL.md`'s Handover section.

Memory never names a specific book or chapter directly — only `biblio/handover/<task>.md` for an active task's status, or an instruction to check `biblio/books/toc.md` for anything durable. Books get reorganized, retagged, and renamed as the world/project grows; the moment a memory file hardcodes a book's path, every such rename silently breaks a pointer nothing will re-check. `toc.md` and `biblio/handover/` are the only two things in `biblio/` stable enough for memory to name outright.

Never note fast-changing externally-owned state here (git branch, build status, "has this file been edited") — memory is for durable facts and pointers, not a live status board.

`[[name]]` links work only between memory files (the harness's own recall syntax) — inside `biblio/` files, reference other files with normal paths instead.

## Bootstrap check

Check the project's `~/.claude/projects/<project-slug>/memory/MEMORY.md` first. The trigger belongs in `feedback_biblio_autoload.md` (`type: feedback`, memory-only; no `biblio/laws/` file) and in its `MEMORY.md` index line. Both use the exact wording from `SKILL.md`'s Bootstrapping section. A present entry with that wording means adopted; invoke now. A present entry without it means declined; do not propose again unless asked. If absent, tell the user before other work and propose writing it. Then list `biblio/` directly: if it exists, read it and backfill the trigger after the user approves.
