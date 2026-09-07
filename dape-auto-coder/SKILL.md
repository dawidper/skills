---
name: dape-auto-coder
description: Run the coder side of a file-driven coding and review loop using reviewer_handoff.md and coder_handoff.md. Implement an authorized task, verify it, publish an artifact-bound handoff, and act on the review. Use when asked to start or resume this paired workflow; ordinary coding requests do not require the loop.
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
   `APPROVE` → close only the approved task and code scope using the
   repository's conventions; unblock dependents only when their other
   prerequisites are met. Commit any required closure changes and push
   where authorized, then ask the owner what is next. A newer HEAD is not
   automatically approved; additional implementation changes need review. A metadata-only closure is not new
   implementation work.

The reviewer's side, for reference: waits for `reviewer_handoff.md`, reviews
the named artifact, writes and verifies `coder_handoff.md`, archives both,
confirms the incoming file is unchanged, removes `reviewer_handoff.md`, and
waits again. Only the recipient removes a consumed handoff. On interruption,
preserve this state and resume it instead of resetting the round.

## Repository rules and scope

- Follow the owner's product decisions and the repository's contribution
  rules. Do not carry assumptions about product naming or feature priorities
  from another project into this one.
- Use the available interactive question tool when permitted for decisions
  requiring the owner; otherwise ask concisely through the available interface.
  Make routine implementation choices independently and record material
  assumptions in the handoff. Existing authorization remains valid.
- Discover the supported test setup, required environment and infrastructure
  isolation rules before running checks. Serialize suites when sharing a
  mutable resource could contaminate their results.
- Preserve unrelated owner and other-session changes. Stage explicit paths
  and inspect their contents; do not commit modified files wholesale merely
  because they are in the repository. Never commit the two handoff files.
- Follow the repository's compatibility and migration policies when applicable.

## The work

1. **Claim** the authorized task using the repository's existing tracking
   convention, if any. Record the actual session identity and base commit
   where required. Keep commits scoped to the task; additional commits are
   expected for review corrections. Include relevant planning updates when
   the repository requires them.
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
3. **Implement**, then inspect the resulting diff and verify scripted edits;
   check every command's exit code explicitly (an empty tail is not success).
4. **Checks**: derive the required commands from repository instructions,
   build scripts and CI configuration. Run checks appropriate to the changed
   components and their risk, including integration, compatibility, browser
   or documentation checks when applicable. Do not assume a language, package
   manager, directory layout or test target exists. Retain and poll the tool's
   running session for long commands with interruptible waits; do not leave
   untracked background jobs. Do not publish while a required check is running.
5. **Records**: update existing task and review records as required by the
   repository. Include task/round, reproduction evidence, changes, check
   results and limitations. Record acceptance only after a matching approval.
   If no such records exist, retain this information in the handoff archive.
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
   Update an existing session log only when required by the repository.

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
each. Evidence. Anything the owner must know or decide. End with "next step
is the reviewer's" and that the watch is armed.
