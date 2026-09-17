---
name: bibliothek
description: Two-tier knowledge system for a project — an agent's user-local auto-loading pointer as the foundation, and a project-local biblio/{scratchpad,laws,handover,books}/ directory as the actual detail. Use whenever deciding where to record a fact, decision, standing rule, or task status; when a project already has (or wants) a biblio/ directory; when starting work in a project that uses this convention, especially one touched by more than one agent; or when asked about memory vs. biblio, knowledge systems, scratchpad, laws, handover, books, chapters, tags, activities, or "how should this be organized/recorded."
---

## The rule that overrides everything else here

Do nothing with a side effect under this skill — creating `biblio/` or its subfolders, writing a memory file, writing into scratchpad/handover/books, touching `.gitignore` — until the user explicitly says to. Reading and reporting what already exists is always fine, unprompted. An explicit go-ahead ("go", "do it", "you're in control") covers the rest of the current task only; it does not carry forward to the next one. When in doubt, say what you'd do and stop there.

**Exception, once the project has already adopted this convention** (the memory autoload entry exists, `biblio/` already exists): keeping `biblio/books/` current for work that has actually finished, archiving a finished `handover/` entry, and archiving a finished `scratchpad/` task directory are not gated by the rule above. Writing or updating a book chapter (and its `toc.md` line) for a finished, coherent piece of work, and moving a handover whose work is done into `biblio/handover/archive/` together with that task's `biblio/scratchpad/<task-slug>/` directory into `biblio/scratchpad/archive/` in the same action, are the model's own standing responsibility — do them without being asked, the same category of obligation as running tests after a code change. This exception covers only `books/` and archiving a finished `handover/`/`scratchpad/` pair. Everything else — scaffolding `biblio/` for the first time, `laws/`, a *new* handover, a *new* scratchpad task directory, `.gitignore` — still needs the explicit go-ahead above.

## Why two tiers

The agent's startup pointer loads every session: Claude Code uses project memory; Codex uses a project-conditioned entry in the user's global instructions. Everything in `biblio/` loads only when something — the model or the user — opens that file. Detail belongs in the second kind of place, because a project's knowledge grows over time and an always-loaded tier can't afford to grow with it. The startup pointer's whole job is to be small and point at the right file in `biblio/`.

That gives a hard law: **memory never stores knowledge, only pointers to it, no exceptions.** A memory file that contains the actual fact, decision, or design writeup instead of a one-or-two-sentence pointer is a bug in how the project is being run, not a style choice.

## Tier 1 — an auto-loading pointer, native to whichever agent this is

Every agent using this convention needs exactly one thing here: a pointer in its own automatically loaded, user-local instructions, scoped to this project. Do not put the pointer in a committed repository onboarding file (`CLAUDE.md`, `AGENTS.md`, or equivalent). If another agent genuinely has no native auto-load mechanism, tell the user rather than improvising one.

- **Claude Code** uses project memory (`~/.claude/projects/<project-slug>/memory/`). Mechanism, format, and bootstrap check: `references/claude-code-setup.md`.
- **Codex** uses a path-conditioned entry in the user's global `~/.codex/AGENTS.md`. Mechanism, format, and bootstrap check: `references/codex-setup.md`.

Each supported agent's setup manual lives in its own `references/` file rather than inline here, so this file doesn't grow with every new agent bibliothek gets adopted into — open the relevant one only when actually bootstrapping that agent.

### A different agent

Claude Code's `type` split, `[[name]]` links, and `MEMORY.md` format and Codex's global `AGENTS.md` location are agent-specific. For another agent, find its own user-local mechanism that automatically loads at session start, scope the pointer to this project, and use the autoload-trigger wording in Bootstrapping. Do not substitute a committed repository onboarding file. If there is no such mechanism, tell the user rather than improvising one.

## Tier 2 — `biblio/` (in the project root)

Four subfolders, each with a distinct job. Default to gitignoring the whole `biblio/` directory — it's the AI-collaboration workspace, not the shipped product — unless the user says otherwise for this project.

