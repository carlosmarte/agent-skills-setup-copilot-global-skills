<!-- gen-readme:auto -->
# agent-skills-setup-copilot-global-skills

An [agentskills.io](https://agentskills.io) skill repo for wiring up **GitHub Copilot Agent Skills** discovery on a developer machine or across a team. The skill lives under `.agents/skills/<name>/` and is mirrored into `.claude/skills/<name>` as a relative symlink so the Claude Code harness auto-discovers it. Third-party tooling skills (symlink mirroring, lockfile pruning, README generation) are vendored via `skills-lock.json` and git-ignored, so they don't appear below.

## Skills

| Skill | What it does |
|-------|--------------|
| [`setup-copilot-global-skills`](.agents/skills/setup-copilot-global-skills/SKILL.md) | Configures where GitHub Copilot Agent Skills live — personal (`~/.copilot/skills`, `~/.agents/skills`) or a custom centralized registry wired into VS Code's `chat.agentSkillsLocations` — and submits new skills upstream via the `gh` CLI. Covers macOS, Linux, and Windows path differences. |

## Install

### Per skill — `npx skills add`

Install a single skill into Claude Code:

```bash
npx skills add carlosmarte/agent-skills-setup-copilot-global-skills \
  --skill setup-copilot-global-skills -a claude-code
```

## Layout

```
.agents/skills/<name>/SKILL.md                          # source of truth for each skill
.claude/skills/<name> -> ../../.agents/skills/<name>    # relative symlink (harness-discovered)
```
