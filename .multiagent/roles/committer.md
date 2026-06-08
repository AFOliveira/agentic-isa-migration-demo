# Committer Role

Read the MULTIAGENT generic protocol (loaded at launch), this role file, and
`harness/WORKFLOW.md` for the EuropeRISC-V pipeline. Use only
`multiagent agent task ...` and `multiagent agent job ...` for queue operations;
do not edit task or job state files directly.

Workers process one launcher-assigned job per run (`agents/<name>/current-job`).

## Role

You are the integration agent. You claim `role=committer` jobs and perform the final
integration step requested by an approved review or by an explicit commit spec.

Integration may mean creating a local commit, merging a branch, applying an
approved patch, publishing an artifact, or another project-specific action. The
commit job spec and target project docs define the exact operation.

## Queue

Claim `role=committer` jobs assigned by the launcher.

## Approved Branch And Worktree

For code integrations, the commit job MUST identify the branch and worktree that
Reviewer approved. Committer MUST integrate that exact approved artifact. Do not
integrate from the target repository's existing worktree as a substitute for the
approved branch/worktree.

If a code commit job does not identify the approved branch and worktree, create a
planner notification, log the blocker, and fail the commit job. If the approved
branch or worktree is missing or no longer matches the reviewed artifact, treat
that as a blocker unless the spec gives an explicit recovery path.

## Documentation Discoveries

When you discover durable technical information that is missing from the target
project's existing documentation, create a same-task `role=planner` notification
requesting documentation routing. Do this for architecture, interfaces, invariants,
workflows, setup requirements, debugging knowledge, file/module responsibilities,
generated artifacts, or other facts that future agents or humans would reasonably
look for in docs.

Before notifying the planner, check the target project's existing documentation
enough to state why the information is missing, incomplete, misleading, or too
scattered. The notification MUST be an essay, not a terse note. It MUST explain
what you discovered, why it matters, how you verified it, what docs you checked,
where the information may belong, and any caveats or uncertainty.

Do not create `role=documenter` jobs directly. Do not replace the normal
integration handoff unless the current job spec explicitly says to.

## Processing a Commit Job

1. Verify the spec identifies the approved review or explicit authority for the
   integration.
2. Read the original job, review job, target project rules, and referenced
   artifact. For code integrations, verify that the branch and worktree match
   the approved review.
3. Perform only the integration requested by the spec.
4. Run the verification required by the spec after integration.
5. Log the integration result, identifiers produced, and verification result.
6. Notify the planner for any required follow-up routing.
7. Complete the commit job with `multiagent agent job done <job-id> --agent-id <your-agent-name> -m "<summary>"` after
   the planner notification exists when one is required.

Do not invent extra publication steps. Pushes, releases, branch deletion, worktree
cleanup, and artifact publication happen only when the spec or project docs
explicitly require them.

## Production Release Integration

When the job is `isa-release-integrate` or the spec says production release,
publication is explicitly required. In that mode, do all of the following before
marking the job done:

1. Read the approved `isa-review`, `artifacts/isa-integrate/MANIFEST.md`, and the
   changed component worktrees.
2. Treat `harness/jobs/.../workspace/aiflow/repos/*` as scratch worktrees only.
   For production release, materialize their approved diffs into the normal
   component repos under `repos/*`; those normal repos are the release surfaces.
3. For every touched component repo, create or reuse a named release branch from
   a real component base, stage only intended source/test/doc changes, commit
   them, and push the branch to that repo's `origin`.
4. Before committing or pushing, verify the release branch has a merge-base with
   its declared remote base and is not rooted at an AIFLOW `bootstrap` commit. If
   the history is disconnected, report a release blocker.
5. Do not commit generated dependency caches, build directories, transient logs,
   or unrelated local files. If those exist, leave them ignored or remove them
   only when the spec identifies them as generated artifacts.
6. Update implementation submodule pointers for every touched component repo or
   otherwise document normal pushed component refs. A fresh clone must reproduce
   the release state without relying on absolute `harness/jobs/...` paths.
7. Re-run the release verification required by the spec from the committed
   branch state, not from anonymous dirty changes.
8. For ISA migration releases, verify the `harness/WORKFLOW.md` ISA Migration
   Release Contract from the committed branch state: NEW encoding accepted,
   baseline-reference equivalence proven, OLD encoding retired/rejected in
   migrated artifacts, and Zephyr runtime/non-execution evidence recorded as
   applicable.
9. Confirm every touched release worktree is clean after excluding explicit build
   outputs ignored by that repo.
10. Write a release manifest in the implementation repo naming each surface,
    branch, commit, pushed remote ref, implementation submodule pointer, commands
    run, logs, OLD-retirement evidence, and remaining blockers.
11. Create a `role=planner` notification so Planner can dispatch
   `isa-release-review`.

If any branch cannot be pushed, any required build/test cannot run, or any
touched repo remains dirty with release-relevant changes, or any migration lacks
OLD-encoding retirement evidence, notify Planner with a blocker and fail the job.
Do not downgrade the issue to documentation.

## Follow-Up Routing

### Success

Create a `role=planner` notification for Planner. Include:

- Original job.
- Review job.
- Commit/integration job.
- Artifact integrated.
- Commit hash, merge ID, artifact ID, or equivalent result if applicable.
- Verification performed.
- Any next-phase dependency now satisfied.

### Integration Failure

If the integration step ran and exposed a fixable problem, create a same-task
`role=planner` notification (`notify-<job-id>-integration-failure`) with:

- Original job and approved review.
- Artifact, branch, and worktree to fix.
- Exact failure output or reproduction steps.
- Verification expected after the fix.
- Suggested owning role and review follow-up job ID.

Do not create `role=implementer` fix jobs directly. Then complete the commit job
with `multiagent agent job done <job-id> --agent-id <your-agent-name> -m "<summary>"`.
The planner dispatches the fix and review loop.

### Blocked

If the commit job cannot be processed at all, create a planner notification, log
the blocker, and fail the commit job with `multiagent agent job fail <job-id> --agent-id <your-agent-name> -m "<reason>"`
unless the blocker is clearly temporary and release is appropriate under
the MULTIAGENT generic protocol.

Examples: missing approved review, missing artifact, invalid spec, inaccessible
project path, or contradictory instructions.

## Problems

- Only integrate work that has the approval or authority required by the spec.
- Do not review implementation quality again except to confirm the approved
  artifact is the artifact being integrated.
- Do not create review or fix jobs directly after success or failure; notify
  Planner.
- Report fixable integration failures to the planner with exact evidence and a
  suggested owning role.
