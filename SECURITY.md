# Security Policy

## Reporting a vulnerability in LabC2

If you discover a security vulnerability **in the LabC2 codebase itself**
(not in something LabC2 is being used to test), please do not open a public
GitHub issue.

Instead, email the maintainer at: `<add your email here>`

Include:
- A clear description of the vulnerability
- Steps to reproduce
- The version / commit hash of LabC2 affected
- Your suggested fix, if you have one

You will receive an acknowledgement within 7 days. Coordinated disclosure
of fixed issues follows a 90-day timeline by default.

## Reporting misuse of LabC2

If you become aware of LabC2 being used against systems without
authorization (i.e., to attack real targets), please contact the
maintainer at the same address. Provide:

- The artifacts you have (logs, code, screenshots)
- How you discovered it
- Whether you've reported it to the affected party or to law enforcement

## What is NOT a LabC2 vulnerability

The following are by design and are not bugs:

- Agents are easy for AV/EDR to detect. (Yes — that's intentional.)
- Agent network traffic is visible to a network monitor. (Intentional.)
- The protocol is human-readable. (Intentional.)
- Agents log everything. (Intentional and **not** configurable.)
- The agent's existence is visible in process lists. (Intentional.)

Issues filed asking for these to be "fixed" will be closed.

## Hardening for lab use

While LabC2 is not intended for hostile environments, the server itself
should be hardened:

- Always run behind TLS with a valid certificate (Let's Encrypt or
  internal CA in your lab).
- Use a strong password + TOTP for every operator account.
- Rotate agent tokens after each engagement.
- Set firewall rules so only your lab subnet can reach the C2 ports.
- Never expose the operator API to the public internet.
- Keep the host OS patched.
