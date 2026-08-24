---
name: wine-windows-debug
description: Diagnose and fix Windows applications running under Wine in containers, from evidence collection through environment, registry, container, or Wine source changes, compilation, regression testing, and a reviewable solution report. Use when a user reports a Wine runtime, compatibility, rendering, dependency, installer, crash, or behavior problem. Do not use for native Windows debugging that does not involve Wine.
---

# Wine Windows Debug

Drive the issue to an evidence-backed candidate fix with minimal user interruption. The normal user checkpoints are only:

1. the user verifies whether the original observable problem is solved;
2. the user reviews whether the final solution report meets project rules.

Host permission prompts, missing credentials, unavailable hardware, or an undiscoverable mutation target may still require a precise blocking request. Do not ask for information that can be discovered safely from the workspace, container, repository, process state, or logs.

## Operating contract

- Treat the opened workspace and its configured container/Wine resources as the authorized scope. Do not modify unrelated host state.
- Prefer a disposable container and dedicated `WINEPREFIX`. Never alter the user's primary prefix or host-wide registry unless the project configuration explicitly selects it.
- Run an unknown Windows executable only inside the configured isolation boundary. Keep network and credentials unavailable unless the application demonstrably requires them and the project permits them.
- Preserve user changes. Do not stash, discard, reset, or overwrite a dirty worktree.
- Never push branches, publish artifacts, or send reports externally.
- Do not claim the issue is fixed until objective checks pass and the user confirms the original symptom is gone.

Read [project-config.md](references/project-config.md) when locating or establishing project-specific commands and paths. If `.codex/wine-workflow.yaml` exists, treat it as the project contract. If it does not exist, inspect `AGENTS.md`, README files, container manifests, launch scripts, CI, and repository structure; infer safe values and proceed. Create a project config only when persistence will materially improve later runs.

## Required workflow

Follow the detailed state machine in [workflow.md](references/workflow.md). Maintain a short run ledger containing the current state, evidence, hypotheses, experiment, result, and next decision. Continue autonomously while a safe evidence-producing action remains.

The high-level sequence is:

1. **Discover and baseline** — identify executable, container, Wine binary/version/architecture, prefix, source revision, graphics stack, launch command, and success signal. Capture repository dirtiness before mutation.
2. **Reproduce** — run the smallest faithful reproduction and save the command, exit status, relevant logs, and observable failure. A failed reproduction is a finding, not permission to guess.
3. **Diagnose** — rank falsifiable hypotheses and choose the cheapest discriminating experiment. Use the adaptive log guidance in [diagnostics.md](references/diagnostics.md).
4. **Fix by escalation** — prefer, in order: launch/environment settings; prefix/DLL/registry settings; container dependencies/configuration; Wine source patch. Skip a lower layer only when evidence excludes it.
5. **Verify** — rerun the original reproduction under controlled A/B conditions, run targeted regression tests, and check that the apparent fix did not come from an unrelated prefix, cache, image, or dependency change.
6. **User validation** — provide exact validation steps and the expected result. Pause for the user's judgment of the real application behavior.
7. **Document** — after validation, produce a report based on [final-report-template.md](assets/final-report-template.md), including rollback and residual risks. Pause for final compliance review.

## Mutation discipline

- Make one primary causal change per experiment. Record before/after values and a rollback command or file.
- Prefer reproducible files such as a Dockerfile, Containerfile, Compose override, launch script, or checked-in registry fragment over an unexplained live-container edit.
- Before registry changes, export the affected key or relevant prefix registry files. Record the exact `WINEPREFIX`, key, value, type, and rollback procedure.
- Treat DLL overrides, winetricks verbs, symlinks, environment variables, package installations, graphics/runtime changes, and cache invalidation as real mutations and record them.
- Do not conceal a workaround as a root-cause fix. Label workaround, compatibility configuration, container fix, and Wine source fix distinctly.

## Wine source changes

Only patch Wine source when evidence shows that configuration or dependency changes are insufficient or the project explicitly requires a source-level fix.

1. Inspect the source repository, base revision, remotes, build instructions, and current status.
2. Create a separate local branch named `codex/wine-fix-<issue-slug>`. If the current worktree is dirty or in active use, create a linked worktree for the branch instead of disturbing it.
3. Reproduce against the unmodified base using the same build and runtime conditions.
4. Implement the smallest source change that tests the leading hypothesis. Keep generated files and build output out of commits.
5. Compile in an isolated build directory, capture the exact command and compiler/toolchain versions, and test the built Wine rather than a system Wine by mistake.
6. Run the original reproducer plus relevant Wine tests or project smoke tests. Compare with the base result.
7. After verification, create at most one local commit per logical fix when repository policy permits. Do not push. Report branch, worktree, commit, diff summary, and remaining uncommitted files.

## Completion and stopping rules

- A passing process exit alone is insufficient when the reported problem is visual, interactive, performance-related, or functional.
- If an experiment fails, update the hypotheses from the new evidence. Never repeat the same unchanged experiment.
- After three failed interventions in one mutation layer, return to diagnosis before escalating; do not accumulate random tweaks.
- Stop as blocked only when safe investigation paths are exhausted or the next action requires unavailable user-only access. State the exact missing capability and preserve the run ledger.
- The final handoff must identify: disposition, confidence, user validation status, changed files/state, Wine branch/commit if any, test evidence, rollback, report path, and unresolved risks.
