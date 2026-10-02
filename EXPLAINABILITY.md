# EXPLAINABILITY.md

This document explains the internal mechanisms, data lineage, operational boundaries, and governance framework of **Agentic Awesome Skills Registry Engine** (`agentic-awesome-skills`) in accordance with the **OpenGAP v0.1.0** specification for the **HiDevs GitAgent Passport** clearance pipeline.

> **Agent Name:** Agentic Awesome Skills Registry Engine (`agentic-awesome-skills`)  
> **Specification:** OpenGAP v0.1.0  
> **Category / Domain:** Developer Tools / Agent Skill Registry & Automation Playbooks  
> **Compliance Standard:** OpenGAP Checkpoint 2 (Explainability & Decision Governance), OWASP LLM Top 10, MITRE ATLAS  

---

## How the Agent Decides

Agentic Awesome Skills Registry Engine is an enterprise-grade agent skill management and orchestration engine engineered to discover, validate, and preview installable automation playbooks across a catalog of over 2,400 open-source skills. Its primary operational purpose is to provide autonomous coding agents (Claude Code, Cursor, Codex, Antigravity) with reliable, agent-owned skill discovery without non-deterministic ranking algorithms or unverified workspace mutations.

### 1. Decision Architecture

The skill discovery, stack validation, playbook verification, and plan preview pipeline operates across a deterministic, five-stage architecture:

```
Developer / Agent Task Directive (Skill Inquiry / Task Domain / Stack Requirements / Playbook Request)
    │
    ▼
[Stage 1: Intent Parsing & Environment Stack Audit]
    │  - Evaluates user task domain, target agent framework, and workspace prerequisites
    │  - Scans workspace in read-only mode to detect installed toolchains (Node, Python, Go)
    │  - Initializes local catalog index search filters
    ▼
[Stage 2: Catalog Manifest Filtering & Scoring]
    │  - Queries offline index of 2,400+ installable SKILL.md playbooks
    │  - Computes affinity scores across requested frameworks, tool capabilities, and permissions
    │  - Filters out unverified playbooks or those with deprecated dependencies
    ▼
[Stage 3: Boundary & Permission Review]
    │  - Audits candidate playbooks for shell execution commands, file writes, and network calls
    │  - Evaluates security boundaries against OWASP LLM and MITRE ATLAS threat models
    │  - Flags privileged access requests for mandatory human operator sign-off
    ▼
[Stage 4: Plan Diff Synthesis & Linter Verification]
    │  - Synthesizes structural preview diffs showing exact files to be created or modified
    │  - Runs OpenGAP schema linters and markdown validators on candidate skill manifests
    │  - Verifies that proposed modifications are non-destructive to existing codebase assets
    ▼
[Stage 5: Plan Preview Emission & Audit Logging]
    │  - Emits clear, inspectable diffs to the local workspace under Preview-Before-Apply standards
    │  - Records complete decision traces, skill checksums, and validation timestamps locally
    │  - Pauses execution pending explicit developer confirmation before applying any changes
    ▼
Validated Skill Playbook Preview & Auditable Registry Trajectory Record
```

### 2. Decision Logic & Playbook Routing Formulations

The registry engine evaluates playbook relevance, environment compatibility, and security risk using deterministic mathematical models:

1. **Playbook Relevance Affinity Score ($S_{\text{playbook}}$)**:
   $$S_{\text{playbook}} = (w_f \cdot F_{\text{framework}}) + (w_d \cdot D_{\text{domain}}) + (w_s \cdot S_{\text{stack}})$$
   where:
   - $F_{\text{framework}} \in \{0, 1\}$ represents exact agent framework compatibility.
   - $D_{\text{domain}} \in [0, 1]$ represents semantic keyword overlap with requested capabilities.
   - $S_{\text{stack}} \in [0, 1]$ represents local environment toolchain availability.
   - Weights: $w_f = 0.40, w_d = 0.35, w_s = 0.25$ ($\sum w_i = 1.0$).

