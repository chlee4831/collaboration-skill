# Collaboration rules for development work

- Read the root `AGENTS.md` first. Common rules, user approval, reporting, and writing are there.
- The Orchestrator and every run that holds a role read this whole document.

## Roles

| Role | Does | Produces | Tier |
|---|---|---|---|
| Orchestrator | Takes requests, picks the team, orders work, owns all contact with the user | Task record, final report | high |
| Architect | Decides design: scope, split, interfaces, acceptance criteria. Classifies approval | Design, verdicts with reasons | top |
| Developer | Implements in its own branch and worktree, self-checks, cleans up | Feature commits, handoff | high |
| Reviewer | Reviews a fixed commit: design match, defects, impact, protected paths, test sufficiency | Review report (`pass`, `changes required`) | high |
| Tester | Runs acceptance and regression tests on a fixed commit | Test report (`pass`, `fail`, `unverified`) | mid |
| Integrator | Merges one verified feature into trunk under the integrator lock | Merge commit | high |

- Roles are responsibilities. One run may hold several, within "Invariants".
- Only the Orchestrator lasts for the whole task. Other roles are subagents spawned per stage, named `<Role>: <target>`.
- Agents keep no private state. Everything needed to resume is in the task folder.
- The Architect does not edit product files either. Defects go to the Developer.
- Tiers are top, high, mid, light; map them to the models your tool offers. The Orchestrator may adjust per task and records it.
  - Mechanical chores (record updates, commits, comparing outputs) go to light.
  - A lower tier never lowers test items, criteria, or approval boundaries.
  - If the tool cannot choose a model per subagent, ignore tiers and note it in the task record.

## Team

Start with the smallest team that satisfies "Invariants". Add a role when the risk it guards against is present.

| Team (besides Orchestrator) | When |
|---|---|
| Solo: one run edits, self-checks, integrates | Text only (docs, comments, messages). No code or behavior change |
| Pair: Developer + one verifier (reviews, tests, integrates) | Code changes inside one module, no design choice, automated tests settle it |
| Full: Architect, Developer per part, Reviewer, Tester, Integrator | A design choice is open; interface, data format, or criteria change; spans modules or repos |

- Judge by: what breaks if it is wrong, whether anything is still undecided, how it can be verified.
- In-between teams are fine. When unsure, go larger.
- A team may grow mid-task, never shrink. Anyone who sees scope growth or a design problem tells the Orchestrator.
- Overlap with another feature (same files or same meaning) is solved by order, not team size: the later work waits.

## Invariants

These hold for every team.

1. A run that changed code or behavior is never the only verifier of that change. Review, test verdict, and integration belong to other runs. Verifiers do not fix what they find; it goes back to the Developer. Text-only changes are the exception.
2. Acceptance criteria are written before implementation. Review and tests run on a fixed commit, and that commit is what gets integrated. New commits mean re-verifying what changed.
3. `unverified` is not `pass`. A required item left unverified blocks integration unless the user explicitly approves.
4. Integration is done one feature at a time, only by the holder of the integrator lock.
5. Verdicts belong to their role. The Orchestrator does not replace or overturn them.
6. Team size never changes what needs user approval (`AGENTS.md` "User approval").

## Branches and worktrees

The core rules are in `AGENTS.md` "Common rules". Details:

- The Developer creates the feature from the trunk tip:

  ```bash
  git fetch
  git worktree add --no-track .worktrees/<feature> -b feature/<feature> origin/<trunk>
  git -C .worktrees/<feature> push -u origin feature/<feature>
  ```

- Verifiers work read-only in the feature's worktree, after confirming HEAD is the target commit and the tree is clean.
- After integration: keep the branch, remove the worktree (`git worktree remove`), commit nothing more to it. Follow-up work gets a new branch.
- Check whether a feature is integrated by its trailer: `git log <trunk> --grep "Feature-Branch: feature/<feature> "`.
- Several repos: worktrees go in `.worktrees/<feature>/<repo>`. Use the same branch name in each repo a feature touches, integrate them together, and run git as `git -C <repo>`.
- If git reports a missing or broken worktree (the project folder is mounted at a different path than where the worktree was made), run `git worktree repair` from the main checkout first.

## Workflow

Every file of a feature goes in its task folder, `.collaboration/tasks/<feature>/`.

### 1. Intake (Orchestrator)

- Register as Orchestrator if this session has not already ("Locks and handover").
- Fetch. Check open task folders, branches, and worktrees for overlap.
- Write the task record, `.collaboration/tasks/<feature>/task.md`:

  ```
  # <feature>
  Request: "<user's words, verbatim>"
  Scope: files allowed / excluded
  Acceptance criteria:
  Team: role → run, tier. Why this size
  Approval: pre-approved | needs approval. Reason
  Base: <trunk commit>   Branch: feature/<feature>
  ## Log
  <date>: action, result (commit, verdict), next step
  ```

