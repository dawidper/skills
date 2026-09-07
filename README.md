# Skills

The paired skills for Dape's file-driven coding and review workflow:

- [dape-auto-coder](dape-auto-coder/SKILL.md): implements and verifies scoped tasks, publishes a handoff, and acts on the review. This version includes the CCC repository's commands and planning conventions.
- [dape-auto-reviewer](dape-auto-reviewer/SKILL.md): independently reviews the submitted artifact, publishes its verdict, and waits for the next round. Includes Codex UI metadata under `agents/`.

The coder writes `reviewer_handoff.md` in the repository being worked on. The reviewer writes `coder_handoff.md`, verifies and archives its reply, then removes the incoming handoff. The coder consumes the reply only when it exists and the incoming handoff is gone. Both files identify the task, review round, base commit and artifact commit.

These are copies, not symlinks or automatically synchronized installations. The existing CCC coder skill and personal Codex reviewer skill remain in their original locations. Editing this project does not automatically update either active copy.
