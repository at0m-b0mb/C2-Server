# LabC2 Roadmap

LabC2 is a long-running learning project. This roadmap breaks the work into
phases so progress is visible and the project is always in a runnable state.

## Phase 0 — Foundation (current)

- [x] Project scaffold + ethical positioning
- [x] License and ethics policy
- [x] Architecture document
- [ ] Wire-protocol specification v1
- [ ] JSON Schemas for every message type
- [ ] Lab setup guide (Vagrant + 2 VMs)
- [ ] Conformance test harness skeleton

## Phase 1 — MVP (2–4 weeks)

Goal: one working server + two working agents that interoperate.

- [ ] Python server (FastAPI + SQLite)
  - [ ] Operator auth (password + TOTP)
  - [ ] Agent registration + token issuance
  - [ ] Task queue endpoints
  - [ ] Result submission endpoints
  - [ ] Minimal HTMX dashboard
- [ ] Bash agent
  - [ ] Banner, allow-list, logging
  - [ ] All five core task types
- [ ] PowerShell agent
  - [ ] Banner, allow-list, logging
  - [ ] All five core task types
- [ ] Conformance tests: Python server × {Bash, PowerShell} agents

## Phase 2 — Second server, three more agents (3–5 weeks)

- [ ] Go server (net/http + GORM)
- [ ] Python agent
- [ ] Go agent (compiled, cross-platform)
- [ ] C# agent (compiled with `csc.exe`, no .NET SDK required on host)
- [ ] Conformance tests: 2 servers × 5 agents

## Phase 3 — Three more servers (4–6 weeks)

- [ ] Rust server (axum)
- [ ] TypeScript server (Fastify)
- [ ] C# server (ASP.NET Core minimal APIs)
- [ ] Conformance tests: 5 servers × 5 agents

## Phase 4 — Polish and showcase (2–3 weeks)

- [ ] Rust agent
- [ ] JavaScript (Node) agent
- [ ] Java server (Spring Boot)
- [ ] Cross-language benchmark table (req/sec, mem usage)
- [ ] Recorded demo video for portfolio
- [ ] Blog post(s) explaining design decisions
- [ ] Reference deployment guide (Docker Compose lab)

## Phase 5 — Optional research extensions

These are the "Research/CTF-grade" features. They go beyond the MVP and
make the project stand out, but they are explicitly *not* evasion features.

- [ ] WebSocket transport (alternative to long-poll HTTPS)
- [ ] gRPC transport (alternative to REST)
- [ ] Tor hidden-service deployment guide (for *learning* anonymous services)
- [ ] Multi-operator collaboration (live cursors, shared task queue)
- [ ] Plugin system for custom task types
- [ ] Differential conformance fuzzer (find protocol bugs across implementations)

## Definition of done for each phase

A phase is "done" when:

1. All checkboxes are ticked.
2. The conformance test matrix is green for everything in that phase.
3. There is at least one screencap or asciinema demo in `docs/demos/`.
4. The README "Implementation status" table is updated.
5. There is a git tag (`v0.1.0`, `v0.2.0`, etc.) on `main`.

## How to use this roadmap

- Pick the next unchecked box that depends only on already-checked boxes.
- Open an issue with the checkbox as the title.
- Cut a branch, implement, PR, merge, tick the box.
- Don't skip ahead unless you have a good reason; the phases are ordered
  so the conformance tests grow incrementally instead of all at once.
