---
name: dape-auto-coder
description: The ccc coder workflow — claim a backlog task, verify before publishing, keep each commit scoped to one task with its planning rows, exchange artifact-bound handoffs through reviewer_handoff.md and coder_handoff.md, act on the verdict, and ask the owner what is next. Use whenever working a backlog task in this repository.
---

# dape-auto-coder — the coder's round

Three parties: the **owner** (decides; answers questions), the **coder** (this
session), the **reviewer** (a separate session that never talks to the coder
except through two files). The backlog (`platform/BACKLOG.md`) is the status
authority; `platform/EXECUTION.md` defines claim, verification and closure.

## The two files, and waiting

Communication with the reviewer is only through two files at the repository
root, never committed (`git add` explicit paths only, never `-A`):

| File | Written by | Read by | Meaning |
|---|---|---|---|
| `reviewer_handoff.md` | coder | reviewer | "review this" — the reviewer removes it only after verifying and archiving its completed reply |
| `coder_handoff.md` | reviewer | coder | the verdict — the coder verifies and archives it before removing it |

Every round:

1. **Recover, do not clear.** Inspect both files and the previous round's
   records. If `reviewer_handoff.md` exists, wait: it belongs to the reviewer
   to consume, even if a reply has started appearing. If only
   `coder_handoff.md` exists, consume it through step 5. If neither exists,
   recover the current task/round from the archives and planning records;
   start only the work already authorized. Never blindly delete both files.
2. Do the work (below), including the publishing gate.
3. Publish the completed `reviewer_handoff.md` (structure below), read it
   back, and archive that exact submission outside the working tree. Use
   atomic publication where supported so its appearance is not a partial
   draft. Give the owner a short summary in chat. Do not overwrite or alter
   a submitted handoff while the reviewer owns it.
4. **Wait on your own** until the reply exists **and the incoming handoff
   is gone**. With a persistent Monitor:
   `cd <repo> && until [ -f coder_handoff.md ] && [ ! -e reviewer_handoff.md ]; do sleep 30; done; echo ready`
   (`persistent: true`). The reviewer's removal is the completion signal,
   not the first appearance of its reply. If Monitor is unavailable, use
   the product's interruptible wait mechanism in 30–60-second intervals and
   check the same condition. Avoid busy polling; do not ask the owner
   whether the file has arrived or claim a background watch that is not running.
5. Read the completed reply and verify its **Task, Round, Base and Artifact**
   against the archived submission. A mismatched, incomplete or blocked
   reply is not approval: preserve it and resolve the specific discrepancy.
   Archive a matching reply and its evidence paths outside the working tree,
   then remove **only `coder_handoff.md`**. `REQUEST_CHANGES` → the next
   review round, using the last reviewed artifact as the comparison base.
   `APPROVE` → close only the approved task and code scope (DONE row,
   acceptance on closure rows, its finding, dependents whose other
   prerequisites are also met), commit the closure, push, and ask the owner
   what is next. A newer HEAD is not automatically approved; additional
   implementation changes need review. A metadata-only closure is not new
   implementation work.

The reviewer's side, for reference: waits for `reviewer_handoff.md`, reviews
the named artifact, writes and verifies `coder_handoff.md`, archives both,
confirms the incoming file is unchanged, removes `reviewer_handoff.md`, and
waits again. Only the recipient removes a consumed handoff. On interruption,
preserve this state and resume it instead of resetting the round.

## Owner rules (always in force)

- **Never invent product names.** Engineering identifiers are fine; anything
  a customer would read as a name is the owner's decision.
- **Questions and decisions requiring the owner go through `AskUserQuestion`.**
  Batch related decisions into one call; put the recommended option first.
  Make routine implementation choices independently and record material
  assumptions in the handoff. Do not ask what a careful colleague would decide
  alone. If the tool is unavailable, ask one concise question through the
  available interface rather than inventing a tool call or assuming approval.
- Features over CI polish, unless the task is the gate itself.
- The integration suite runs **only** via `make test-integration` in
  `observability/` (`-p 1`), one at a time, with the environment sourced
  first; the browser suite and the Go integration suite never share the
  database concurrently.
- Owner-modified working-tree files (`CLAUDE.md`, `observability/AGENTS.md`,
  `observability/CLAUDE.md`, task cards, `platform/README.md`, `reviews/*`,
  untracked cards, `merge.md`, `.agents/`, `.claude/`) are never committed
  wholesale. `night_work.md`, `reviewer_handoff.md`, `coder_handoff.md` are
  never committed.
- Merged migrations are immutable; a mistake gets a corrective migration.

