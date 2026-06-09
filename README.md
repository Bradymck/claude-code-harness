# Claude Code Autonomous Dev Harness

A portable toolkit for Claude Code that turns feature ideas into shipped PRs with minimal supervision. Five files, zero dependencies beyond Claude Code + `gh` CLI.

## The Pipeline

```
idea ──▶ /bootstrap ──▶ /prd ──▶ /ralph ──▶ ralph-executor ──▶ /pr ──▶ merged
         (new project   (spec     (convert    (autonomous       (git
          setup only)    it)       to JSON)    build agent)      hygiene)
```

1. **`/bootstrap`** — (new/unfamiliar projects only) Structured discovery → codebase exploration → deliberation → coordination docs → agents launched. Skip for projects you already know.
2. **`/prd`** — Asks 3-5 clarifying questions with lettered options ("1A, 2C, 3B"), then writes a PRD with small, verifiable user stories to `tasks/prd-<feature>.md`.
3. **`/ralph`** — Converts the PRD to `prd.json`: stories sized to one context window each, dependency-ordered, every criterion machine-checkable.
4. **`ralph-executor`** — Subagent that implements one story end-to-end: read → plan → implement → typecheck → test → commit. Signals `RALPH_DONE` / `RALPH_BLOCKED` / `RALPH_ERROR`.
5. **`/pr`** — Nine-step git hygiene checklist: identity check, build pass, explicit file staging (never `git add .`), feature branch, conventional commit, `gh pr create`, returns PR URL.

## Install

One-liner:

```bash
git clone https://github.com/Bradymck/claude-code-harness && mkdir -p ~/.claude/skills && cp -r claude-code-harness/skills/* ~/.claude/skills/
```

Or step by step:

```bash
# Skills (global — available in every project)
mkdir -p ~/.claude/skills
cp -r skills/* ~/.claude/skills/

# Agent (per-project, or put in ~/.claude/agents/ for global)
mkdir -p <project>/.claude/agents
cp agents/ralph-executor.md <project>/.claude/agents/
```

Restart Claude Code (or start a new session). Verify with `/bootstrap`, `/prd`, `/ralph`, `/pr` appearing in the slash-command list.

## Contents

```
harness-portable/
├── README.md                      (this file)
├── skills/
│   ├── bootstrap/SKILL.md         project bootstrap wizard
│   ├── prd/SKILL.md               PRD generator
│   ├── ralph/SKILL.md             PRD → prd.json converter
│   └── pr/SKILL.md                PR with full git hygiene
└── agents/
    └── ralph-executor.md          autonomous implementation subagent
```

## Design Principles

- **Stories fit one context window.** Ralph spawns fresh instances with no memory — if a story can't be described in 2-3 sentences, split it.
- **Acceptance criteria are machine-checkable.** "Typecheck passes" not "works correctly". UI stories require browser verification.
- **Dependencies first.** Schema → backend → UI → aggregate views. A story never depends on a later one.
- **Never `git add .`** — stage files by name to avoid committing secrets.
- **PRs always, merges never.** The agent ships PRs; the human merges.

## Customizing

Each SKILL.md is self-contained markdown — edit freely:
- `pr/SKILL.md`: hardcode your git identity in step 1 to make it self-enforcing
- `ralph-executor.md`: add your project's auth model, conventions, or gotcha docs under "Project Conventions"
- `bootstrap/SKILL.md`: swap the generic deliberation phase for your own review agent if you have one

## Credits

This harness builds on prior work — go star the originals:

- **`/prd`** is adapted from [Ryan Carson's ai-dev-tasks](https://github.com/snarktank/ai-dev-tasks) (`create-prd.md`, Apache-2.0) — the clarifying-questions flow, PRD structure, and `tasks/prd-[feature-name].md` convention are his.
- **`/ralph`** and the `prd.json` format (`userStories` / `passes` / `notes`, one-story-per-context-window sizing) are adapted from [Ryan Carson's ralph](https://github.com/snarktank/ralph) (MIT).
- The **Ralph technique** itself — running a coding agent in a loop with fresh context per iteration, memory in files and git — was created and named by [Geoffrey Huntley](https://ghuntley.com/ralph/).
- `/bootstrap` and `/pr` are original to this repo.

## License

MIT (this repo). The `/prd` skill is a derivative of Apache-2.0 licensed work from [ai-dev-tasks](https://github.com/snarktank/ai-dev-tasks) — see [NOTICE](NOTICE).
