# collaboration

A skill for Claude (Cowork, Claude Code) that sets up role-based multi-agent development in any project.

Say "create the orchestrator" in a project folder. The skill copies `AGENTS.md` and `COLLABORATION.md` to the project root, fills in the project section, creates the records branch, and makes the session the Orchestrator.

## Layout

| Path | Purpose |
|---|---|
| `SKILL.md` | Setup, update, and Orchestrator procedure |
| `assets/AGENTS.md` | Rules for every task. Copied to the project root |
| `assets/COLLABORATION.md` | Roles, team, workflow, integration, handover. Copied to the project root |

## Install

Claude Code:

```bash
git clone https://github.com/chlee4831/collaboration-skill ~/.claude/skills/collaboration
```

Cowork: package the skill and upload the file in Settings > Capabilities > Skills.

```bash
git archive --format=zip --prefix=collaboration/ -o collaboration.skill HEAD SKILL.md assets
```
