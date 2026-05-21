# Ethical Use Policy

LabC2 is a dual-use security project. The same skills and code that help
defenders understand attacker tradecraft can, in the wrong hands, harm
real people and real systems. This document is the moral contract that
goes with the legal license in [LICENSE](LICENSE).

## The rule

**Use LabC2 only on systems you own or have written, prior authorization
to test.** That's it. Everything else is a corollary.

## What "authorization" means

Authorization is:

- ✅ Your own VM lab on your own hardware
- ✅ A Capture-the-Flag environment that explicitly permits offensive tooling
- ✅ A bug-bounty program target, used within the scope of the program
- ✅ A paid penetration-test engagement with a signed Statement of Work
- ✅ A university lab assignment using infrastructure provided by the course
- ✅ HackTheBox, TryHackMe, PortSwigger Web Security Academy, or similar legal training platforms

Authorization is **not**:

- ❌ "I'm just testing" on a friend's computer
- ❌ "I'll undo it after" on a stranger's server
- ❌ "It's exposed on the internet so it's fair game"
- ❌ "They probably won't notice"
- ❌ "It's my employer's system" — without written approval from your employer's security team
- ❌ Any system whose owner has not specifically said yes

If you can't produce documentation that proves authorization, you do not
have authorization.

## What to do if you find a vulnerability while using LabC2

If during authorized testing you discover a real, exploitable vulnerability
in third-party software:

1. **Stop the active test.** Do not pivot, exfiltrate, or weaponize.
2. **Document.** Note exactly what you did and when.
3. **Report responsibly.** Use the vendor's security contact or a coordinated
   disclosure program (e.g., CERT/CC, HackerOne).
4. **Wait.** Give the vendor a reasonable disclosure window (90 days is
   typical) before publishing.

## Project values

Contributors to LabC2 commit to:

1. **Defender-first framing.** Features should help defenders understand
   what they need to detect, not make detection harder.
2. **No evasion features.** We deliberately reject AV/EDR bypasses,
   in-memory loaders designed to defeat scanning, AMSI/ETW tampering,
   process injection, and anti-analysis tricks.
3. **Loud over quiet.** Agents log everything, banner themselves loudly,
   and check in over standard TLS to advertised endpoints.
4. **Transparency.** All code is open, readable, and commented for learners.
5. **No real-world targeting.** Documentation never references real
   organizations as practice targets.

## If you see misuse

If you become aware of someone using LabC2 against systems they don't own
or aren't authorized to test, please:

- Report it to the project maintainers via the contact in [SECURITY.md](SECURITY.md).
- If a crime is in progress, report it to your local law enforcement
  (in the US: IC3.gov; in the EU: your national CSIRT).

## Acknowledgement

By cloning, building, or running this code you confirm that you have read
this document and agree to abide by it.