## The work

1. **Claim**: the backlog row → `IN_PROGRESS` (owner "the Claude Code
   session", base commit). One task per commit; additional commits are
   expected for review corrections. The relevant rows travel in each commit.
2. **For a defect, reproduce before fixing**: a detached worktree at the base commit
   (`git worktree add --detach <tmp> <sha>`), the new test copied in (adapt
   removed APIs by sed), run, record the exact failure text, remove the
   worktree after preserving the test/probe and its result outside it. For a
   new feature, use meaningful acceptance tests and negative cases rather
   than manufacturing a pre-fix bug. For a gate or drill task, dated evidence
   replaces the defect reproduction. In later rounds, fix blocking findings
   and necessary regressions; preserve accepted decisions and defer unrelated
   improvements. Record a reasoned disagreement rather than silently ignoring
   a finding or implementing it blindly.
3. **Implement**, then verify scripted edits with `grep` before trusting
   them; check every command's exit code explicitly (an empty tail is not
   success).
4. **Checks**, by area:
   - `observability/`: `make lint`, `go test -race ./...`, `make test-integration`
     when the database is touched, `go test ./internal/docs` when documents are.
   - `ccc-agent/`: `go test -race ./...`, `make lint`, `GOOS=linux
     .tools/golangci-lint-<v> run ./...`, `GOOS=linux go vet ./...` (and
     `-tags docker ./internal/agent/`), `make test-boundary` when the image,
     worker, executor or channel changed (Docker; ~2 min after the build).
   - web: `npm run typecheck && npm run lint && npm run format:check && npm test && npm run build`;
     the Playwright suite when the portal's pages changed.
   - docs: `npm run docs:generate`, `docs:check`, `docs:validate` — exit codes.
   Long chains (>10 min) run under a Monitor, never an untracked backgrounded
   `&`; if unavailable, retain and poll the tool's running session with
   interruptible waits. Do not start publishing while any required check is running.
5. **Rows**: the backlog row (status, what was built, round notes) and a
   closure record in `platform/REVIEW_DISPOSITIONS.md` (date, task/round,
   what was reproduced and how, the fix, the checks, the limitations), both
   in the task's commit; update both headers' commit only at acceptance.
6. **Publishing gate, then commit and push** to `main`, explicit paths:
   - Required checks must have finished successfully before committing the
     implementation, pushing it or announcing review readiness. A failed,
     skipped or unavailable required check needs a documented owner-approved
     exception before publishing; no exception is inferred from "probably flaky".
   - Preserve each command, exit code, tested source snapshot/revision and
     evidence path. A clean rerun does not explain an earlier failure. Resolve
     unexplained failures or obtain an explicit exception; distinguish proven
     pre-existing failures from suspicions and do not silently expand the task
     to fix unrelated code.
   - Inspect the staged diff and confirm the implementation matches what was
     tested. Relevant edits after testing invalidate the affected checks;
     rerun them. Map pre-commit test evidence to the resulting artifact SHA,
     distinguishing tests of a working snapshot from tests of a commit.
   - Use accurate co-author/model attribution and the actual session URL when
     available, following repository conventions. Do not hardcode a model name,
     invent a session URL or leave a placeholder trailer.
7. Append the round to `night_work.md` (uncommitted history).

## reviewer_handoff.md

Begin with these identity fields, echoed by the reviewer:

```text
Task: <task ID, or explicit IDs for an agreed batch>
Round: <review round being requested, starting at 1>
Base: <full resolved comparison commit SHA>
Artifact: <full resolved submitted commit SHA>
```

Round numbers refer to the review requested/answered, not the number of fix
commits. The first fixes after review round 1 request round 2. Never substitute
`main`, `HEAD` or an unresolved placeholder for a commit. The reviewer approves
this artifact and task scope only, not subsequent code. Name the verdict being answered.

Then: what each finding got (cause, correction, where); pre-fix reproduction
(table: finding, how, what it showed); new tests; checks run (with results,
including anything flaky and whether it is pre-existing); decisions taken
without asking; limitations recorded; files; a suggested verification order;
the commit; archived submission and evidence locations. Say what did not run
and why, and identify any owner-approved publishing exception. Never claim a
check that did not pass. Keep handoff archives and test evidence outside the
working tree until the cycle is closed; `night_work.md` is a summary, not a
replacement for the original verdict or its evidence.

## Owner summary (chat)

Lead with the outcome and the commit. What each finding got, in a sentence
each. Evidence. Anything the owner must know or decide. End with "next step
is the reviewer's" and that the watch is armed.
