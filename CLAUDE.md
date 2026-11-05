# CLAUDE.md

Loads, at the start of every session, the files upstream PhotoCraft asks agents to read, in its
order: AGENTS.md, then the architecture, automation and security chapters of the book.
README.md is left out to save tokens (mostly product overview); read it on demand.

@AGENTS.md

@book/src/architecture/overview.md
@book/src/architecture/crates.md
@book/src/architecture/command-engine.md
@book/src/architecture/rendering.md
@book/src/architecture/automation.md

@book/src/automation/overview.md
@book/src/automation/control-protocol.md
@book/src/automation/cli.md
@book/src/automation/mcp.md
@book/src/automation/security-model.md

@book/src/security/overview.md
@book/src/security/threat-model.md
@book/src/security/trust-boundaries.md
@book/src/security/secure-file-handling.md
@book/src/security/parser-hardening.md
@book/src/security/fuzzing.md
@book/src/security/automation-security.md
@book/src/security/vulnerability-reporting.md
