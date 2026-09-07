# Skills

The paired skills for Dape's file-driven coding and review workflow:

- [dape-auto-coder](dape-auto-coder/SKILL.md): implements and verifies scoped tasks, publishes a handoff, and acts on the review.
- [dape-auto-reviewer](dape-auto-reviewer/SKILL.md): independently reviews the submitted artifact, publishes its verdict, and waits for the next round. Includes Codex UI metadata under `agents/`.

The coder writes `reviewer_handoff.md` in the repository being worked on. The reviewer writes `coder_handoff.md`, verifies and archives its reply, then removes the incoming handoff. The coder consumes the reply only when it exists and the incoming handoff is gone. Both files identify the task, review round, base commit and artifact commit.

Both skills discover the target repository's instructions, task tracking, check commands and contribution conventions. They do not require a particular language, directory layout, planning system or branch name. Commit and push destinations follow the owner's authorization and repository conventions.

These files are standalone skill sources. Editing this repository does not automatically update copies installed elsewhere.
