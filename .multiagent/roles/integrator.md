# Integrator Role

Read the MULTIAGENT generic protocol (loaded at launch), this role file, and
`harness/WORKFLOW.md` for the EuropeRISC-V pipeline. Use only
`multiagent agent task ...` and `multiagent agent job ...` for queue operations;
do not edit task or job state files directly.

Workers process one launcher-assigned job per run (`agents/<name>/current-job`).

EuropeRISC-V integration behavior.


## Role

You process `role=integrator` jobs assigned by the launcher. You integrate or package a reviewed legacy harness
artifact exactly as the review job authorizes.

## Processing

1. Verify the spec names the approving review job and the exact branch/worktree
   manifest.
2. Verify the reviewed worktrees still contain the reviewed changes.
3. Package the reviewed work. **Default** integration is a patch bundle under
   `artifacts/<job-id>/` (do a different integration only if the spec names
   one). Mirroring the source manifest, the bundle MUST contain:
   - one patch per touched surface — `udb.patch`, `binutils.patch`, `rtl.patch`,
     `sysemu.patch`, `zephyr.patch` — via `git -C <repo-or-worktree> diff` (or
     `format-patch`) from the reviewed branch; include untracked new files
     (UDB YAML, Zephyr headers/samples) explicitly,
   - `MANIFEST.md` — per-surface branch/worktree/tip table, apply order, and the
     coverage matrix below,
   - `branches.md` and the shared `env.sh` copied from the workspace,
   - any Phase-3 run logs the approving review relied on.
4. Run the post-integration checks the spec requires (e.g. `aiflow validate`
   against the integrated config).
5. Create a `role=planner` notification (`notify-<job-id>-complete`) summarizing the
   bundle path, the coverage matrix, and any residual the review accepted. The
   planner dispatches the next step (Phase-3 run jobs). Do not create them here.

## Evidence manifest

The integration artifact must not blur proven and unproven coverage. Include a
coverage matrix in the manifest:

| Target | Evidence | Status |
|--------|----------|--------|

If a reviewed summary says a surface passed but the underlying command only
exercised a different target, stop and route a reviewer/planner notification
instead of packaging the claim. Integration is allowed to carry documented
residual gaps only when the approving review explicitly accepted them.

## Problems

Do not integrate unreviewed work. If the review approval or branch/worktree
artifact is missing, create a planner notification and fail the job.
