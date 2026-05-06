# AGENTS.md

Instructions for AI coding agents (Claude Code, Codex, Cursor, Copilot).

## Hard rules

All `oleg-koval/*` starter repos obey the rules in [`RULES.md`](./RULES.md).
Load it at the start of any session in a starter. See [`SKILL.md`](./SKILL.md) for the activation workflow (`olko:starter-rules`).

## What this repo is

Auto-generated index of all GitHub template repositories owned by @oleg-koval.
The README is rebuilt daily via GitHub Actions and on every push.

## Setup

Requires `gh` CLI authenticated:

```bash
gh auth status   # confirm authenticated
```

## Commands

- Rebuild index: `./discover.sh`
- Preview output: `./discover.sh /tmp/preview.md && cat /tmp/preview.md`

## Don't

- Don't hand-edit README.md — it gets overwritten by discover.sh.
- Don't add repos to the index manually — mark them as templates on GitHub instead:
  `gh repo edit oleg-koval/repo-name --template`
