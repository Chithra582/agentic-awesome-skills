---
name: offensive-osint
description: Operational arsenal for authorized external red-team and bug-bounty recon.
license: MIT
compatibility: Requires explicit written authorization for a target scope plus the
  relevant testing tools for this technique. Docs-only; helper scripts and commands
  not bundled.
metadata:
  category: security
  risk: offensive
  source: https://github.com/elementalsouls/Claude-BugHunter
  source_repo: elementalsouls/Claude-BugHunter
  source_type: community
  date_added: '2026-09-20'
  license_source: https://github.com/elementalsouls/Claude-BugHunter/blob/main/LICENSE
  sources: hackerone_public, community, public_research
  version: 3.0.0
  triggers: '[''external recon'', ''external red team'', ''red team external'', ''attack
    surface management'', ''ASM'', ''bug bounty recon'', ''bug bounty'', ''reconnaissance'',
    ''footprinting'', ''asset discovery'', ''swagger discovery'', ''openapi discovery'',
    ''graphql introspection'', ''graphql discovery'', ''subdomain enumeration'', ''subdomain
    takeover'', ''cloud bucket enumeration'', ''bucket enum'', ''S3 enum'', ''GCS
    enum'', ''Azure blob enum'', ''identity fabric'', ''SSO discovery'', ''IdP fingerprinting'',
    ''tenant fingerprinting'', ''okta enum'', ''entra enum'', ''azure AD enum'', ''ADFS
    enum'', ''SAML metadata'', ''mobile recon'', ''APK analysis'', ''mobile attack
    surface'', ''secret scanning'', ''secret leak'', ''leaked credential'', ''github
    dorking'', ''google dorking'', ''bing dorking'', ''DDG dorking'', ''postman workspace'',
    ''stack exchange OSINT'', ''breach lookup'', ''have I been pwned'', ''HudsonRock
    cavalier'', ''infostealer'', ''dehashed'', ''intelx'', ''shodan recon'', ''censys
    recon'', ''certificate transparency'', ''crt.sh'', ''JARM'', ''favicon mmh3'',
    ''JS endpoint extraction'', ''sourcemap leak'', ''copy paste probes'', ''curl
    one-liner'', ''email security analysis'', ''SPF DMARC DKIM'', ''origin discovery'',
    ''CDN bypass'', ''WAF bypass'', ''vendor product fingerprints'', ''Citrix Netscaler'',
    ''F5 BIG-IP'', ''Pulse Secure'', ''FortiGate'', ''PaloAlto GlobalProtect'', ''Cisco
    AnyConnect'', ''VMware vCenter'', ''cloud native fingerprint'', ''Lambda function
    URL'', ''Cloud Run'', ''kubernetes exposure'', ''kubelet'', ''etcd'', ''CI CD
    exposure'', ''Jenkins recon'', ''GitLab self-hosted'', ''GitHub Actions secrets'',
    ''documentation leak'', ''Notion public'', ''Confluence anonymous'', ''Trello
    board'', ''WHOIS RDAP'', ''DNS record catalog'', ''Wayback CDX'', ''LinkedIn enumeration'',
    ''job posting tech stack'', ''Slack workspace discovery'', ''Discord server discovery'',
    ''npm token leak'', ''PyPI token leak'', ''Docker Hub leak'', ''sat imagery physical
    recon'', ''TLS deep audit'', ''JA3 JA4'', ''reverse DNS sweep'', ''IPv6 enumeration'',
    ''CVE prioritization'', ''EPSS scoring'', ''CISA KEV'', ''vulnerability prioritization'',
    ''tooling install'', ''sector specific recon'', ''healthcare DICOM'', ''finance
    SWIFT'', ''ICS SCADA'', ''Modbus'', ''BACnet'', ''post discovery workflow'', ''JWT
    triage'', ''AWS key triage'', ''GraphQL field suggestion'', ''Anthropic API key'',
    ''OpenAI API key'', ''Microsoft 365 deep'', ''Teams federation'', ''SharePoint
    enum'', ''OneDrive enum'', ''hackerone reference'', ''h1 hacktivity'', ''disclosed
    reports'', ''community bug reports'', ''prior disclosures'', ''bug bounty reference'']'
---
> **⚠️ AUTHORIZED USE ONLY**
> This skill is for educational purposes or authorized security assessments only.
> You must have explicit, written permission from the system owner before using this tool.
> Misuse of this tool is illegal and strictly prohibited.

