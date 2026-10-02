---
name: dape-auto-coder
description: In Claude Code, run the coder side of a file-driven coding and review loop using reviewer_handoff.md and coder_handoff.md. Implement an authorized task, verify it, publish an artifact-bound handoff, and act on the review. Use when asked to start or resume this paired workflow; ordinary coding requests do not require the loop.
---

# dape-auto-coder — the coder's round

Three parties: the **owner** (decides; answers questions), the **coder** (this
session), the **reviewer** (a separate session that never talks to the coder
except through two files).

Before starting, resolve the target repository from the owner's context and
read its applicable instructions. Discover its task tracking, verification,
branch, remote and closure conventions from the repository and existing
authorization. Use its actual paths, commands and status vocabulary. If no
planning system exists, identify the authorized task in the handoff; do not
create a backlog or closure file just to satisfy this skill. Commit and push
only within the authorized scope and destination.

## Token-efficient handoffs

Prioritize token efficiency over prose in agent-to-agent handoffs, balancing
information passed against tokens used. Prefer terse structured records,
abbreviations, symbols, and evidence paths over narrative, repetition, or pasted
logs. Handoff files may use non-human notation when the recipient can interpret
it unambiguously; human-readable prose is optional. Preserve exact protocol
identity fields and verdict markers, actionable findings, evidence/results,
exceptions, and unresolved decisions. Templates below specify information, not
verbosity. In manual mode, apply this to owner-relayed blocks without introducing
handoff files.

## Claude Code execution

- Run this role in the current Claude Code session. Read applicable `CLAUDE.md`
  instructions and scoped repository rules. The owner runs the other role in
  a separate session; do not launch a subagent or agent team unless authorized.
- Use `Read`, `Glob` and `Grep` for inspection and `Bash` for repository commands
  when available. Use `Edit` or `Write` for authorized file changes.
- Use `AskUserQuestion` for missing decisions when available. Use Claude Code's
  permission flow for restricted actions; a question timeout is not permission.
- Prefer `Monitor` with `persistent: true` for handoff watches when exposed.
  Retain its task identifier, report only actual events, and stop this role's
  monitor when the owner pauses or stops the loop. If unavailable, use bounded
  30–60-second waits and file checks through the available execution tools.
- For background commands, retain the task identifier and output path; read
  the output and verify completion and exit status before publishing. Do not
  assume tools or background persistence exist in every Claude Code environment.

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
4. **Wait on your own** until `coder_handoff.md` exists and
   `reviewer_handoff.md` is absent, using the platform execution guidance above.
   The reviewer's removal is the completion signal, not the first appearance
   of its reply. Avoid busy polling; do not ask the owner whether the file
   has arrived or claim a background watch that is not running.
5. Read the completed reply and verify its **Task, Round, Base and Artifact**
   against the archived submission. A mismatched, incomplete or blocked
   reply is not approval: preserve it and resolve the specific discrepancy.
   Archive a matching reply and its evidence paths outside the working tree,
   then remove **only `coder_handoff.md`**. `REQUEST_CHANGES` → the next
   review round, using the last reviewed artifact as the comparison base.
   `APPROVE` → close only the approved task and code scope using the
   repository's conventions: the task record to done, acceptance on its
   closure records, the finding it answered, and dependents whose other
   prerequisites are also met. Commit the closure, push where authorized,
   then ask the owner what is next. A newer HEAD is not
   automatically approved; additional implementation changes need review. A metadata-only closure is not new
   implementation work.

The reviewer's side, for reference: waits for `reviewer_handoff.md`, reviews
the named artifact, writes and verifies `coder_handoff.md`, archives both,
confirms the incoming file is unchanged, removes `reviewer_handoff.md`, and
waits again. Only the recipient removes a consumed handoff. On interruption,
preserve this state and resume it instead of resetting the round.

## Owner rules (always in force)

- Follow the owner's product decisions and the repository's contribution
  rules. Do not carry assumptions about product naming or feature priorities
  from another project into this one.
- **Never invent product names.** Engineering identifiers are fine; anything
  a customer would read as a name is the owner's decision.
- **Questions and decisions requiring the owner go through `AskUserQuestion`.**
  Batch related decisions into one call; put the recommended option first.
  Make routine implementation choices independently and record material
  assumptions in the handoff. Do not ask what a careful colleague would
  decide alone. If the tool is unavailable, ask one concise question through
  the available interface rather than inventing a tool call or assuming
  approval. Existing authorization remains valid.
