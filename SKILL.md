---
name: collaboration
description: Sets up and runs role-based multi-agent development in a project folder - an Orchestrator that forms a team of Architect, Developer, Reviewer, Tester and Integrator subagents working on feature branches and git worktrees. Use this whenever the user asks to create, start, or become an orchestrator ("오케스트레이터 생성", "오케스트레이터 만들어줘", "create the orchestrator"), to set up or update agent collaboration rules or an AGENTS.md in a project ("협업 규칙 세팅", "규칙 갱신"), or makes a development request in a project that already has a `.collaboration/` folder or a `COLLABORATION.md`, even if they never say "skill" or "collaboration".
---

# Collaboration

This skill installs two rule documents at a project's root and makes the current session its Orchestrator.

- `AGENTS.md`: rules for every task. Tools such as Claude Code and Codex load it on their own.
- `COLLABORATION.md`: roles, team sizing, workflow, integration, locks, handover, record layout. Only runs that do development work read it.

The rules live in the project, not in this skill, because subagents and other tools never see this skill. They only see the project's files. So after setup, the project's copies are the source of truth; do not answer rule questions from the templates in `assets/`.

`AGENTS.md` has two parts, split by a line that starts with `<!-- collaboration-skill:`. Above it is the project's own content, filled in at setup. Below it are this skill's rules, replaced as a whole on update.

## 1. Pick the mode

The project root is the git top level of the folder the user is working in, or that folder itself if it is not a repo yet.

| What you find at the root | Do |
|---|---|
| `AGENTS.md` with the marker line and no `TBD`, and `COLLABORATION.md` | "4. Become the Orchestrator". If the user asked to update the rules, "3. Update" |
| Neither file | "2. Set up" |
| An `AGENTS.md` without the marker line, or a `COLLABORATION.md` that did not come from this skill | Do not overwrite. Show the user what exists and ask: merge these rules into it, or stop |
| This skill's files with `TBD` still present | Resume "2. Set up" from the first unfinished step |

## 2. Set up

1. Inspect the project: git state, remotes, branches, nested repos, build and test commands, existing docs and conventions.
2. If the root is not a git repo, ask the user before `git init`. The workflow depends on branches and worktrees.
3. Copy the two files in `assets/` to the project root.
4. Create the records branch. Records get their own branch so that writing them never moves trunk; a trunk that moves during verification would invalidate the commit that was reviewed.
   - If `git ls-remote --heads origin collab` prints a branch, it already exists: run `git fetch && git worktree add .collaboration collab` and skip the block below.

   ```bash
   git worktree add --orphan -b collab .collaboration
   mkdir -p .collaboration/tasks .collaboration/handover .collaboration/locks
   echo "locks/" > .collaboration/.gitignore
   git -C .collaboration add .gitignore && git -C .collaboration commit -m "Set up records"
   git push -u origin collab
   ```

5. Add `.collaboration/` and `.worktrees/` to the root `.gitignore`.
6. Fill in "Project" in `AGENTS.md`: replace every `TBD` (`none` where nothing applies). Add the project's own documents (coding standards, design docs) as rows of the "Read before working" table, or delete the `TBD` row.
   - Ask the user only what cannot be detected, in one message, with a proposed default per item. Typical items: release branch, integration mode, protected paths, exclusive resources.
7. Commit `AGENTS.md`, `COLLABORATION.md`, and `.gitignore` to trunk by explicit path and push. This is the one direct trunk commit the rules allow; say so in the report.

Setup is finished when no `TBD` remains in `AGENTS.md`. Then continue with section 4.

## 3. Update

Use this when the user asks to bring the project's rules up to this skill's current version. The rule documents are tracked files, so they change through a feature branch like anything else.

1. Become the Orchestrator (section 4) and run the update as a text-only feature.
2. In the feature worktree, replace everything from the marker line down in `AGENTS.md` with the same part of `assets/AGENTS.md`, and replace `COLLABORATION.md` with `assets/COLLABORATION.md`. Leave everything above the marker line untouched.
3. Show the user a summary of what the diff changes and ask before integrating (`AGENTS.md` "User approval"). Rules govern every later task, so the user should see them change.
4. If the new rules move a record path, say so in the report. Do not move existing records unless asked.

## 4. Become the Orchestrator

1. Read the project's `AGENTS.md` and `COLLABORATION.md` in full.
2. If `.collaboration/` is missing on this machine, attach it (`COLLABORATION.md` "Records").
3. Register as Orchestrator (`COLLABORATION.md` "Locks and handover"). If the lock is held by someone else, follow the rule there; do not take it over on your own.
4. If the user's message carried a development request, start at "Workflow, 1. Intake". Otherwise report in the format of `AGENTS.md` "Reporting" and wait for a request.