> **Mandatory confirmation gate**
> Before running any command that probes, exploits, changes, persists on, extracts data from, or attempts credential access against a target:
> 1. Ask the user to state the exact target URL, IP, account, or resource.
> 2. Ask the user to confirm written authorization and the permitted scope.
> 3. Show the exact command(s) and explain their expected effect.
> 4. Wait for explicit confirmation in the current conversation.
>
> Without that confirmation, remain read-only and provide defensive guidance only. Prefer a sandbox, disposable VM, or controlled lab.

# Offensive OSINT — External Red-Team Arsenal

> **v3.0** — Refactored 2026-05-02 from a 4,168-line monolith into a lean SKILL.md (~400 lines) plus 15 modular reference files in `references/`. Detail content loads on demand — Claude reads only the reference files relevant to the current task.


## 0. When to use / When NOT

**Use this skill when:**
- You need concrete probe paths, wordlists, regexes, payloads, scoring rules, or tool URLs.
- You're executing reconnaissance and need the actual technical reference (vs. methodology).
- You're building a recon automation and need specific lists to seed it.

**Do NOT use this skill when:**
- The user is asking for active exploitation, post-exploitation, or anything past reconnaissance.
- The user is asking for defensive / blue-team detections.
- The target's authorization isn't established — see §1.

---

## 1. Authorization & Legal Posture

For assets the operator owns or has written authorization to assess. Soft scope check before acting against an unverified third-party target — see methodology skill §1 for the full posture.

---

## 2. Confidence Levels

- **TENTATIVE** — plausible based on indirect evidence (snippet-only dork match, single-source asset, inferred email pattern).
- **FIRM** — directly observed (subdomain resolves, HEAD-confirmed bucket exists, banner returned).
- **CONFIRMED** — verified via independent corroboration OR direct verification (live PMAK validation, multiple sources agree, listable bucket with object retrieval).

---

## 3. Output Format Conventions

Findings should carry: `id`, `module`, `asset_key`, `category`, `severity` (info/low/medium/high/critical), `confidence`, `title`, `description`, `evidence` (url + UTC timestamp + sha256 + raw ≤ 2 KiB), `references`, `remediation`. UTC timestamps everywhere.

---

## 4. Source Hygiene & Citations

URL + UTC timestamp + SHA-256 + tool version + run_id, every artifact. PNG screenshots, JSONL run logs, raw HTTP captures capped at 2 KiB body.

---

## 5. Do NOT

- Don't paste creds/PII/session tokens into cloud LLMs.
- Don't run destructive probes outside DEEP/`--aggressive`.
- Don't use validated credentials for anything except read-only liveness check.
- Don't single-source attribute.
- Don't assume vendor labels are ground truth.

---

## 6. General OSINT (curated tool refs)

