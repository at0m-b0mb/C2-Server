# Contributing to LabC2

Thanks for your interest! LabC2 is a long-running learning project, so
contributions of new language implementations, conformance tests, and
documentation improvements are all welcome.

## Before contributing

1. Read [ETHICS.md](ETHICS.md) and [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md).
2. Skim [ARCHITECTURE.md](ARCHITECTURE.md) and [protocol/PROTOCOL.md](protocol/PROTOCOL.md).
3. Check [ROADMAP.md](ROADMAP.md) — your contribution should fit into a phase.

## What we accept

- ✅ New language implementations of the protocol (server or agent)
- ✅ Improvements to existing implementations (clarity, idiomatic code, tests)
- ✅ Documentation improvements
- ✅ Conformance test cases for the protocol
- ✅ Bug fixes
- ✅ Lab setup improvements (Vagrant, Docker)

## What we do NOT accept

- ❌ AV/EDR evasion features
- ❌ AMSI/ETW tampering
- ❌ Process injection, hollowing, sideloading
- ❌ Persistence mechanisms (registry, services, scheduled tasks, cron)
- ❌ Credential dumping / theft modules
- ❌ Lateral movement automation
- ❌ Anti-debugging / anti-VM tricks
- ❌ Code that disables the lab-mode guard rails

PRs adding any of the above will be closed. Don't take it personally —
these features belong in production red-team frameworks, not in a
learning project.

## How to add a new agent implementation

1. Open an issue titled `New agent: <language>` so we can avoid duplicate work.
2. Create `agent/<language>/` and add:
   - `README.md` with build/run instructions
   - Source files
   - A `<language>.conformance.json` describing which task types are supported
3. Implement all five core task types (see PROTOCOL.md §4).
4. Enforce all four lab-mode guard rails (banner, allow-list, logging, kill switch).
5. Run the conformance test suite against your agent + the Python reference server.
6. Open a PR linking the issue.

## How to add a new server implementation

1. Open an issue titled `New server: <language>`.
2. Create `server/<language>/` and add:
   - `README.md` with build/run instructions
   - Source files
   - A migration script for the schema
3. Implement every endpoint in PROTOCOL.md §3.
4. Reject `register` calls where `agent_kind != "labc2_educational"`.
5. Pass the conformance test suite against the Bash and PowerShell reference agents.
6. Open a PR.

## Code style

- Each implementation uses its language's idiomatic style and linter.
  Python → `ruff`. Go → `gofmt` + `go vet`. Rust → `cargo fmt` + `clippy`.
  TypeScript → `prettier` + `eslint`. C# → `dotnet format`. Etc.
- Heavy commenting is welcome — this is a learning project.
- Prefer clarity over cleverness.

## Commit / PR style

- Conventional commits: `feat(agent/bash): add download task`.
- One logical change per PR.
- PR description must reference the issue and explain the design choice
  if non-obvious.
- All PRs run the conformance test matrix in CI; PRs that break the matrix
  cannot merge.
