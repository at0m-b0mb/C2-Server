# LabC2 — A Multi-Language Educational C2 Framework

> ⚠️ **EDUCATIONAL AND AUTHORIZED-USE ONLY.** This project is designed for
> cybersecurity learning, red-team training, CTF practice, and authorized
> penetration testing inside lab environments you own or have written
> permission to test. Use against systems you do not own or are not
> authorized to test is **illegal** in most jurisdictions and a violation
> of this project's license. See [ETHICS.md](ETHICS.md).

LabC2 is an open, educational Command-and-Control (C2) framework built as a
**multi-language reference implementation**. The same lightweight wire
protocol is implemented in many popular languages on both the server side
and the agent (client) side, so it doubles as a study in cross-language
network programming, authentication, and protocol design.

## Why this project exists

Modern blue and red teamers are expected to understand how C2 frameworks
work end-to-end. Most production frameworks (Sliver, Mythic, Havoc, Empire)
are large and battle-hardened — great to use, but hard to *learn from*
because every line of code is doing something subtle.

LabC2 takes the opposite approach: small, readable, heavily commented
implementations of the same protocol in many languages, with safety rails
baked in so it cannot easily be repurposed for harm.

## What's intentionally NOT in scope

This project will **not** ship:

- AV / EDR evasion techniques
- AMSI bypasses, ETW patching, or unhooking
- Process injection / hollowing / DLL sideloading
- Domain fronting, malleable profiles tuned for evasion
- Persistence rootkits or bootkits
- Credential theft modules
- Lateral movement automation

It will ship: a clean protocol, multiple reference implementations,
strong auth, TLS, lab-mode guard rails, and clear documentation.

## Architecture at a glance

```
┌──────────────────┐      TLS + token      ┌──────────────────┐
│  Operator (you)  │ ────────────────────▶ │   C2 Server      │
│   web dashboard  │ ◀──────────────────── │  (any language)  │
└──────────────────┘                       └────────┬─────────┘
                                                    │
                                          HTTPS + signed
                                          JSON tasks/results
                                                    │
                                     ┌──────────────┴──────────────┐
                                     ▼                             ▼
                            ┌────────────────┐            ┌────────────────┐
                            │  Agent (any    │            │  Agent (any    │
                            │   language)    │   ...      │   language)    │
                            └────────────────┘            └────────────────┘
```

See [ARCHITECTURE.md](ARCHITECTURE.md) for detail and
[protocol/PROTOCOL.md](protocol/PROTOCOL.md) for the wire-protocol spec
that every implementation must satisfy.

## Implementation status

### Server implementations

| Language       | Status      | Path                                       |
| -------------- | ----------- | ------------------------------------------ |
| Python (FastAPI) | 🚧 planned | [server/python](server/python)            |
| Go             | 🚧 planned  | [server/go](server/go)                    |
| Rust           | 🚧 planned  | [server/rust](server/rust)                |
| TypeScript     | 🚧 planned  | [server/typescript](server/typescript)    |
| C# (.NET)      | 🚧 planned  | [server/csharp](server/csharp)            |
| Java           | 🚧 planned  | [server/java](server/java)                |

### Agent implementations

| Language    | OS Target      | Pre-installed? | Status      | Path                            |
| ----------- | -------------- | -------------- | ----------- | ------------------------------- |
| PowerShell  | Windows        | ✅ yes         | 🚧 planned  | [agent/powershell](agent/powershell) |
| Bash        | Linux/macOS    | ✅ yes         | 🚧 planned  | [agent/bash](agent/bash)        |
| Python      | Cross-platform | mostly         | 🚧 planned  | [agent/python](agent/python)    |
| Go          | Cross-platform | compiled       | 🚧 planned  | [agent/go](agent/go)            |
| C# (.NET)   | Windows        | ✅ yes         | 🚧 planned  | [agent/csharp](agent/csharp)    |
| Rust        | Cross-platform | compiled       | 🚧 planned  | [agent/rust](agent/rust)        |
| JavaScript  | Cross-platform | sometimes      | 🚧 planned  | [agent/javascript](agent/javascript) |

## Quick start (lab use only)

> Run the server and agents **only** inside an isolated VM lab. Recommended:
> two VMs on a host-only network. See [lab/README.md](lab/README.md).

```bash
# 1. Clone
git clone https://github.com/<you>/LabC2.git
cd LabC2

# 2. Start the Python reference server (once implemented)
cd server/python
pip install -r requirements.txt
python -m labc2_server --lab-mode

# 3. On the lab target VM, run an agent
cd agent/bash
LABC2_SERVER=https://10.0.0.1:8443 LABC2_TOKEN=<token> ./agent.sh
```

## Safety features built into every agent

Every agent implementation **must** enforce these (see [ARCHITECTURE.md](ARCHITECTURE.md)):

1. **Lab-mode banner.** Prints `LabC2 educational agent — authorized lab use only` on startup.
2. **Server allow-list.** Compiled-in list of acceptable C2 server hostnames; will not connect elsewhere.
3. **Verbose logging.** Cannot be disabled. All commands and results are logged locally to a file.
4. **Kill switch.** Server can send a `terminate` command that the agent must honor.
5. **Authorization beacon.** First check-in declares "I am a LabC2 educational agent" in plain JSON.
6. **No silent execution.** Agents never daemonize or hide; they run in the foreground.

## Documentation

- [ETHICS.md](ETHICS.md) — Ethical-use policy (binding under the project license)
- [ARCHITECTURE.md](ARCHITECTURE.md) — High-level design
- [protocol/PROTOCOL.md](protocol/PROTOCOL.md) — Wire-protocol specification
- [ROADMAP.md](ROADMAP.md) — Phased delivery plan
- [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md) — Community standards
- [CONTRIBUTING.md](CONTRIBUTING.md) — How to contribute new language implementations
- [SECURITY.md](SECURITY.md) — Vulnerability reporting
- [lab/README.md](lab/README.md) — How to set up a safe testing lab

## License

LabC2 is released under a custom Educational and Authorized-Use License
(see [LICENSE](LICENSE)) — based on AGPL-3.0 with additional restrictions
prohibiting unauthorized use. By using this code you accept those terms.

## Author

Built as a learning project and cybersecurity portfolio. Contributions
welcome, especially additional language implementations of the protocol
spec.