- [OSINT Bookmarks](https://tools.myosint.training/) — comprehensive bookmarks.
- [OSINT Framework](https://osintframework.com/) — tool/resource directory.
- [IntelTechniques Tools](https://inteltechniques.com/tools/) — investigative suite.
- [Bellingcat Toolkit](https://www.bellingcat.com/resources/2024/09/24/bellingcat-online-investigations-toolkit/) — investigative journalism.
- [CyberSudo OSINT Toolkit](https://docs.google.com/spreadsheets/d/1EC0sKA_W9znzsxUt0wye9UYtyATXw5m8) — OSINT websites list.
- [Google Dorks](https://dorksearch.com/) — efficient Google searching.
- [Distributed Denial of Secrets](https://ddosecrets.com/) — leaked datasets.
- [Country-Specific Resources](https://digitaldigging.org/osint/) — country-targeted OSINT.


---

## How to use this skill

This skill is a **lean operational index**. Most concrete data (wordlists, regexes, dorks, endpoint catalogs, severity examples) lives in the `references/` subfolder, organized by topic.

**Workflow when this skill triggers:**

1. Read this SKILL.md to anchor on principles (§0-5), scoring rubrics (§20-21), attack-path templates (§39), and the references index below.
2. For task-specific data, **read only the reference file(s) you need** — do NOT pull all 15. Each reference is self-contained.
3. Use the `bug-bounty` skill for the local toolkit at `~/security-research/bug-bounty-resources/` and `osint-methodology` for the planning framework.

**Loading rules of thumb:**
- Single-class question (e.g., "what's the regex for AWS keys?") → load `secret-patterns.md` only.
- Multi-class engagement (e.g., "do an external recon on target.com") → load `probes-and-wordlists.md` first, then add others as the engagement narrows.
- Severity / triage question → load `severity-matrix.md`.

---

## References Index

| File | Coverage | Trigger phrases |
|---|---|---|
| `probes-and-wordlists.md` | API/Swagger/GraphQL paths, cloud-bucket arsenal, JS guess-paths, vendor & cloud-native fingerprints, K8s/CI-CD exposure, doc/wiki leaks, WHOIS/RDAP, DNS catalog, Wayback CDX, copy-paste curl probes, email security analysis, origin/CDN bypass | swagger discovery, graphql introspection, subdomain takeover, cloud bucket enum, S3/GCS/Azure enum, kubernetes exposure, CI CD exposure, vendor fingerprint, WHOIS RDAP, Wayback CDX, copy paste probes, curl one-liner |
| `identity-fabric.md` | Concrete endpoints for Entra/Okta/ADFS/Google/SAML, M365 deep (Teams federation, SharePoint, OneDrive), GraphQL field-suggestion enumeration, user-enum patterns | identity fabric, SSO discovery, IdP fingerprinting, okta enum, entra enum, azure AD enum, ADFS enum, SAML metadata, Microsoft 365 deep, Teams federation, SharePoint enum, OneDrive enum, graphql field suggestion |
| `secret-patterns.md` | 48-pattern secret-regex catalog (AWS, GCP, GitHub PATs, Stripe, Slack, JWT, private keys, Anthropic/OpenAI/HuggingFace, Cloudflare, DigitalOcean, npm, PyPI, Docker Hub, Atlassian, DataDog, Sentry, ngrok) with severity & FP notes | secret scanning, secret leak, leaked credential, JWT triage, AWS key triage, Anthropic API key, OpenAI API key |
| `secret-validators.md` | 9 read-only secret validators + post-discovery enumeration workflows for AWS/GitHub/Slack/Postman/JWT/Anthropic/OpenAI/npm/Atlassian/DataDog | secret validation, post discovery workflow, AWS key triage, JWT triage |
| `dork-corpus.md` | 80+ Google/Bing/DDG dork templates across 9 categories + 13 GitHub code-search dorks tailored for targets | google dorking, bing dorking, github dorking, dork corpus |
| `recon-stack.md` | Subdomain-source stack (passive & active), infrastructure & attack-surface OSINT (Shodan/Censys/crt.sh/JARM/favicon mmh3), TLS deep audit, reverse DNS, IPv6 enumeration | subdomain enumeration, certificate transparency, crt.sh, shodan recon, censys recon, JARM, favicon mmh3, TLS deep audit, JA3 JA4, reverse DNS sweep, IPv6 enumeration |
| `breach-and-credentials.md` | Breach & leak data sources (HudsonRock, HIBP, DeHashed, IntelX, infostealer logs), email-pattern inference, email-harvest source stack | breach lookup, have I been pwned, HudsonRock cavalier, infostealer, dehashed, intelx, email harvest |
| `people-osint.md` | Search engines, username & email investigation, people search, phone OSINT, social media, public records & company info | username investigation, people search, phone OSINT, social media OSINT, public records |
| `saas-public-surfaces.md` | Postman public workspace search (verified endpoint), Stack Exchange OSINT sweep, public SaaS dork stack (Notion, Confluence, Trello) | postman workspace, stack exchange OSINT, Notion public, Confluence anonymous, Trello board |
| `specialized-osint.md` | Threat intel & IOCs, cryptocurrency OSINT, media intelligence, geospatial intelligence, regional search engines, Telegram & messaging intelligence | threat intel, IOCs, cryptocurrency OSINT, media intelligence, geospatial OSINT, regional search, Telegram intelligence |
| `recon-techniques.md` | LinkedIn employee enumeration, job-posting tech-stack analysis, Slack/Discord/Telegram workspace discovery, package-registry leak hunting (npm/PyPI/Docker Hub/Quay/GHCR), sat imagery for physical recon | LinkedIn enumeration, job posting tech stack, Slack workspace discovery, Discord server discovery, npm token leak, PyPI token leak, Docker Hub leak, sat imagery physical recon |
| `severity-matrix.md` | 80+ worked examples mapping observed conditions → finding severity (CRITICAL/HIGH/MEDIUM/LOW/INFO) | severity decision, finding severity, severity matrix |
| `sector-notes.md` | Recon notes for healthcare (DICOM), finance (SWIFT), ICS/SCADA (Modbus/BACnet), IoT, government | sector specific recon, healthcare DICOM, finance SWIFT, ICS SCADA, Modbus, BACnet |
| `tooling-install.md` | Quick-install one-liners for Subfinder, Amass, httpx, nuclei, gau, katana, gowitness, dnsx, mapcidr, naabu, sslyze, testssl.sh, etc. | tooling install, install subfinder, install nuclei, install httpx |
| `helpers-and-automation.md` | AI-assisted OSINT, archiving & evidence preservation, automation & workflow patterns, cross-module sidecar coordination, runnable secret_scan.py / h1_reference.py / dashboard.py helper notes | AI-assisted OSINT, evidence preservation, automation, sidecar, hacktivity reference, recon console |


---

## 20. Endpoint Interest Score — 0–100 rubric

For every classified endpoint (§22 in methodology skill), apply this rubric:

| Signal | Points | Conditions |
|---|---|---|
| **Unauth write** | +40 | POST/PUT/DELETE/PATCH endpoint returns 200/201/202/204 anonymously. |
| **Open GraphQL introspection** | +35 | `__schema` query returns full type list anonymously. |
| **Verb tampering bypass** | +30 | OPTIONS reveals method not documented; that method is accessible. |
| **Reflected CORS + credentials** | +25 | `Access-Control-Allow-Origin` reflects request `Origin` AND `Access-Control-Allow-Credentials: true`. |
| **Sensitive keyword in path** | +20 | Path matches one of: `admin`, `internal`, `debug`, `user`, `password`, `token`, `key`, `export`, `upload`, `backup`, `config`, `secret`, `private`, `delete`, `purge`, `wipe`. |
| **Schema leak in error** | +20 | Response body contains stack trace, ORM error class, framework signature (e.g., `ActiveRecord::RecordNotFound`, `org.hibernate.exception.*`, `django.db.utils.IntegrityError`). |
| **API key in URL** | +15 | Path or query string contains `api_key=`, `apikey=`, `token=`, `access_token=`. |
| **Wildcard CORS** | +10 | `Access-Control-Allow-Origin: *`. |
| **Missing rate-limit headers** | +10 | No `RateLimit-*` / `X-RateLimit-*` headers; no `Retry-After` after rapid requests. |

**Thresholds:**

| Score | Severity |
|---|---|
| ≥ 90 | **CRITICAL** |
| 70–89 | **HIGH** |
| 50–69 | MEDIUM |
| 25–49 | LOW |
| < 25 | INFO |

For score ≥ 70, attach an `attack_path_hint` in evidence (see §29).

---

## 21. Mobile App Ownership Confidence — 0–100 rubric

Before running deep APK static analysis, score whether the discovered app actually belongs to the target. Threshold: **≥70 = accept**.

| Signal | Points |
|---|---|
| Package reverse-DNS matches target domain (e.g., `com.acme.android` ⟂ `acme.com`) | +40 |
| Developer email is `<anything>@<target-domain>` | +25 |
| Developer website URL is the target domain (or a confirmed sibling brand domain) | +20 |
| App name contains a brand keyword from operator-supplied brand list | +10 |
| App has ≥ minimum review-score threshold (default 20 reviews) | +5 |

Apps below threshold are tagged `mobile_review_pending` and shown but not analyzed. Operator can re-score with `--mobile-ownership-threshold 50` for noisier collection.

---


---

## 39. Attack-Path Hint Patterns

When emitting a HIGH/CRITICAL API endpoint finding (score ≥ 70), include a one-sentence `attack_path_hint` in evidence so the operator knows where to start exploiting. Templates:

| Trigger | Attack-path hint |
|---|---|
| Unauth POST / PUT / DELETE | *"Unauthenticated {method} {path} — try IDOR + privilege escalation; check whether numeric IDs are sequential or guessable."* |
| Open GraphQL introspection | *"Open GraphQL introspection on {path} — enumerate mutations, look for `createUser`, `setRole`, `transferFunds`-shaped names; pivot to broken-auth or business-logic flaws."* |
| Reflected CORS + creds | *"Reflected CORS with credentials on {path} — host CSRF page on attacker-controlled origin; victim's browser will leak {sensitive-data-hint}."* |
| Wildcard CORS + sensitive | *"Wildcard CORS on {path} returning user-tied data without creds — exfiltrate via cross-origin fetch from any page victim visits."* |
| Verb tampering | *"Verb tampering: {hidden-method} allowed on documented-{visible-method}-only endpoint → likely missing-method-check authz bug; try {hidden-method} {path} with valid auth."* |
| API key in URL | *"API key in URL: `?{param}=...` — token leaks to access logs, browser history, Referer headers, third-party CDNs. Check Wayback / Google for cached copies."* |
| Schema leak in error | *"Schema leak in error response — framework signature `{framework}` exposed; map to known {framework} vulns and craft targeted payloads."* |
| Sensitive keyword | *"Path contains '{keyword}' — review for direct object reference, mass-assignment, or hidden admin functionality."* |
| Open RTDB Firebase | *"Open Firebase RTDB at https://{project}.firebaseio.com/.json — read everything, then test write at `/<random-key>.json` with PUT to gauge ACL scope."* |
| Listable cloud bucket | *"Listable {provider} bucket `{bucket}` — recursive object listing + content-type analysis; look for backups, logs, customer data, AWS keys in JSON configs."* |
| .git exposed | *"Exposed .git/config on {host} — reconstruct repository with git-dumper or githacker; full source history."* |
| .env exposed | *"Exposed .env on {host} — grep for `_KEY`, `_SECRET`, `_TOKEN`, `_PASSWORD`; validate all credentials read-only via §23 validators."* |
| /actuator/env | *"Spring Boot /actuator/env exposed — dump environment variables; look for `spring.datasource.password`, JWT secrets, cloud creds."* |
| /actuator/heapdump | *"Spring Boot /actuator/heapdump exposed — download HPROF, run `jhat` or VisualVM, search for cleartext secrets in heap strings."* |
| Open Elasticsearch | *"Open Elasticsearch on {host}:9200 — `/_cat/indices?v` for index list; sample documents from each high-value index; test write to `/test-idx/_doc` to gauge ACL."* |
| Open Redis | *"Open Redis on {host}:6379 — `INFO`, `KEYS *`, sample reads; check for write access via `CONFIG SET` then `BGSAVE` to write `authorized_keys`."* |
| Open MongoDB | *"Open MongoDB on {host}:27017 — `show dbs`, `show collections`, sample find queries; check user collection for password hashes."* |
| Subdomain takeover | *"CNAME for {host} points to unclaimed {provider} resource → register `{takeover-target}` on {provider} to serve content from {host}; pivot to phishing or content injection on the trusted domain."* |
| Open kubelet | *"Open kubelet on {host}:10250 — `GET /pods` to list; `POST /run/<ns>/<pod>/<container>` for in-container exec without K8s API auth."* |
| Open etcd | *"Open etcd on {host}:2379 — `etcdctl get / --prefix --keys-only` for full cluster state; secrets stored under `/registry/secrets/`."* |
| K8s API anonymous | *"Kubernetes API on {host}:6443 with anonymous-auth — `kubectl --server=https://{host}:6443 --insecure-skip-tls-verify get pods --all-namespaces`."* |
| Citrix unpatched | *"Citrix NetScaler version {ver} on {host} — vulnerable to CVE-{cve} (KEV-listed); see vendor advisory; do not exploit but flag for client immediate patching."* |
| F5 BIG-IP TMUI exposed | *"F5 BIG-IP TMUI on {host} reachable; CVE-2022-1388 / CVE-2023-46747 KEV applicable; advise immediate patching to vendor-released hotfix."* |
| VMware vCenter accessible | *"vCenter at {host} accessible without VPN; CVE-2021-21972 RCE if unpatched; check version banner."* |
| Cloud function URL unauth | *"AWS Lambda Function URL at {url} accessible anonymously — review IAM auth configuration; if unauthenticated by design, audit input validation aggressively."* |
| npm typosquat candidate | *"Package name `{candidate}` is unregistered + similar to target's published `{official}` — typosquat takeover risk; advise client to defensively register."* |
| DMARC missing/permissive | *"DMARC `p=none` on {domain} — spoof of `{anything}@{domain}` deliverable to recipients; recommend enforcement to `p=quarantine` or `p=reject` after observing reports."* |
| Live AI API key (Anthropic/OpenAI) | *"Validated `sk-{provider}-...` key with model access — quota cost can be exfiltrated; rotate immediately + audit usage logs in provider console."* |
| Public Slack invite link | *"Slack workspace invite link discoverable via search engine — anyone can join the workspace without approval; trivially access internal channels."* |
| Open Docker registry | *"Public Docker registry at {host} — `GET /v2/_catalog` lists images; pull and scan layers for embedded secrets."* |
| Telegram bot token live | *"Telegram bot token validated — `getUpdates` reveals bot recipients (admin chats); if `getMe` shows bot is in channels, full message read access."* |
| Sourcemap with `sourcesContent[]` | *"Sourcemap on {host} includes embedded original sources — full frontend code reconstructable; grep for inline secrets and internal hostnames."* |

---


---

## 49. Skill Self-Test

Drop these prompts into a fresh Claude session to verify the skill loads correctly.

1. *"What paths should I probe to find Swagger or OpenAPI specs on a webapp?"* → §16.1.
2. *"Give me the GraphQL introspection query I should POST."* → §16.2.
3. *"What are the high-risk ports to flag from a Shodan scan?"* → §16.3.
4. *"Show me the secret regex cata

<!-- Truncated for OpenGAP token limits -->
