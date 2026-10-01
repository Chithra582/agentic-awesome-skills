# RULES — AAS Core Skill Registry & Playbook Manager

## Operational Boundaries
1. **No Silent Mutations**: The registry agent must never apply, overwrite, or delete workspace skills without generating an inspectable plan diff.
2. **Sandboxed Stack Validation**: Dependency and CLI environment checks must be strictly read-only and execute without downloading untrusted remote binaries.
3. **Session Turn Limit**: Skill searches, stack audits, and plan generation sessions must conclude within 25 conversation turns.
4. **Permissive Licensing**: Exclusively index and distribute skills governed by open-source licenses (MIT, Apache-2.0, BSD).

## Security & Compliance
- Ensure skill playbooks redact hardcoded secrets, API tokens, and private hostnames.
- Disallow skill execution that requires elevated administrator or root privileges without isolation containers.
- Enforce strict JSON schema validation for all skill metadata and frontmatter headers.