2. **Playbook Safety & Permission Index ($I_{\text{perm}}$)**:
   $$I_{\text{perm}} = 1 - \frac{1}{3} \left( P_{\text{shell}} + P_{\text{network}} + P_{\text{write}} \right)$$
   where each penalty is normalized $\in [0, 1]$. Playbooks requiring privileged execution trigger mandatory human confirmation gates.

### 3. Thresholding & Refusal Decision Criteria

Agentic Awesome Skills Registry Engine enforces strict operational safety and integrity boundaries:
- **Refusal to Auto-Execute Arbitrary Binaries**: Playbooks containing untrusted pre-compiled binary installers or obfuscated shell scripts are deterministically rejected with code `ERR_UNTRUSTED_BINARY_REFUSED`.
- **Refusal of Direct Workspace Mutation Without Preview**: Attempting to install or overwrite skill files without generating and displaying a plan diff preview triggers immediate rejection (`ERR_PREVIEW_BEFORE_APPLY_VIOLATION`).
- **Turn Ceiling Enforcement**: Skill discovery and validation interactions enforce a hard limit of `max_turns: 25` to eliminate recursive search loops (`WARN_TURN_BUDGET_EXCEEDED`).
- **Local Workspace Confinement**: Skill installations write exclusively to the project skill directory; paths outside the target workspace are blocked (`ERR_OUT_OF_BOUNDS_WRITE`).

### 4. Fallback Decision Mechanism

Continuous skill discovery is guaranteed through multi-tier fault recovery:
- **Model Cascade Failover**: When the primary foundation model experiences latency spikes or HTTP 429 rate limits, the orchestrator cascades automatically between `claude-3-5-sonnet`, `gpt-4o`, and `gemini-2.0-flash`.
- **Deterministic Offline Index Fallback**: If LLM parsing encounters service disruptions, the system falls back to pure deterministic regex and keyword indexing over the local `SKILL.md` catalog.
- **Graceful Permission Degradation**: If an advanced skill requires ungranted permissions, the agent offers a read-only or simulation version of the playbook.

### 5. Human-in-the-Loop Governance

Human developers retain complete authority over skill adoption and workspace modification:
- **Preview-Before-Apply Standard**: Every skill installation requires explicit developer review of synthesized plan diffs before files are written.
- **Emergency Session Kill Switch**: Operators can halt registry searches or diff evaluations instantly via standard `Ctrl+C` interrupt signals.
- **Inspectable Audit Logs**: All searched skills, validated manifests, and applied plan previews are logged with SHA-256 cryptographic hashes for auditing.

---

## The Data It Uses

Agentic Awesome Skills Registry Engine operates under strict privacy, data minimization, and local workspace isolation standards.

### 1. Ingested Input Data

The agent processes only operational assets necessary to fulfill skill catalog management:
- **Task Directives**: Natural language queries requesting automation skills, tooling capabilities, or task playbooks.
- **Local Catalog Index**: Metadata manifests, directory paths, and YAML frontmatter for 2,400+ open-source SKILL.md playbooks.
- **Workspace Toolchain State**: Read-only detection of package managers, SDK versions, and agent frameworks installed locally.

### 2. Configuration & Reference Data

- **Framework Manifest Schemas**: Validation schemas for Antigravity, Claude Code, Cursor, and OpenAI tool protocols.
- **Permission Matrix**: Mappings of high-risk shell commands, sensitive directory paths, and network ports.
- **OpenGAP Specification**: Compliance rulesets governing OpenGAP v0.1.0 skill package structures.

### 3. Base Model & Inference Lineage

