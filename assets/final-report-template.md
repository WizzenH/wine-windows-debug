# Wine compatibility resolution: <issue title>

## Disposition

- Status: `<SOLVED_CONFIRMED | WORKAROUND_CONFIRMED | CANDIDATE_AWAITING_VALIDATION | NOT_REPRODUCED | BLOCKED_WITH_EVIDENCE>`
- Confidence: `<high | medium | low>`
- User validation: `<date and result, or pending>`

## Scope and environment

- Application/build:
- Container image/service:
- Wine binary/version/architecture:
- WINEPREFIX:
- Wine source base revision:
- Graphics/audio/windowing/runtime details relevant to this issue:

## Original symptom and reproduction

- Reported symptom:
- Minimal reproduction:
- Expected behavior:
- Baseline result:
- Baseline evidence:

## Root cause

### Observations

-

### Inference

-

### Confirming experiment

-

## Regression provenance

- Classification: `<confirmed regression | suspected regression | pre-existing implementation gap | not attributable | not investigated because source regression was excluded>`
- Bad revision:
- Last known-good revision:
- Suspected or confirmed introducing commit:
- Affected component/path/symbol:
- History methods used: `<path log | -S | -G | blame | bisect | controlled revert/cherry-pick>`
- Good/bad build evidence:
- Relevant follow-up, revert, downstream, or vendor patches:

Do not name an introducing commit without controlled evidence. If only a suspect range is known, report the range and why it could not be narrowed.

## Solution

- Solution class: `<launch configuration | prefix/registry | container | Wine source | workaround>`
- Changed files:
- Runtime/container/registry changes:
- Why each change is necessary:
- Diagnostic changes removed before final verification:

## Wine source branch

- Repository:
- Base revision:
- Branch/worktree:
- Commit(s):
- Suspected or confirmed introducing commit:
- Diff summary:
- Build command and toolchain:

Write `Not applicable` when Wine source was not modified.

## Verification evidence

| Test | Base result | Candidate result | Evidence |
|---|---|---|---|
| Original reproduction | | | |
| Nearby regression/smoke test | | | |
| Clean/repeated run | | | |

## User validation

- Validation procedure supplied:
- User-observed result:
- Remaining behavioral caveats:

## Rollback

1.
2.

Include exact registry restoration, container/config reversal, and branch selection steps that apply. Do not rely on destructive reset commands.

## Residual risks and follow-up

-

## Run ledger summary

| State | Evidence or decision | Result |
|---|---|---|
| | | |