- Features over CI polish, unless the task is the gate itself.
- Discover the supported test setup, required environment and infrastructure
  isolation rules before running checks. Run integration suites only through
  the repository's supported entry point, one at a time, with their
  environment loaded first; two suites never share a mutable resource such
  as a test database concurrently.
- Preserve unrelated owner and other-session changes. Stage explicit paths
  (`git add <paths>`, never `-A`) and inspect their contents. Owner-modified
  files (agent instructions such as `CLAUDE.md`/`AGENTS.md`, task cards,
  review records, `.claude/`, `.agents/`) are never committed wholesale.
  Never commit the two handoff files or an uncommitted session log.
- Merged migrations are immutable; a mistake gets a corrective migration.
  Follow the repository's other compatibility policies when applicable.

## The work

1. **Claim** the authorized task using the repository's existing tracking
   convention, if any: its record to in-progress, with the actual session
   identity and base commit. One task per commit; additional commits are
   expected for review corrections. The relevant planning records travel in
   each commit when the repository keeps them.
2. **For a defect, reproduce before fixing**: a detached worktree at the base commit
   (`git worktree add --detach <tmp> <sha>`), the new test copied in (adapt
   the probe to the base revision when needed), run, record the exact failure
   text, remove the worktree after preserving the test/probe and its result outside it. For a
   new feature, use meaningful acceptance tests and negative cases rather
   than manufacturing a pre-fix bug. For a gate or drill task, dated evidence
   replaces the defect reproduction. In later rounds, fix blocking findings
   and necessary regressions; preserve accepted decisions and defer unrelated
   improvements. Record a reasoned disagreement rather than silently ignoring
   a finding or implementing it blindly.
3. **Implement**, then inspect the resulting diff and verify scripted edits
   with `grep` before trusting them; check every command's exit code
   explicitly (an empty tail is not success).
4. **Checks**: derive the required commands from repository instructions,
   build scripts and CI configuration, per changed component. Typically:
   - lint and the unit tests (with the race detector where the language has
     one) for every component touched;
   - the integration suite when the database or another external resource
     is touched;
   - cross-platform build/vet when the component ships for another OS, and
     its container/boundary tests when the image or its runtime changed;
   - the frontend's typecheck, lint, formatting, tests and build, plus the
     browser suite when pages changed;
   - documentation and contract tests when documents change. When a
     generator derives files from what changed (docs, clients, servers),
     regenerate and commit its output with the change, then run its drift
     check.

   Do not assume a language, package manager, directory layout or test
   target exists. Long chains (over ~10 minutes) run under a Monitor, never
   an untracked backgrounded `&`; otherwise retain and poll the tool's
   running session with interruptible waits. Do not publish while a
   required check is running.
5. **Records**: update existing task and review records as required by the
   repository — the task record (status, what was built, round notes) and a
   closure record (date, task/round, what was reproduced and how, the fix,
   the checks, the limitations), both in the task's commit. Record
   acceptance, and update any header naming the accepted commit, only after
   a matching approval. If no such records exist, retain this information in
   the handoff archive.
6. **Publishing gate, then commit and push where authorized**, explicit paths:
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
7. Preserve the round history in the handoff archive outside the working tree.
   If the repository keeps an uncommitted session log, append the round to
   it; it is a summary, not a replacement for the archived handoffs and evidence.

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
a branch name, `HEAD` or an unresolved placeholder for a commit. The reviewer
approves this artifact and task scope only, not subsequent code. Name the verdict being answered.

Then: what each finding got (cause, correction, where); pre-fix reproduction
(table: finding, how, what it showed); new tests; checks run (with results,
including anything flaky and whether it is pre-existing); decisions taken
without asking; limitations recorded; files; a suggested verification order;
the commit; archived submission and evidence locations. Say what did not run
and why, and identify any owner-approved publishing exception. Never claim a
check that did not pass. Keep handoff archives and test evidence outside the
working tree until the cycle is closed; a session summary does not replace
the original verdict or its evidence.

## Owner summary (chat)

Lead with the outcome and the commit. What each finding got, in a sentence
each. Evidence. Anything the owner must know or decide. Say the next step is the reviewer's and state whether the watch is active.
Only claim it is armed after starting the supported wait mechanism.
