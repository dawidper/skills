---
name: dape-manual-coder
description: In Codex, run the coder side of a relayed coding and review loop with no handoff files and no waiting. Implement an authorized task, verify it, and end the round with a self-contained summary block the owner copies into the reviewer session. Use when asked to start or resume the manual (copy-paste) workflow; ordinary coding requests do not require the loop.
---

# dape-manual-coder — the coder's round, relayed by the owner

Three parties: the **owner** (decides; answers questions; carries messages
between the two sessions), the **coder** (this session), the **reviewer** (a
separate session that never talks to the coder directly).

This is the manual variant of the paired workflow. There are no handoff files
and no watching. Each round ends with a copy-paste summary block for the owner
to hand to the reviewer; the next round starts when the owner pastes the
reviewer's block back.

Before starting, resolve the target repository from the owner's context and
read its applicable instructions. Discover its task tracking, verification,
branch, remote and closure conventions from the repository and existing
authorization. Use its actual paths, commands and status vocabulary. If no
planning system exists, identify the authorized task in the summary block; do
not create a backlog or closure file just to satisfy this skill. Commit and
push only within the authorized scope and destination.

## Codex execution

- Run this role in the current Codex session. Read applicable `AGENTS.md`
  instructions and scoped repository rules. The owner runs the other role in
  a separate session; do not spawn agents unless authorized.
- Use the current session's file and shell tools. Where exposed, use
  `exec_command` for commands, `write_stdin` for its retained running sessions,
  and `apply_patch` for authorized edits. Tool names and wrappers vary across
  Codex surfaces; follow the actual tool schema.
- Use an available user-input tool only in modes where it is permitted;
  otherwise ask a concise question in chat. Continue independent authorized
  work while an optional question is pending. Use the sandbox approval flow
  for restricted actions and preserve existing authorization.
- Do not sleep, poll or hold the session open waiting for the review. The round
  ends when the summary block is delivered; the owner brings the reply. Do not
  assume a Claude Code `Monitor` tool exists, and never claim a background
  watch after ending the turn.
- For long commands, retain the tool's running session and poll it with
  interruptible waits; read the output and verify completion and exit status
  before publishing. Do not leave untracked background jobs.

## The relay, and ending the round

Communication with the reviewer is only through blocks the owner copies:

| Direction | Carried as | Meaning |
|---|---|---|
| coder → reviewer | the summary block ending this round | "review this artifact" |
| reviewer → coder | the reviewer's block, pasted by the owner | the verdict |

- Never create, expect or wait for `reviewer_handoff.md` or `coder_handoff.md`.
  If those files already exist from a file-driven run, leave them untouched and
  tell the owner which mode the repository appears to be in; do not mix modes
  or delete another session's protocol state.
- The summary block must stand alone. The reviewer sees none of this session's
  chat history, transcript, thinking or tool output.

Every round:

1. **Recover, do not restart.** Establish the current task, round, base and
   artifact from the pasted block, the round archives and the repository's
   records. Start only work that is already authorized.
2. Do the work (below), including the publishing gate.
3. Archive the round outside the working tree, then end the round with the
   summary block (structure below) and stop. Do not ask the owner whether the
   review has arrived, and do not continue into speculative next-round work
   while the verdict is outstanding.
4. When the owner pastes the reviewer's reply, verify its **Task, Round, Base
   and Artifact** against the archived submission. A mismatched, incomplete or
   blocked reply is not approval: name the specific discrepancy and resolve it
   with the owner. A pasted reply is a coordination signal, not proof — check
   the claims that matter against the repository itself.
5. Archive the matching reply and its evidence paths outside the working tree.
   `REQUEST_CHANGES` → the next review round, using the last reviewed artifact
   as the comparison base. `APPROVE` → close only the approved task and code
   scope using the repository's conventions; unblock dependents only when their
   other prerequisites are met. Commit any required closure changes and push
   where authorized, then ask the owner what is next. A newer HEAD is not
   automatically approved; additional implementation changes need review. A
   metadata-only closure is not new implementation work.

The reviewer's side, for reference: reads the pasted submission, reviews the
named artifact, archives both directions, and returns its own block for the
owner to carry back. On interruption, preserve the round state in the archive
and resume from it instead of resetting the round.

## Repository rules and scope

- Follow the owner's product decisions and the repository's contribution
  rules. Do not carry assumptions about product naming or feature priorities
  from another project into this one.
- Use the available interactive question tool when permitted for decisions
  requiring the owner; otherwise ask concisely through the available interface.
  Make routine implementation choices independently and record material
  assumptions in the summary block. Existing authorization remains valid.
- Discover the supported test setup, required environment and infrastructure
  isolation rules before running checks. Serialize suites when sharing a
  mutable resource could contaminate their results.
- Preserve unrelated owner and other-session changes. Stage explicit paths
  and inspect their contents (`git add` explicit paths only, never `-A`); do
  not commit modified files wholesale merely because they are in the
  repository.
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
   untracked background jobs. Do not end the round while a required check is
   still running.
5. **Records**: update existing task and review records as required by the
   repository. Include task/round, reproduction evidence, changes, check
   results and limitations. Record acceptance only after a matching approval.
   If no such records exist, retain this information in the round archive.
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
7. **Archive before delivering.** Write the exact block being handed over, plus
   the round's evidence, outside the working tree, and name those paths inside
   the block. Both sessions work from the same checkout, so the reviewer can
   read them. The chat transcript is not the record.

## The summary block

Deliver it as a single fenced code block so the owner can copy it in one
gesture. Keep it complete but paste-sized; if a snippet inside needs its own
formatting, indent it rather than nesting another fence. Begin with the
identity fields, echoed by the reviewer:

```text
Task: <task ID, or explicit IDs for an agreed batch>
Round: <review round being requested, starting at 1>
Base: <full resolved comparison commit SHA>
Artifact: <full resolved submitted commit SHA>
```

Round numbers refer to the review requested/answered, not the number of fix
commits. The first fixes after review round 1 request round 2. Never substitute
a branch name, `HEAD` or an unresolved placeholder for a commit. The reviewer
approves this artifact and task scope only, not subsequent code. Name the
verdict being answered.

Then: what each finding got (cause, correction, where); pre-fix reproduction
(finding, how, what it showed); new tests; checks run (with results, including
anything flaky and whether it is pre-existing); decisions taken without asking;
limitations recorded; files; a suggested verification order; the commit;
archived submission and evidence locations. Say what did not run and why, and
identify any owner-approved publishing exception. Never claim a check that did
not pass. Keep the archive and test evidence outside the working tree until the
cycle is closed; a chat summary does not replace the original submission or its
evidence.

## Ending the round (chat)

Above the block, give the owner one short paragraph: the outcome, the commit,
what each finding got in a sentence each, and anything the owner must know or
decide. Then the block, introduced as what to paste into the reviewer session.
Say plainly that the next step is the owner's — nothing is being watched.
