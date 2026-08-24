# Optional project contract

Prefer repository-local instructions in `AGENTS.md`. Use `.codex/wine-workflow.yaml` when stable machine-readable paths and commands will prevent repeated discovery. A sample is available at [wine-project.example.yaml](../assets/wine-project.example.yaml).

## Discovery order

1. applicable `AGENTS.md` files;
2. `.codex/wine-workflow.yaml`;
3. README/build documentation and CI definitions;
4. Dockerfile/Containerfile, Compose files, devcontainer configuration, and launch scripts;
5. safe inspection of running containers and repositories.

Current user instructions override the project contract. Do not silently substitute a different Wine repository, prefix, executable, or container when a configured target is unavailable.

## Required semantics

The contract should establish, directly or by discoverable command:

- how to reproduce the reported issue;
- how to determine success;
- which container and prefix are disposable or protected;
- where Wine source lives and which base revision to use;
- how to configure, compile, install/stage, and test Wine;
- where reviewable reports belong;
- which local mutations and commits are allowed.

Values may be omitted when they are reliably discoverable. Never place passwords, API keys, private certificates, or raw customer data in the contract.

## Command handling

- Resolve relative paths against the workspace root unless a field states otherwise.
- Run commands in the declared environment; do not assume a host command is available inside the container or vice versa.
- Record expanded commands in the run ledger, while redacting secrets.
- Validate targets before recursive delete, image/container recreation, or worktree cleanup. Follow host approval requirements even when the contract permits the operation.

