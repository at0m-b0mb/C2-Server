# LabC2 Architecture

This document describes the high-level design of LabC2. For the exact
on-the-wire contract that all implementations must satisfy, see
[protocol/PROTOCOL.md](protocol/PROTOCOL.md).

## Core design goals

1. **Language-agnostic protocol.** The server and agent talk over a plain
   JSON-over-HTTPS protocol so any language with an HTTP client can
   implement an agent and any language with a web framework can implement
   a server.
2. **Readable over clever.** Implementations should prioritize clarity for
   learners over micro-optimization or stealth.
3. **Strong defaults.** TLS is mandatory; tokens are mandatory; logging
   cannot be disabled; agents declare themselves on first check-in.
4. **Lab-safe.** Hardcoded server allow-lists, banner output, and a
   universally-honored `terminate` command.

## Components

### 1. The C2 Server

Responsibilities:

- Maintain a database of registered agents and their state.
- Expose a REST API for operators (web dashboard / CLI).
- Expose a REST API for agents (registration, task pickup, result post).
- Authenticate operators with username+password+TOTP (or API key).
- Authenticate agents with pre-shared tokens.
- Issue tasks to agents and store their results.
- Serve a web dashboard (static SPA or server-rendered HTML).

Stack-agnostic, but each implementation should use the most idiomatic
choices for its language:

- Python: FastAPI + SQLAlchemy + SQLite/PostgreSQL
- Go: net/http + GORM + SQLite/PostgreSQL
- Rust: axum + sqlx + SQLite/PostgreSQL
- TypeScript: Fastify + Prisma + SQLite/PostgreSQL
- C#: ASP.NET Core minimal APIs + EF Core
- Java: Spring Boot + JPA

### 2. The Agent (a.k.a. implant, beacon)

Responsibilities:

- On startup, print the lab-mode banner to stdout/stderr.
- Verify the configured server URL is on the compiled-in allow-list.
- Register with the server (POST /agents/register).
- Loop: sleep → fetch tasks → execute → post results → repeat.
- Honor the `terminate` task by exiting cleanly.
- Append every executed command + result to a local log file.

All agents implement the same five core task types (see PROTOCOL.md):

| Task         | Description                              |
| ------------ | ---------------------------------------- |
| `shell`      | Execute a shell command, return output   |
| `download`   | Read a local file, return its content    |
| `upload`     | Write content to a local file            |
| `sysinfo`    | Return OS, hostname, user, network info  |
| `terminate`  | Exit the agent cleanly                   |

Additional task types are optional but must be declared in PROTOCOL.md.

### 3. The Operator UI

A web dashboard served by the C2 server. Features:

- Login (username + password + TOTP).
- Agent list with last-seen timestamps and status.
- Per-agent task queue and result history.
- "New task" form per agent.
- Audit log of operator actions.

Each server implementation can ship its own UI, but a reference SPA
(plain HTML + HTMX + small CSS) lives in [server/_shared_ui](server/_shared_ui)
and can be served by any backend.

## Security model

### What we authenticate

| Channel             | Mechanism                                         |
| ------------------- | ------------------------------------------------- |
| Operator → Server   | Username + password + TOTP, then session cookie   |
| Agent → Server      | Pre-shared agent token (in `Authorization` header)|
| Server ⇄ Agent      | TLS 1.2+ with certificate pinning (optional)      |

### What we do **not** try to hide

- Agents make no effort to obscure their network traffic. Standard HTTPS
  on a standard port to an advertised hostname.
- Agents do not obfuscate strings, encrypt payloads in memory, or hide from
  process listings.
- Agents do not auto-elevate or persist across reboots.

This is deliberate. LabC2 is a learning tool; if you want to learn evasion,
read papers and audit production frameworks in a research lab — don't
import those techniques here.

## Lab-mode guard rails

Every agent enforces these on startup:

1. **Banner.** Prints to stderr:
   ```
   ╔══════════════════════════════════════════════════════╗
   ║  LabC2 educational agent — authorized lab use only   ║
   ║  Logging to: <path>                                  ║
   ║  Server: <url>                                       ║
   ╚══════════════════════════════════════════════════════╝
   ```
2. **Allow-list check.** Refuses to start if the configured server URL
   does not match a compiled-in regex pattern (default: only RFC1918 and
   `localhost`).
3. **Mandatory log file.** Refuses to start if the log file can't be opened
   for append.
4. **Self-declaration.** First registration payload includes
   `"agent_kind": "labc2_educational"` and `"lab_mode": true`.

The allow-list patterns can be edited at build time, but the agent will
always print the patterns it was compiled with on startup so the host
operator knows which targets it can reach.

## Directory layout

```
LabC2/
├── README.md
├── LICENSE
├── ETHICS.md
├── ARCHITECTURE.md            ← this file
├── ROADMAP.md
├── CODE_OF_CONDUCT.md
├── CONTRIBUTING.md
├── SECURITY.md
├── protocol/
│   ├── PROTOCOL.md            ← the wire-protocol spec
│   └── schemas/               ← JSON Schema definitions
├── server/
│   ├── python/                ← FastAPI implementation
│   ├── go/                    ← Go net/http implementation
│   ├── rust/                  ← axum implementation
│   ├── typescript/            ← Fastify implementation
│   ├── csharp/                ← ASP.NET Core implementation
│   ├── java/                  ← Spring Boot implementation
│   └── _shared_ui/            ← reference web dashboard
├── agent/
│   ├── powershell/
│   ├── bash/
│   ├── python/
│   ├── go/
│   ├── csharp/
│   ├── rust/
│   └── javascript/
├── lab/                       ← Vagrant/Docker lab setup
│   └── README.md
└── tests/
    └── protocol_conformance/  ← cross-language conformance tests
```

## Interoperability contract

Because every server and every agent speaks the same protocol, any
combination must work:

- Python server + Bash agent ✅
- Go server + PowerShell agent ✅
- Rust server + C# agent ✅
- etc.

The [tests/protocol_conformance](tests/protocol_conformance) suite runs
every server against every agent in a matrix to enforce this.
