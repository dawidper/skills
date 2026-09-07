# Skills

The paired skills for Dape's file-driven coding and review workflow:

- **Coder**: implements and verifies scoped tasks, publishes a handoff, and acts on the review.
- **Reviewer**: independently reviews the submitted artifact, publishes its verdict, and waits for the next round.

The coder writes `reviewer_handoff.md` in the repository being worked on. The reviewer writes `coder_handoff.md`, verifies and archives its reply, then removes the incoming handoff. The coder consumes the reply only when it exists and the incoming handoff is gone. Both files identify the task, review round, base commit and artifact commit.

Both skills discover the target repository's instructions, task tracking, check commands and contribution conventions. They do not require a particular language, directory layout, planning system or branch name. Commit and push destinations follow the owner's authorization and repository conventions.

These files are standalone skill sources. Editing this repository does not automatically update copies installed elsewhere.

## Dedicated versions

Choose one version per role for each client. Each folder is self-contained;
copy the whole skill folder from the matching harness directory, keeping its
name. Replace any older generic installation of the same skill to avoid
duplicate discovery.

| Client | Coder | Reviewer | Repository install directory | Personal install directory |
| --- | --- | --- | --- | --- |
| Claude Code | [Coder](claude-code/dape-auto-coder/SKILL.md) | [Reviewer](claude-code/dape-auto-reviewer/SKILL.md) | `.claude/skills/` | `~/.claude/skills/` |
| Codex | [Coder](codex/dape-auto-coder/SKILL.md) | [Reviewer](codex/dape-auto-reviewer/SKILL.md) | `.agents/skills/` | `~/.agents/skills/` |

In Claude Code, invoke `/dape-auto-coder` with the authorized task, or
`/dape-auto-reviewer` to start the review loop. In Codex, use
`$dape-auto-coder` or `$dape-auto-reviewer` in your prompt. Both remain eligible
for automatic selection when the request matches their descriptions.

Claude Code versions include guidance for its file tools, `AskUserQuestion`,
permission flow and `Monitor` when available. Codex versions include
`agents/openai.yaml` metadata and guidance for session tools, sandbox approvals,
mode-dependent questions and interruptible waits. Neither version pins a model
or assumes every client exposes the same tools.

Run the two roles in separate sessions against the same repository checkout.
Either client can fill either role, including a mixed Claude Code/Codex pair.
Use one coder and one reviewer per checkout; concurrent independent pairs need
separate checkouts to avoid sharing the two handoff filenames. Pausing a session
must preserve its handoff state; a stopped session is not an active watch.

When maintaining these copies, apply protocol changes to all versions; keep
platform tool differences in their execution sections. They are not generated
or automatically synchronized.

Platform packaging and invocation follow the official
[Claude Code skills documentation](https://code.claude.com/docs/en/skills) and
[Codex skills documentation](https://learn.chatgpt.com/docs/build-skills).
Claude monitoring guidance follows its
[tools reference](https://code.claude.com/docs/en/tools-reference).
