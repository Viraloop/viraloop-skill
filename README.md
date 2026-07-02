# Viraloop Claude Skill

An agent skill that lets Claude (and other SKILL.md-compatible agents) drive [Viraloop](https://viraloop.io): generate AI short-form social videos and schedule them to TikTok, Instagram and YouTube.

## Install

With the [skills CLI](https://github.com/anthropics/skills):

```bash
npx skills add Viraloop/viraloop-skill
```

Or manually: copy this repo's contents into your agent's skills directory (for Claude Code: `~/.claude/skills/viraloop/`).

## Requirements

- A Viraloop account with an API key: create one at https://viraloop.io/settings/developers
- Node.js 18+ for the CLI (the skill installs `viraloop` from npm on first use, or uses `npx`)

Set `VIRALOOP_API_KEY` in the agent's environment, or let the agent run `viraloop login` interactively.

## What is in here

- `SKILL.md`: the skill itself (setup, command reference, workflows, error handling)
- `references/api-reference.md`: full API reference
- `references/workflows.md`: step-by-step recipes

## Docs

- Developer docs: https://viraloop.io/developers
- OpenAPI spec: https://viraloop.io/openapi/v1.json
- Agent index: https://viraloop.io/llms.txt

## Note

This repo is generated from the main Viraloop codebase (`npm run generate:agents` + `npm run sync-skill`). Do not edit files here directly; changes will be overwritten on the next sync.
