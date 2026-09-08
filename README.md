# Skills

The paired skills for Dape's coding and review workflow, in two modes:

- **Coder**: implements and verifies scoped tasks, hands over the artifact, and acts on the review.
- **Reviewer**: independently reviews the submitted artifact, returns its verdict, and takes the next round.

**Automatic (`dape-auto-*`)** relays through two files and each session waits
for the other. **Manual (`dape-manual-*`)** has no handoff files and no
waiting: each round ends with a self-contained block that the owner copies into
the other session. Pick one mode per pair; do not mix them in one checkout.

## Automatic mode

The coder writes `reviewer_handoff.md` in the repository being worked on. The reviewer writes `coder_handoff.md`, verifies and archives its reply, then removes the incoming handoff. The coder consumes the reply only when it exists and the incoming handoff is gone. Both files identify the task, review round, base commit and artifact commit.

## Manual mode

Nothing is written to the repository root and neither session waits. The coder
implements, verifies and commits, then ends its turn with a fenced summary
block naming the task, round, base and artifact commits. The owner pastes that
block into the reviewer session; the reviewer reviews the named artifact and
ends with its own fenced verdict block, which the owner pastes back. Each side
archives the block it received, the block it sent and its evidence outside the
working tree, so the record does not depend on either chat transcript. Use this
mode when the two sessions cannot share a checkout conveniently, when a session
cannot be left waiting, or when you want to see and control every exchange.

Both skills discover the target repository's instructions, task tracking, check commands and contribution conventions. They do not require a particular language, directory layout, planning system or branch name. Commit and push destinations follow the owner's authorization and repository conventions.

These files are standalone skill sources. Editing this repository does not automatically update copies installed elsewhere.

## Dedicated versions

Choose one version per role for each client. Each folder is self-contained;
copy the whole skill folder from the matching harness directory, keeping its
name. Replace any older generic installation of the same skill to avoid
duplicate discovery.

| Client | Mode | Coder | Reviewer | Repository install directory | Personal install directory |
| --- | --- | --- | --- | --- | --- |
| Claude Code | Automatic | [Coder](claude-code/dape-auto-coder/SKILL.md) | [Reviewer](claude-code/dape-auto-reviewer/SKILL.md) | `.claude/skills/` | `~/.claude/skills/` |
| Claude Code | Manual | [Coder](claude-code/dape-manual-coder/SKILL.md) | [Reviewer](claude-code/dape-manual-reviewer/SKILL.md) | `.claude/skills/` | `~/.claude/skills/` |
| Codex | Automatic | [Coder](codex/dape-auto-coder/SKILL.md) | [Reviewer](codex/dape-auto-reviewer/SKILL.md) | `.agents/skills/` | `~/.agents/skills/` |
| Codex | Manual | [Coder](codex/dape-manual-coder/SKILL.md) | [Reviewer](codex/dape-manual-reviewer/SKILL.md) | `.agents/skills/` | `~/.agents/skills/` |

In Claude Code, invoke `/dape-auto-coder` or `/dape-manual-coder` with the
authorized task, and `/dape-auto-reviewer` or `/dape-manual-reviewer` for the
review side. In Codex, use `$dape-auto-coder`, `$dape-manual-coder`,
`$dape-auto-reviewer` or `$dape-manual-reviewer` in your prompt. All remain
eligible for automatic selection when the request matches their descriptions.
Installing both modes side by side is fine; name the one you want so the
request does not match the other.

Claude Code versions include guidance for its file tools, `AskUserQuestion` and
permission flow; the automatic versions add `Monitor` when available, and the
manual versions explicitly forbid watches and polling. Codex versions include
`agents/openai.yaml` metadata and guidance for session tools, sandbox approvals
and mode-dependent questions; the automatic versions add interruptible waits.
No version pins a model or assumes every client exposes the same tools.

Run the two roles in separate sessions. Either client can fill either role,
including a mixed Claude Code/Codex pair, and the two sides do not have to run
the same client. Both sides of a manual pair still assume one shared repository
checkout, since the blocks name commits and local evidence paths. Use one coder
and one reviewer per checkout; concurrent automatic pairs need separate
checkouts to avoid sharing the two handoff filenames. Pausing an automatic
session must preserve its handoff state; a stopped session is not an active
watch. A manual session holds no state between rounds beyond its archives, so
it can be closed and resumed freely.

When maintaining these copies, apply protocol changes to all four versions of
the affected role; keep platform tool differences in their execution sections
and mode differences in the relay and waiting sections. They are not generated
or automatically synchronized.

Platform packaging and invocation follow the official
[Claude Code skills documentation](https://code.claude.com/docs/en/skills) and
[Codex skills documentation](https://learn.chatgpt.com/docs/build-skills).
Claude monitoring guidance follows its
[tools reference](https://code.claude.com/docs/en/tools-reference).