### 2. Design

- Without an Architect, the task record is the design.
- With an Architect: the Architect writes scope, split, interfaces, and acceptance criteria, not implementation, in the task record or `design.md` beside it.
- No product code until the Architect approves and the Orchestrator says go. Reading code and preparing tests is fine.
- Items only the user can decide go to the user through the Orchestrator, with the Architect's recommendation.

### 3. Implement (Developer)

- Self-check before every commit (build, related tests), even when a Tester exists.
- Before requesting review, clean up only what this feature changed: debug output, temporary code, dead code, duplication.
  - No behavior change. Keep it in a separate `Refactor ...` commit.
  - A bug found on the way gets its own commit and is flagged to the Reviewer.
- Request review with `<short-hash>-handoff.md`: commits, what changed, self-check results, what was not verified. A Solo run records its self-check here; it is the report for integration.

### 4. Verify (Reviewer, Tester)

- Reviewer: every finding has a location, a reason, and the expected impact.
- Tester: does not edit sources or change criteria. Checks per-item results, not only the exit code. Environment errors and waiting on the user are not `pass`.
- Reports are `<short-hash>-review.md` and `<short-hash>-test.md`: target commit, commands, verdict, evidence, open items.
- A fix means a new commit, a new review, and re-running the affected tests.

### 5. Integrate (Integrator)

1. The Developer fetches, merges `origin/<trunk>` into the feature, and has the changed parts re-verified. The result is the candidate commit.
2. Acquire the integrator lock. Fetch. Confirm local trunk equals `origin/<trunk>`, the candidate contains it, the diff stays in scope, and the reports name the candidate commit.
3. Get approval if required (`AGENTS.md` "User approval").
4. Write `merge-message.txt` in the task folder and merge in the main checkout:

   ```bash
   git merge --no-ff -F .collaboration/tasks/<feature>/merge-message.txt feature/<feature>
   git rev-parse '<trunk>^2' '<trunk>^{tree}'   # must equal the candidate commit and its tree
   ```

   - Message: one-line summary, short body, last line `Feature-Branch: feature/<feature> @ <short-hash>`.
   - If either value differs, stop and do not push.
5. Fetch again. If `origin/<trunk>` has not moved, push.
6. Release the lock, remove the worktree, and add the result to the task record's log.

- `PR` mode: instead of steps 4–5, open a PR from the candidate commit and stop. The user merges it.
- Failure before push: reset local trunk to the pre-merge commit and continue on the feature. No approval needed.
- Failure after push: fix on `feature/<feature>-fix-<n>` and integrate again.

## Locks and handover

- A lock is the file `.collaboration/locks/<name>` holding `<id> <time>`. Names: `orchestrator`, `integrator`, and one per exclusive resource. Locks are local to the machine and untracked.

  ```bash
  mkdir -p .collaboration/locks
  ( set -C; echo "<id> $(date '+%F %T')" > .collaboration/locks/<name> )   # fails if held
  ```

- An id is a short role name plus 4 random hex digits, such as `orch-3fa9` or `integ-4d1b`.
- Release your own lock by deleting it. A silent holder is not grounds to remove its lock.
- The Orchestrator keeps its lock while its session serves the user. When the session ends with nothing pending, it releases the lock. Pending means a feature in progress, an awaited user decision, a live subagent, or a held lock.
- To pause or hand over with work pending: keep the lock, update the task records, and write `.collaboration/handover/<id>.md`:
  - Status: `waiting` or `handed over`.
  - Per feature: branch, tip commit, next step.
  - Pending user questions and answers, verbatim.
  - Live subagents, held locks, uncommitted work.
- A session asked to be Orchestrator while the lock is held:
  - If the holder's handover file says `handed over`: overwrite the lock with its own id, check the handover against reality (branch tips, locks, worktrees), record the differences, and mark the file `taken over by <id>`.
  - Otherwise: leave the lock and tell the user. Only the user can name a successor without a handover.

## Records

`.collaboration/` is a worktree of the branch `collab`. If it is missing on this machine: `git fetch && git worktree add .collaboration collab`.

```
.collaboration/
├── tasks/<feature>/
│   ├── task.md                    assignment and log
│   ├── design.md                  only with an Architect
│   ├── <short-hash>-handoff.md    Developer's review request
│   ├── <short-hash>-review.md
│   ├── <short-hash>-test.md
│   └── merge-message.txt
├── handover/<id>.md
├── issues.md
└── locks/                         untracked
```
