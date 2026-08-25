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

## Component-aware diagnosis

The workflow maps symptoms to the layer that can actually own the failure before applying changes. The [component responsibility map](references/components.md) covers the application and launcher, environment, Wine prefix and registry, PE loader, Wine core, Win32 UI, X11/Wayland, graphics translation and host drivers, audio, input, networking, installers, fonts/locale, containers, and Wine source.

For each layer it records:

- the problems that layer can solve;
- the evidence that identifies it;
- the failures it usually cannot solve;
- the adjacent boundary needed for a controlled A/B test.

## Regression provenance

When a Wine source regression is plausible, the skill inspects relevant path and symbol history before writing a new patch. It uses path logs, `git log -S`, `git log -G`, `git blame`, release/tag comparison, controlled good/bad builds, diagnostic reverts or cherry-picks, and `git bisect` when the reproducer is deterministic.

A recent commit touching the same file is treated only as a lead. The final report classifies the result as a confirmed regression, suspected regression, pre-existing implementation gap, or not attributable, and records the introducing commit or bounded suspect range with test evidence.

## Safety and repository discipline

- Prefer a disposable container and dedicated `WINEPREFIX`.
- Preserve dirty worktrees; never stash, reset, or overwrite user changes.
- Wine source patches use a separate `codex/wine-fix-*` branch or linked worktree.
- Build output and generated files stay out of commits.
- Never push automatically.
- Do not declare success before objective checks and user validation.

## Project configuration

The skill can discover project commands from `AGENTS.md`, README files, container manifests, launch scripts, and CI. For stable machine-readable configuration, adapt `assets/wine-project.example.yaml` to `.codex/wine-workflow.yaml` in the target project.
