# EXPLAINABILITY — AAS Core Skill Registry & Playbook Manager

> **Admissibility & Transparency Report for OpenGAP / Agent Passport**  
> *Agent Name:* AAS Core Skill Registry & Playbook Manager (`aas-core-skill-registry`)  
> *Specification:* OpenGAP v0.1.0  
> *Domain:* Developer Tools / Skill Registry & Playbook Manager  

---

## 1. Overview & Operational Purpose

AAS Core Skill Registry & Playbook Manager is an enterprise-grade agent skill management and orchestration engine engineered to discover, validate, and preview installable automation playbooks across a catalog of over 2,400 open-source skills. Its primary operational purpose is to provide autonomous coding agents (such as Claude Code, Cursor, Codex, and Antigravity) with reliable, agent-owned skill discovery without non-deterministic ranking algorithms or unverified workspace mutations.

By enforcing the Preview-Before-Apply standard and verifying every playbook against OpenGAP schema requirements, the agent ensures that development teams can adopt modular automation playbooks with verified trust boundaries, minimal token overhead, and zero unvetted code execution.

---

## 2. How the Agent Decides (Decision-Making Logic)

AAS Core Skill Registry & Playbook Manager operates across a deterministic, multi-stage decision pipeline:

```
[Stage 1: Intent & Stack Audit] ──> [Stage 2: Catalog Manifest Filter] ──> [Stage 3: Boundary & Permission Review]
                                                                                                  │
                                                                                                  ▼
[Stage 6: Trace Log & Commit]   <── [Stage 5: Plan Diff Preview Emit]  <── [Stage 4: Linter & Schema Verification]
```

### 2.1 Intent Parsing & Environment Stack Audit
- **Decision:** The agent inspects user requirements and conducts a read-only audit of the target workspace to discover installed languages, package managers, and agent runners.
- **Rules:** If target workspace prerequisites are unsatisfied, pause and report missing dependencies. Never attempt automated package installation without explicit user approval.

### 2.2 Catalog Filtering & Playbook Selection
- **Decision:** Query the local catalog index to identify playbooks corresponding to the requested task domain and verified tech stack.
- **Rules:** Select playbooks strictly matching the target agent framework. Filter out unmaintained or deprecated playbooks that fail strict validation checks.

### 2.3 Permission & Boundary Review
- **Decision:** Evaluate requested tools, shell scripts, and external network permissions specified in candidate playbooks.
- **Rules:** Flag any skill requesting root/sudo access, raw socket operations, or unconstrained filesystem deletion for mandatory human sign-off.

### 2.4 Plan Diff Synthesis & Preview Emission
- **Decision:** Synthesize proposed workspace modifications into a structured preview diff before applying changes.
- **Rules:** Render unified diffs for all affected files. Present clear instructions on how the human supervisor can accept, modify, or abort the proposed plan.

---

## 3. Data Flow & Boundary Privacy

The registry engine operates entirely locally within the user workspace and adheres to strict boundary isolation standards.

| Component / Boundary | Data Received | Processing & Retention | Destination / External Transmission |
|---|---|---|---|
| Workspace Inspector | File trees, package manifests, CLI versions | Ephemeral in-memory capability comparison; zero persistence | Local runtime memory |
| Catalog Database | Local SKILL.md playbooks, metadata manifests | Read-only disk queries; zero external telemetric reporting | Local search buffer |
| Plan Diff Engine | Selected skill names, destination directory paths | In-memory diff calculation; written to local plan artifact | Local filesystem preview file |
| Audit Logger | Selected skill IDs, execution timestamps, validation states | Structured JSON audit trail written to workspace logs | Local disk log storage |

AAS Core Skill Registry & Playbook Manager complies with operational security and privacy standards:
- **No Cloud Data Exfiltration:** The agent operates completely offline or within local network boundaries, transmitting no repository source code to external servers.
- **Epistemic Isolation:** Discovery queries, evaluation plans, and skill manifests are processed in discrete session contexts, eliminating cross-session memory contamination.
- **Sanitized Model Payloads:** Model queries contain only abstract capability names and schema definitions, redacting proprietary workspace paths or source code.
- **Data Minimization:** Only repository metadata required for prerequisite validation (e.g. package.json dependencies) is inspected during stack audits.

---

## 4. Known Limitations & Failure Modes

Reviewers, auditors, and users should note the following operational constraints:

1. Stale Local Catalog Indexes
   - *Limitation:* The offline skill catalog reflects the repository state at checkout time and does not automatically fetch real-time upstream community additions.
   - *Mitigation:* The agent provides explicit catalog timestamp indicators and instructions for running local git pull synchronizations.

2. Shell Environment Discrepancies Across Platforms
   - *Limitation:* Skills containing auxiliary bash scripts may encounter syntax or path differences when executed on native Windows command prompts.
   - *Mitigation:* The agent audits the active operating system and advises using WSL, Git Bash, or cross-platform PowerShell wrappers where appropriate.

3. Context Window Token Inflation from Excessive Skills
   - *Limitation:* Loading dozens of comprehensive skill instructions simultaneously can consume significant LLM context tokens and degrade reasoning accuracy.
   - *Mitigation:* The agent enforces modular, task-specific skill selection and warns developers when total active skill instructions exceed 10,000 tokens.

4. Unverified Third-Party Community Playbooks
   - *Limitation:* While frontmatter schemas are strictly linted, community-authored instructions may occasionally propose non-optimal implementation techniques.
   - *Mitigation:* Every skill displays verified source repository provenance and author attribution, and requires human plan inspection prior to application.

---

## 5. Verification, Safety & Human Oversight

AAS Core Skill Registry & Playbook Manager incorporates robust verification, safety gates, and human oversight controls across every layer of execution:

- **Real-Time Human Approval Gate:** All workspace mutations, skill additions, and deletions require authenticated confirmation following the presentation of the plan diff.
- **Emergency Session Interrupt:** Operators can abort catalog inspections, stack validations, or planning routines instantly using standard break signals.
- **Step Quota Guardrails:** Strict session turn caps (maximum 25 turns) prevent recursive discovery loops or automated multi-turn drift.
- **Structured Audit Logging:** Every catalog query, stack validation result, and generated plan diff is recorded in structured JSON logs for audit review.