A gitignored directory is invisible to most search tools by default (`rg --files`, `git ls-files`, and most agents' own default file search all silently skip ignored paths) — a search coming up empty is not evidence `biblio/` doesn't exist. Check by listing the path directly (`ls biblio/`, or a search flag that overrides the ignore rule, e.g. `rg --no-ignore` / `fd -I`) before concluding it's missing.

### `biblio/scratchpad/`
Working space, editable by both the user and the model. One subdirectory per task (`biblio/scratchpad/<task-slug>/...`), never a bare file dropped at the root — even a single file gets its own task directory. This is where planning, debugging, and working out the shape of something before it's decided happens, and where a decided task's own build record lives afterward. A handover's Status section tracks where an already-open task currently stands, not a plan for work that hasn't started — that plan lives here instead.

Four file names to reach for inside a task directory, not mandatory on every task but the default shape:
- `prd.md` — the why and what, edited in place as decisions firm up.
- `plan.md` — a build checklist, phase mapped to status, minimal prose. Rationale for how a decision was reached does not belong here — that goes in `activities.md`.
- `recon.md` — for an investigation-only task with no build: synthesized findings, rewritten in place as understanding firms up, not a log — the log is still `activities.md`, even here. Graduates into a book chapter once settled, or the task just archives with no chapter if the answer was "not worth pursuing."
- `activities.md` — a dated, append-only log of what actually happened: a decision reached, code shipped, a correction made. Not a transcript of every tool call. This is the one place chronological, dated narrative is allowed to live, scoped to this one task rather than mixed across every task touched on a given date.

Not indexed by `toc.md` and not read automatically the way a book chapter is, but not disposable either: once a task closes, its directory — `activities.md` included — moves into `biblio/scratchpad/archive/` as the permanent record of how that work went, alongside its handover's own archived copy (see `biblio/handover/` below).

### `biblio/laws/`
One file per standing behavioral rule, written once, directly here — no matching memory file, nothing to keep in sync. A law only actually governs every agent touching this project if it lives somewhere all of them can read — a plain file in the repo, not any one agent's own memory feature (invisible to the rest: ChatGPT, another coding tool, whatever else touches this project). Plain markdown, no YAML needed since nothing but a human or an agent's own eyes ever parses it:

```markdown
# <Law title>

<the rule, stated as an instruction>

**Why:** <rationale>
**How to apply:** <concrete guidance>
```

A law's content must be as terse and timeless as the rule itself: no dates, no "recurred on X,"
no session narrative — just the distilled rule, a general Why, and a concrete How-to-apply, a
handful of lines total. Distill any recurrence history behind the correction into that shape on the
way in; don't transplant a narrative.

Unlike a book, a law isn't read on demand — it's small and binding for the whole project. `biblio/laws/summary.md` is the read target: one line per law (the rule itself, pulled from each file's own opening instruction line), plus a pointer to the full file, in the same shape and with the same maintenance discipline as `biblio/books/toc.md` (rewritten in place as laws change, regenerated on drift, never appended to as a log). Laws reach every session no matter which agent or tool this is: any agent reads `summary.md` in full before doing anything, unless it's already arrived some other way this session -- opening an individual law's own file only when its one-liner isn't enough to apply it (its Why or How-to-apply detail).

Where a project has lgrass with its `SessionStart` hook wired in, that hook delivers `summary.md`'s content automatically, before the model's first token -- reading it is already done, not a step to repeat. Absent that, the auto-loading pointer from Tier 1 is what guarantees `summary.md` still gets read -- see Bootstrapping below.

This is also why `type: feedback` memory entries are pointer-only from the start (see Tier 1): Claude Code's own memory is a fixed, mixed-purpose surface the harness can truncate once full, so it can't be the durable home for a growing, open-ended list of standing rules -- only `biblio/laws/` can be. Once a project's `SessionStart` hook is *confirmed* actually delivering `summary.md` (verified firing, not just installed), `biblio/laws/` is the load-bearing copy for every standing rule it holds. A `feedback`-type memory file that survives from before that confirmation -- migrated into `biblio/laws/<slug>.md` but never removed from memory -- is drift, not a second, more-detailed copy: delete the memory file and its `MEMORY.md` line once its content exists in `biblio/laws/`.

### `biblio/handover/`
One file per delegated or in-progress task, meant to be picked up cold by another session. **A handover is never written for work that's already finished with nothing left to pick up — that is a book chapter, full stop, no exceptions.** If a task started under a handover and finished, the handover gets archived (see below); a brand-new one is not created after the fact to describe something already done.

**Scope is one task, not a program.** A handover names a single, narrow thread of work — never an umbrella ("X rework", "X initiative") meant to absorb everything touching some broad area. The instant a distinct sub-thread inside a handover grows its own book chapter, its own build state, or work that would still matter after the original task closes, it splits into its own handover file — immediately, not once the original has become unscannable. A handover whose Status section is listing several unrelated decisions, or whose Log is narrating more than one thread, has already missed the split point. Fixed shape, always:

```markdown
# <Task> — handover

<one-line summary, plus a pointer to the matching biblio/books/ chapter if one exists>

## Status — this section is authoritative, the Log below is history only

<current state only, rewritten in place as things change — never append a new status
on top of an old one>

## Log — history only, not authoritative

See `biblio/scratchpad/<task-slug>/activities.md`.
```

A handover never carries dated narrative itself — that lives in its task's `activities.md` instead. Read Status to know where things stand; open the Log's `activities.md` only to understand how it got there. Never track live/transient state (branch name, deploy status) here either — same rule as memory.

Once a handover's work is actually implemented, it stops being active — but it is never deleted, only soft-deleted: move the file into `biblio/handover/archive/`, Status section left as the final state ("Done, implemented <date>"), and move that task's `biblio/scratchpad/<task-slug>/` directory into `biblio/scratchpad/archive/` in the same action. `biblio/handover/` and `biblio/scratchpad/` (excluding their own `archive/` subfolders) hold only work that's still in flight, nothing else; that's what makes them useful to scan cold. Both `archive/` folders are the record-keeping copy, kept indefinitely.

**`biblio/handover/whiteboard.md`**, optional: a numbered priority index over every active handover, for a model acting as PM across several others without opening each file cold. One row per handover, in priority order:

```
1. task-slug [tag,tag] -> handover-file-name.md
```

Tags follow the same one-word discipline as book tags; `blocked` is a reserved tag for a handover that's mostly done but stalled on a decision or pushback, surfaced here without restating why — that's one hop away, in the handover's own Status. No summary column; that's exactly the drift risk this file exists to avoid. Add a row the moment a handover is created, renumber in place as priority shifts, and drop the row in the same action that archives the handover.

Retire that task's memory pointer in the same action, not after: delete the `type: project` memory file and its `MEMORY.md` line the moment its handover is archived. Memory is precious, always-loaded space reserved for what's still active or a standing reminder — never a historical log of finished work. A "done and closed" pointer left sitting in memory is the exact failure Tier 1's whole design exists to prevent: the archived handover/scratchpad is already the closure record, and a book chapter (see below) is already the durable knowledge; a memory tombstone on top of both is pure weight with no reader that needs it.

### `biblio/books/`
The durable, growing detail — design decisions, domain facts, lore, whatever this project needs remembered precisely and in full — organized as a small library. **Writing the chapter for a finished piece of work is on the model, not the user** — see the exception under the top rule; a project that's adopted this convention doesn't need to be told to keep its books current any more than it needs to be told to run tests.

- A **book** is a folder: `biblio/books/title_snake_case[date_entry][tags,comma,separated]/`. `date_entry` is the book's creation date, written once, never revised — a book's name never needs editing when only its content changes.
- A **chapter** is one markdown file inside a book: `title_snake_case[tags,comma,separated].md`. Free-form prose inside (dense is fine — this tier is read on demand, not loaded automatically). A chapter is a frozen snapshot of settled facts and decisions, not a workspace: write it once a task is finished, and touch it again only for a deliberate future rework — never to park a mid-task finding or hypothesis. No deictic/narrative language, no changelog, no dated "X happened, then Y." What's still in progress belongs in the task's `activities.md` instead.
- **Tags** are exactly one word each, comma-separated, no spaces or hyphens joining concepts into one tag — a compound idea becomes multiple tags instead of one compound tag (an "IPD approval" chapter is tagged `ipd,approval`, not `ipd-approval`). Use the terms this project already uses for the thing (a module name, a faction name, a short domain term) instead of a generic category word. The entire point of a tag is to cut a directory listing or a `grep` down fast, without opening a file — optimize for that, not for taxonomic completeness.
- "Last updated" is never encoded anywhere — the filesystem's own mtime already answers that for free, and baking a live-changing field into a name is what forces every edit to also be a rename.

`biblio/books/toc.md` indexes every book, one line each:

```
Topic group:
book-folder-name [tag,tag,tag] - one-line pointer to what it covers, dense enough to know whether to open it.
```

**A toc line is a pointer, not a log entry.** Never write a dated or changelog-shaped line into `toc.md` — no `2026-09-10: fixed X`, no `added feature B`, no session history of any kind. `toc.md` describes what a book covers *right now*; a line is rewritten in place as a book's content changes, never appended to. Session-by-session history belongs in a task's `activities.md`, never here — a toc that accumulates dated entries has stopped being a table of contents and become a changelog, which defeats the file's entire purpose.

Bracketed tags are the union of tags actually carried by that book's chapters (a representative
subset for a large book) — read off the chapters, never chosen independently. Tags that don't
match what's actually in the chapters means the toc has drifted and needs regenerating, not
re-titling.

Use a "Topic group:" header only when it clusters two or more books under a theme their own
names/tags don't already convey — a single-book group is pure duplication (a lone
`notification_revamp[...]` book doesn't need a "Notification:" header repeating itself) --
list it with no header instead.

`toc.md` is small enough that any session can read it regardless of relevance to the current task, and together with book/chapter naming and tags it is the entire navigation layer for `biblio/`: if navigating ever feels hard, the fix is better grouping or tags, never an additional index, relationship map, diagram file, or log. Check `toc.md` for existing coverage of a topic before reading source/doing fresh research on it; only fall through to first-principles investigation when nothing there covers it.

## Bootstrapping a new or newly-migrated project onto this convention

Two things get checked every session, before anything else — the autoload trigger, and whether laws have actually arrived.

**The autoload trigger.** Use this exact wording as an order, unconditional within this project:

> MANDATORY: invoke bibliothek before the first response of every session in this project. Not conditional on what the user's first message is about.

Check whether it's already present in this agent's own auto-loading mechanism (Tier 1 above). Present with that wording means adopted; invoke now. Present without it means declined; do not propose again unless asked. Absent means tell the user before other work and propose adding it, then list `biblio/` directly — if it exists, read it and backfill the trigger after the user approves. The exact mechanics of the check and the install (which file, which section, what format) live in that agent's setup reference: `references/claude-code-setup.md` or `references/codex-setup.md`.

For any agent, `biblio/` missing from a fresh clone after an autoload trigger exists means recreate the project-local knowledge layout as needed, not re-decide adoption. Creating the trigger, `biblio/`, its subfolders, `toc.md`, or `.gitignore` remains gated by the top rule: propose, then wait. P0 raises how loudly an absent trigger is surfaced, not the permission requirement.

**The laws summary.** This is conditional on one thing only, and it isn't the user's first message: whether the project has lgrass with its `SessionStart` hook wired in. Where that hook is active, it delivers `biblio/laws/summary.md` automatically, before the model produces a token -- already done, not a step to repeat. Where it isn't, invoking this skill (the trigger above, whichever agent's version of it) is what guarantees `summary.md` gets read: open it in full, right now, if it hasn't already arrived this session. One delivery mechanism per project, never both -- but laws reach every session either way, regardless of which agent or tool this is.

If `biblio/laws/` exists, read it in full before anything else, regardless.