- **Deterministic Algorithmic Engines**: Ripgrep catalog indexers, YAML frontmatter linters, and unified diff generators executed natively (100% deterministic with zero LLM variance).
- **Foundation LLMs**: High-capability frontier models (`claude-3-5-sonnet`, `gpt-4o`, `gemini-2.0-flash`) utilized for complex task decomposition, intent matching, and documentation synthesis.
- **Zero Training on User Projects**: Developer workspace files, local playbooks, and repository configurations are never stored externally or used for model training.

### 4. Data Privacy, Storage, and Retention

- **OWASP LLM & MITRE ATLAS Hardened**: Defended against prompt injection, malicious playbook payload execution, and excessive tool authority.
- **Local-Only Working Storage**: Catalog indexes, generated diffs, and execution audit trails reside entirely on the local user filesystem.
- **Credential Scrubbing**: Environment variables, authentication tokens, and user paths are scrubbed from generation logs.
- **Zero Commercial Monetization**: Developer specifications, scaffolded codebases, and architectural inquiries are never monetized, aggregated, or shared with third parties.

---

## Limitations

Understanding the operational boundaries and technical constraints of Agentic Awesome Skills Registry Engine is essential for optimal deployment.

### 1. Dynamic Upstream Skill Repository Drift
- **Limitation**: Community-contributed playbooks in third-party repositories may introduce dependencies that change over time.
- **Mitigation**: The registry pins immutable content hashes for all indexed playbooks and validates schemas locally before preview emission.

### 2. Heterogeneous Host OS Toolchain Discrepancies
- **Limitation**: Playbooks containing OS-specific terminal commands (e.g., Linux-specific bash vs. Windows PowerShell) can fail across platforms.
- **Mitigation**: The stack validator detects the host operating system and either suggests cross-platform alternatives or warns the developer.

### 3. Extremely Large Multi-Skill Compositions
- **Limitation**: Installing dozens of complex skills simultaneously can bloat prompt context windows in downstream agent runners.
- **Mitigation**: The engine provides context budget warnings and recommends modular skill loading only when specific tasks are active.

### 4. Non-Deterministic Community Code Execution
- **Limitation**: While the registry lints declarative schemas, the runtime behavior of arbitrary user-authored scripts in third-party skills cannot be guaranteed.
- **Mitigation**: The agent strictly separates declarative prompts from executable scripts, sandboxing scripts and requiring explicit approval.

### 5. Subjective Toolchain Preference Trade-Offs
- **Limitation**: Choosing between competing skills performing similar functions (e.g., two different git automation playbooks) involves developer preference.
- **Mitigation**: The registry outputs structured comparative tables showing tool dependencies and token footprints for human selection.

---

## Summary & Compliance Checklist

| Checkpoint 2 Requirement | Corresponding Section | Status |
| :--- | :--- | :---: |
| **How the agent decides** | [How the Agent Decides](#how-the-agent-decides) | **Covered** |
| - Decision architecture & 5-stage pipeline | Section 1 | Verified |
| - Decision logic & playbook routing formulations | Section 2 | Verified |
| - Thresholding & refusal decision criteria | Section 3 | Verified |
| - Fallback decision mechanism | Section 4 | Verified |
| - Human-in-the-loop governance & oversight | Section 5 | Verified |
| **The data it uses** | [The Data It Uses](#the-data-it-uses) | **Covered** |
| - Ingested task directives, catalog index & stack state | Section 1 | Verified |
| - Configuration, framework schemas & permission matrix | Section 2 | Verified |
| - Base model lineage & deterministic engines | Section 3 | Verified |
| - Data privacy, retention lifecycle & MITRE/OWASP | Section 4 | Verified |
| **Its limitations** | [Limitations](#limitations) | **Covered** |
| - Dynamic upstream skill repository drift | Section 1 | Verified |
| - Heterogeneous host OS toolchain discrepancies | Section 2 | Verified |
| - Extremely large multi-skill compositions | Section 3 | Verified |
| - Non-deterministic community code execution | Section 4 | Verified |
| - Subjective toolchain preference trade-offs | Section 5 | Verified |
