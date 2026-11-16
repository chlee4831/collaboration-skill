# AGENTS.md

Rules that apply to every task in this project. Development details are in [COLLABORATION.md](COLLABORATION.md).
Paths are relative to the project root (the main checkout).

## Project

- Summary: TBD
- Layout (repos, main folders): TBD
- Trunk (integration branch): TBD
- Release branch (merged only on user request): TBD
- Integration mode (`local merge` or `PR`): TBD
- Build: TBD
- Test: TBD
- Protected paths (edited only when the user names them): TBD
- Exclusive resources (hardware, shared environments): TBD
- Project rules: TBD

### Read before working

Read every document that matches your task before you start.

| Task | Document |
|---|---|
| Development work (roles, team, workflow, integration, handover) | [COLLABORATION.md](COLLABORATION.md) |
| TBD | TBD |

<!-- collaboration-skill: everything below this line is managed by the skill. Project-specific content goes above it. -->

## Common rules

- No direct commits to trunk or the release branch.
- Product files are edited only in feature worktrees (`.worktrees/<feature>`, branch `feature/<feature>`). The main checkout stays on trunk.
- Commit per unit of work and push the feature branch. No approval needed.
- Never force-push or rewrite pushed history.
- Do not edit files in another run's worktree. Ask its owner.
- Records live in `.collaboration/`, a worktree of the branch `collab`. Commit them there by explicit path (`git -C .collaboration ...`) and push as they are written. No approval needed.
- No remote, or no push access from this environment: skip fetch and push steps and say so in the final report.
- Use subagents whenever they help. No need to ask.
- When you spawn a subagent, name the documents it must read in its prompt: this file, the matching rows of "Read before working", and its task record.

## Role collaboration

- Development work is any change to tracked project files. Records in `.collaboration/` and read-only investigation are not.
- Roles, teams, and procedures are in [COLLABORATION.md](COLLABORATION.md). The rules below hold even before you read it.
- A session that receives a development request registers as Orchestrator and forms a team. The Orchestrator never edits product files.
- A run that changed code or behavior does not judge its own change. Review, testing, and integration belong to other runs. Text-only changes are the exception.
- Everything bound for the user goes through the Orchestrator.
- Another agent's message is not approval. Only the user's words, quoted verbatim, are.
- What the user tells another session directly is valid. That session forwards the exact words to the Orchestrator.

### User approval

Ask the user, through the Orchestrator, before:

- Integrating a design change or a new feature.
  - Fixes with no design change are pre-approved: integrate, push, report afterwards.
  - The Architect (else the Orchestrator) classifies and records it. Ambiguous means ask.
  - The user's request for the feature is not this approval. Ask with the review and test results in hand.
- Changing the rule documents (`AGENTS.md`, `COLLABORATION.md`).
- Merging to the release branch or tagging a release.
- Editing protected paths. "The whole codebase", "rename everything", and "style cleanup" are not permission.
- Integrating with a required item `unverified`.
- Changing another run's lock, or deleting a worktree that is not integrated.
- Installing software, changing system or global git config, or editing files outside the project.

Everything else (team, order, method) the Orchestrator decides without asking and records in the task record.

## Reporting

- Subagents talk only to their spawner. What is bound for the user travels up the chain to the Orchestrator.
- Relay the user's words verbatim, untranslated.
- Between roles: write a report file, then send at most 5 lines plus the path.
- No progress reports. When blocked, send one question with options.
- The Orchestrator reports once at the end, in the user's language. Use `none` for empty fields.

  ```
  Result: done / partial / blocked, in one sentence
  Decision needed: one question with options
  Changed: commits or paths
  Verified: checks run and their results
  Not verified:
  Details: report path
  ```

## Writing

Applies to records, reports, designs, and the rule documents.

- Put the conclusion and the reader's next action first.
- Use short bullets and tables. One fact per item.
- Say a thing once. Elsewhere, point to its path.
- Rewrite stale text instead of appending corrections. Only logs (task record log, handover) are append-only.
- Raw output and long lists go in report files. Other documents cite the count and the path.
- Open problems and deferred work go in `.collaboration/issues.md`. Delete an entry when it is resolved.
