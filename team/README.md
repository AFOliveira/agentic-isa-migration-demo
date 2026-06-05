# Agent Team Snapshot

This directory records the public-safe MULTIAGENT team configuration used for
the clean-release ISA migration run.

| Path | Contents |
| ---- | -------- |
| `isa-migration.toml` | The 16-worker team roster: Codex planner/reviewer and Cursor domain workers. |
| `roles/*.md` | Per-role prompt instructions loaded for each worker role. |
| `specs/*.md` | Job-spec templates used by the planner to dispatch the ISA migration pipeline. |

The role and spec markdowns are sanitized from the operator environment.
Operator-specific hostnames, user directories, and repository names are replaced
with placeholders such as `<implementation-root>`, `<remote-workload-host>`, and
`operator-board-app-repo`.

Runtime state, transcripts, build logs, caches, and machine-local paths are not
part of this public snapshot.
