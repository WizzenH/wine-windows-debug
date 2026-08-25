# Wine diagnosis state machine

Use these states explicitly in commentary or the local run ledger. A run may move backward when evidence disproves a hypothesis.

## 1. DISCOVERY

Collect facts without mutation:

- application executable, arguments, working directory, input data, and required services;
- container runtime, image digest, Compose service, mounts, devices, environment, and network mode;
- `wine --version`, binary path, architecture, `WINEPREFIX`, DLL overrides, installed verbs/packages, and registry files;
- host and container kernel, CPU architecture, graphics devices, driver versions, Vulkan/OpenGL capabilities, display server, and audio stack when relevant;
- Wine source repository path, revision, patch state, build directory, and documented build/test commands;
- upstream or vendor origin, branch/tag relationship, downstream patch stack, and the last known-good revision when available;
- objective success signal and the user's original symptom.

Record `git status --short`, the container/image identity, and prefix identity before changing anything.

Exit when the reproduction boundary and mutation targets are unambiguous.

## 2. BASELINE_REPRODUCTION

Run the normal launch path first, then the smallest faithful reproducer. Capture:

- exact command and environment differences from normal operation;
- stdout/stderr, exit code or signal, timestamps, process tree, and relevant container logs;
- crash dumps/backtraces, missing module messages, graphics/API errors, registry access failures, or installer logs;
- a screenshot or user-observable description when the failure is visual;
- whether the result is deterministic and how many attempts were made.

Do not enable every `WINEDEBUG` channel at once. Excessive logging changes timing, creates noise, and can hide the causal sequence.

If the issue cannot be reproduced, compare the user's runtime with the test runtime before asking the user. Check mounts, display, devices, prefix, environment, image digest, input data, permissions, and launch wrapper.

## 3. HYPOTHESIS

Maintain a ranked table:

| Rank | Hypothesis | Evidence for | Evidence against | Discriminating test |
|---|---|---|---|---|

Each hypothesis must predict an observable result. Prefer tests that distinguish multiple hypotheses without mutation.

Common classes include:

- wrong prefix, bitness, executable, working directory, or launch environment;
- missing native/Unix dependency or incorrect DLL load order;
- registry/feature detection mismatch;
- graphics translation, driver, synchronization, windowing, font, locale, audio, or input issue;
- installer/runtime prerequisite problem;
- application bug exposed by Wine;
- Wine implementation gap or regression;
- container isolation, mount, device, seccomp, permission, or environment problem.

Read [components.md](components.md) when component ownership is unclear. Select the narrowest component whose behavior can explain all primary observations, but keep adjacent boundaries in the hypothesis table when a translation layer may only be surfacing a lower-layer failure.

### Regression provenance check

Run this check when the symptom appeared after a Wine/application/container update, differs across revisions, points to a recently changed Wine subsystem, or would otherwise require a Wine source patch.

1. Record the exact bad revision, last known-good revision or release, branch topology, and downstream/vendor patches. Do not compare version labels without resolving their commits.
2. Identify the narrowest relevant path, symbol, registry behavior, protocol, or test. Inspect first-parent and full history as appropriate, including renames.
3. Use path history plus pickaxe searches (`git log -S` for changed strings/symbol occurrences and `git log -G` for matching diff lines). Use `git blame` only to locate candidate history; blame is not proof of causation.
4. Read candidate commits in full: parent diff, commit message, tests, follow-up fixes, reverts, and affected architecture. Check whether the project carries or omits the patch downstream.
5. Confirm causality with identical controlled builds of a known-good and bad revision. If practical, test a diagnostic revert or cherry-pick in an isolated worktree.
6. When the range is still broad and the reproducer has a deterministic pass/fail exit signal, run `git bisect` in a disposable linked worktree. Record good/bad anchors, skipped unbuildable commits, test command, and first-bad result. Never run a noisy or subjective visual test unattended as a bisect oracle.
7. Classify the result as `confirmed regression`, `suspected regression`, `pre-existing implementation gap`, or `not attributable`. A temporal correlation or touching the same file is only suspected evidence.

Do not make reverting an upstream patch the permanent solution merely because it restores behavior. Use a revert as an experiment, then determine whether the durable result should be a forward fix, backport, configuration guard, or documented version pin.

## 4. EXPERIMENT

Choose one primary change and write before execution:

- hypothesis being tested;
- exact mutation;
- expected result if true;
- expected result if false;
- rollback;
- test command and success signal.

Apply the smallest reversible experiment. Save its evidence and classify the outcome as supported, contradicted, or inconclusive.

## 5. FIX_CANDIDATE

When an experiment supports a hypothesis, convert the live change into a reproducible fix:

- launch behavior belongs in a wrapper or explicit environment configuration;
- registry state belongs in a documented `.reg` fragment or initialization step;
- container dependencies belong in image or Compose configuration;
- Wine behavior belongs in a separate source branch with a minimal patch.

Remove unrelated diagnostic changes before testing the candidate.

## 6. VERIFICATION

Use the same inputs and runtime boundary for base and candidate. Verify:

1. base still reproduces the failure;
2. candidate passes the original success signal;
3. candidate passes at least one nearby negative or regression case;
4. repeated runs do not depend on warmed caches or a contaminated prefix;
5. logs no longer contain the causal failure, rather than merely hiding it;
6. branch builds invoke the branch-built Wine binaries and libraries.

Record duration, exit status, artifact hashes or revisions, and concise log evidence.

For a regression claim, verification must include the tested good/bad revisions and the introducing commit or bounded suspect range. Confirm that the final candidate works on the intended base rather than only on the historical good revision.

## 7. AWAITING_USER_VALIDATION

Give the user one short validation procedure containing:

- exact build/container/prefix to use;
- exact launch command or action;
- what behavior should now occur;
- any known limitation to check.

Do not ask the user to diagnose logs. Ask only whether the original problem is solved and capture their result verbatim or as a faithful summary.

If validation fails, return to BASELINE_REPRODUCTION using the user's new observation as evidence.

## 8. REPORTING

Write the final report only after the user's runtime validation, unless blocked. Use the supplied template and keep factual evidence separate from inference. Include a complete rollback and distinguish permanent changes from diagnostic residue.

End in one disposition:

- `SOLVED_CONFIRMED`
- `WORKAROUND_CONFIRMED`
- `CANDIDATE_AWAITING_VALIDATION`
- `NOT_REPRODUCED`
- `BLOCKED_WITH_EVIDENCE`

