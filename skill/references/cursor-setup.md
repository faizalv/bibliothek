# Cursor setup for bibliothek

Referenced from `SKILL.md` Tier 1. Open this only when bootstrapping bibliothek in a project Cursor hasn't adopted it in yet.

## Mechanism (user-level `sessionStart` hook)

Cursor does not ship a first-class per-project, user-local always-on memory file comparable to Claude Code's project memory or Codex's path-conditioned `~/.codex/AGENTS.md`.

The Tier 1 pointer for Cursor lives in a **user-level** `sessionStart` hook:

- Config: `~/.cursor/hooks.json` → `hooks.sessionStart`
- Script: `~/.cursor/hooks/bibliothek-session-start.sh`
- Adopted-project registry: `~/.cursor/hooks/bibliothek-projects.txt` (one absolute project root per line)

The hook reads the session's `workspace_roots`, and if the workspace matches a registry entry **or** already contains `biblio/`, it injects `additional_context` with the exact autoload-trigger wording from `SKILL.md`'s Bootstrapping section, plus paths to the skill, `biblio/laws/summary.md`, and `biblio/books/toc.md`.

This file is private to the user's Cursor installation, outside the repository. Never create a committed project `.cursor/rules/`, `AGENTS.md`, or `CLAUDE.md` entry for bibliothek.

### Skill install

```
ln -s /path/to/bibliothek/skill ~/.cursor/skills/bibliothek
```

Cursor also discovers skills under `~/.claude/skills/` and `~/.codex/skills/` for compatibility; prefer `~/.cursor/skills/bibliothek` for Cursor-first installs.

### Hook install

1. Ensure `~/.cursor/hooks/bibliothek-session-start.sh` is executable.
2. Add a `sessionStart` entry in `~/.cursor/hooks.json` (merge with any existing hooks; do not replace unrelated hooks):

```json
{
  "version": 1,
  "hooks": {
    "sessionStart": [
      {
        "command": "/Users/<you>/.cursor/hooks/bibliothek-session-start.sh",
        "timeout": 5
      }
    ]
  }
}
```

3. When adopting a project, append its absolute root path to `~/.cursor/hooks/bibliothek-projects.txt`.

### Caveats

- User hooks apply to local IDE / Agent Chat. Cloud Agents do not load user `sessionStart` the same way — fall back to manually invoking `/bibliothek` or asking the agent to invoke the skill.
- Skills are relevance-gated by default; the hook's `additional_context` is what makes the trigger mandatory for adopted workspaces.
- Do not put the pointer in User Rules as a global always-on instruction for every project — that leaks one project's adoption into unrelated workspaces. Path gating belongs in the hook/registry.

## Bootstrap check

1. Confirm the skill is discoverable at `~/.cursor/skills/bibliothek/SKILL.md` (or the active install path).
2. Confirm `sessionStart` in `~/.cursor/hooks.json` points at `bibliothek-session-start.sh`.
3. Check whether this project's absolute root is listed in `~/.cursor/hooks/bibliothek-projects.txt`, **or** whether `biblio/` already exists (list the path directly; gitignored trees are invisible to default search).
4. A matching registry entry or existing `biblio/` means adopted; invoke now and read `biblio/laws/summary.md` if present.
5. If absent, tell the user before other work and propose: append the project root to the registry, scaffold `biblio/`, and gitignore it. Wait for explicit go-ahead before writing.
