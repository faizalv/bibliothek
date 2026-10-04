# Codex setup for bibliothek

Referenced from `SKILL.md` Tier 1. Open this only when bootstrapping bibliothek in a project Codex hasn't adopted it in yet.

## Mechanism (`~/.codex/AGENTS.md`)

Codex reads the user's global `~/.codex/AGENTS.md` at the start of every session. Keep one path-conditioned pointer there for each adopted project. This file is private to the user's Codex installation, outside the repository; never create a repository `AGENTS.md` for bibliothek. If `~/.codex/AGENTS.override.md` exists, Codex loads it instead of `AGENTS.md`, so put the pointer in the active file or reconcile the override first. Install or link this skill under `~/.codex/skills/bibliothek/` so Codex can discover it, and include its `SKILL.md` path in the pointer.

Use the exact autoload-trigger sentence from `SKILL.md`'s Bootstrapping section, prefixed with a condition matching the project's absolute root path. The path condition prevents instructions for one project from applying in another. Keep only the trigger and paths to the skill, `biblio/laws/summary.md`, and `biblio/books/toc.md` here; all project knowledge stays in `biblio/`. Codex's optional local memories are generated asynchronously and may not load a specific instruction in the next session. Do not edit their generated files as the primary setup method.

Content already delivered in the current session counts as read. Do not reopen a referenced file when its complete contents are already in context. In particular, a complete `biblio/laws/summary.md` injected by SessionStart or System Prompt is the delivery proof for that file; invoke bibliothek as required, but skip reopening the summary. Read the skill and any other required file that has not already been delivered.

## Bootstrap check

Check the active user-local global instruction file, `~/.codex/AGENTS.override.md` if present, otherwise `~/.codex/AGENTS.md`. The trigger belongs in one short entry conditioned on this project's absolute root path, followed by a pointer to `~/.codex/skills/bibliothek/SKILL.md` (or its actual installed path), `biblio/laws/summary.md`, and `biblio/books/toc.md`. State that a complete Laws summary already in session context is delivered content and must not be reopened. A matching entry means adopted; invoke now. If absent, tell the user before other work and propose adding it. Then list `biblio/` directly: if it exists, read it and add the trigger after the user approves. Never create a repository `AGENTS.md` for this setup.
