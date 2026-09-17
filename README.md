# bibliothek

A two-tier knowledge system for AI coding agents working in a project.

## Problem it addresses

An agent's own memory feature (Claude Code's project memory, Codex's global `AGENTS.md`, etc.) loads into every session automatically, in full. That makes it a poor place to store a project's actual, growing knowledge: standing rules, task history, design decisions. A memory surface that holds everything eventually gets truncated or becomes too large to load without diluting attention on every single turn.

bibliothek splits this into two tiers.

## Tier 1: the pointer

Each agent's own auto-loading, user-local mechanism holds exactly one thing: a pointer telling it to invoke the bibliothek skill and read the project's `biblio/` directory. Nothing else lives there. The pointer itself is a fixed, short instruction, not a place for project detail.

## Tier 2: `biblio/`

A directory in the project root, gitignored by default, with four subfolders:

- `biblio/laws/` — one file per standing behavioral rule (e.g. "never use em dashes," "wait for explicit confirmation before touching code"). `biblio/laws/summary.md` is a one-line-per-law index, read in full at the start of every session.
- `biblio/handover/` — one file per in-progress task, meant to be picked up cold by another session. Has a Status section (current state, rewritten in place) and a Log section pointing at that task's `activities.md`. An optional `whiteboard.md` indexes every active handover by priority and tags, for a model acting as PM across several at once.
- `biblio/scratchpad/` — working space, one subdirectory per task. Holds `prd.md`, `plan.md`, `recon.md` (synthesized findings for an investigation-only task), and `activities.md` (a dated, append-only log of what actually happened) while a task is in progress. Moves to `biblio/scratchpad/archive/` once the task closes.
- `biblio/books/` — durable, growing detail: design decisions, domain facts, anything worth remembering in full once a piece of work is finished. A chapter is a frozen snapshot, written once a task is done and touched again only for a deliberate future rework. Organized as books (folders) and chapters (files inside them), indexed by `biblio/books/toc.md`.

## Supported agents

- **Claude Code** — the pointer lives in Claude Code's own project memory (`~/.claude/projects/<project-slug>/memory/`). Setup detail: `skill/references/claude-code-setup.md`.
- **Codex** — the pointer lives in a path-conditioned entry in the user's global `~/.codex/AGENTS.md`. Setup detail: `skill/references/codex-setup.md`.

Each agent's setup manual lives in its own file under `skill/references/` rather than inline in `SKILL.md`, so the core skill file does not grow with every agent that adopts bibliothek. Adding support for another agent means adding one `skill/references/<agent>-setup.md` file and one pointer line in `SKILL.md`, not editing the shared content.

## Layout

```
bibliothek/
├── README.md
└── skill/
    ├── SKILL.md
    └── references/
        ├── claude-code-setup.md
        └── codex-setup.md
```

`skill/` is the actual skill package: what an agent reads when the skill is invoked. Everything above it (this README) is for humans browsing the repository.

## Install

Symlink the `skill/` directory into the agent's skills folder.

Claude Code:

```
ln -s /path/to/bibliothek/skill ~/.claude/skills/bibliothek
```

Codex:

```
ln -s /path/to/bibliothek/skill ~/.codex/skills/bibliothek
```

Then, in a project you want to adopt bibliothek in, invoke the skill. It checks whether the project has already adopted the convention and, if not, proposes setting it up: writing the pointer in the agent's own mechanism, and scaffolding `biblio/` in the project.
