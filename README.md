# wine-windows-debug

A Codex skill for autonomously diagnosing and fixing Windows applications running under Wine in containers.

It drives the issue from reproduction and adaptive log collection through evidence-based diagnosis, reversible environment or registry changes, container fixes, Wine source patches, compilation, regression testing, user validation, and a reviewable final report.

## Install

Clone or copy this repository to:

- Windows: `%USERPROFILE%\.codex\skills\wine-windows-debug`
- Linux/macOS: `~/.codex/skills/wine-windows-debug`

Start a new Codex task after installation.

## Use

Open the affected project in VS Code and invoke:

```text
$wine-windows-debug

Current problem: <describe the Wine behavior>
Please continue autonomously through reproduction, diagnosis, modification, build, and testing until I need to validate the real application behavior.
```

The normal user checkpoints are limited to confirming whether the original problem is solved and reviewing the final solution report. Host permission prompts or unavailable external access may still require intervention.

## Workflow

```text
Discovery
→ Baseline reproduction and logs
→ Ranked falsifiable hypotheses
→ Minimal reversible experiment
→ Environment / WINEPREFIX / registry / container fix
→ Wine source branch and build when required
→ Controlled A/B and regression tests
→ User runtime validation
→ Final report with rollback and residual risks
```

## Safety and repository discipline

- Prefer a disposable container and dedicated `WINEPREFIX`.
- Preserve dirty worktrees; never stash, reset, or overwrite user changes.
- Wine source patches use a separate `codex/wine-fix-*` branch or linked worktree.
- Build output and generated files stay out of commits.
- Never push automatically.
- Do not declare success before objective checks and user validation.

## Project configuration

The skill can discover project commands from `AGENTS.md`, README files, container manifests, launch scripts, and CI. For stable machine-readable configuration, adapt `assets/wine-project.example.yaml` to `.codex/wine-workflow.yaml` in the target project.
