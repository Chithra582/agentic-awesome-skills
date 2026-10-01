---
name: redteam-mindset
description: Red-team operator discipline
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
  sources: authorized-engagement
  report_count: '1'
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

## When to use this skill

Trigger when:
- Engagement scope says "red team", "adversary emulation", "assume breach", "TIBER-style"
- You're tempted to retract a finding because reproducibility failed once
- You're tempted to call a defense "working as intended" instead of probing further
- You hit a blocker (captcha, rate limit, WAF, CA-block, lockout) and consider stopping
- You're about to spend time on IDOR/CSRF/XSS instead of access-yielding bugs
- You found a vuln on app A and there are sister apps B, C, D you haven't touched

DO NOT use for:
- Bug bounty programs (use bug-bounty skill — different scope rules)
- WAPT/PCI-style assessments (use OWASP-aligned skills)
- Pure compliance audits

---

## The one-line summary

**Red team scope = "gain access, prove impact". Bug bounty / WAPT scope = "find a bug, write a report".**

These produce DIFFERENT decisions at every blocker. Internalize the difference before starting.

---

## DO NOT STOP — primary directive (read this twice)

The 9 corrections below are nuance. THIS is the core. If you only retain one thing from this skill, retain this.

### Authorization given at engagement start covers the entire engagement.

Do not insert mid-engagement permission gates. If the user chose a mode at the start ("full engagement", "Option D", "go deep", "test these assets") that authorization stands until the engagement window expires or the user explicitly revokes it. **`AskUserQuestion` mid-loop after the user already chose a path is a stall, not a check.**

### Discipline rules are about correctness of FINDINGS, not throttling of EFFORT.

Read this twice. The bundle's discipline rules — OOB-Or-It-Didn't-Happen, Marker Discipline, Body-Diff, Pre-Severity Gate, Server-Policy-vs-State, Statistical Sampling — all answer one question: *"is this signal actually a finding?"*. **None of them answer the question *"should I send the next probe?"*** The answer to that second question is always *yes*, unless scope is exhausted or the user said stop.

### "Stop at PoC" means stop ESCALATING, not stop TESTING.

When you confirm impact on bug class X, don't keep escalating class X (no need to pivot from `id=1` IDOR to dumping the whole user table). But classes Y and Z have not been tested yet. Run Y. Then Z. Then the rest of the Pattern Library.

### The data-minimization boundary is AS LOUD as "DO NOT STOP". Read this twice.

"Keep digging" applies to **coverage** (untested surfaces, classes, hosts), NOT to **extraction**. These pull in opposite directions and the persistence directive must NOT bleed into over-collection:

- **An access/exfil vulnerability is proven by the MISSING CHECK, not by the volume of data you copy out.** 3 records that should have required auth = complete proof. 3,000 records = the same finding + a liability you created. Pulling more never strengthens the finding.
- **"Keep digging" = test the next endpoint family / the next host / the next class.** It does NOT mean "enumerate every record from the endpoint you already broke." Breadth of *coverage*, not depth of *theft*.
- **The data usually belongs to the target's customers / fourth parties** (their clients, their users' PII, partner financials). They didn't sign the SOW/program. Copying their data onto your host is harm you shouldn't create when the point is already made.
- **A client/program owner saying "dig more / that's not enough" does NOT override this.** Push back: confirm the finding is already conclusive, explain that volume ≠ evidence, and offer harm-minimal alternatives that genuinely add value — a `totalCount` (a number, not the data), proof a *second endpoint family* is affected (breadth), or quantified blast-radius. Get the MINIMUM evidence that satisfies the stated goal.
- **Classify what you did pull precisely** (e.g. "B2B client-inventory data" ≠ "consumer PII") — it changes how the owner triages and discloses, and overstating erodes trust.

The persistence directive and this boundary are not in tension once you separate the two axes: **never stop covering surface; always stop at minimum-necessary extraction.** Authorized ≠ unrestricted.

### Marker Discipline ≠ "one probe per surface."

Marker Discipline is about WHICH payloads to use (synthetic, identifiable, recoverable) — never about HOW MANY. A hardened target needs MORE marker-discipline probes than a soft one, not fewer. If the bundle's `hunt-sqli` Pattern Library lists 12 SQLi classes, you run 12 marker-discipline probes per parameter, not 1.

### Self-throttling anti-patterns — flag immediately if you catch yourself doing any of these

These all came out of a real engagement (authorized-engagement revalidation, 2026-05-17) where this skill *existed* and was *loaded* and the operator still self-throttled. The lesson is to make the failure modes explicit:

1. **Asking "want me to continue?" mid-run** after the user already chose Option D / full engagement / "go deep". The answer they gave at start IS the answer.
2. **Stopping at first-class-returning-401/403.** The bundle has ≥12 auth-bypass classes (header tricks, method tampering, parameter pollution, JSON parser confusion, race on session create, mass-assignment on optional fields, X-Forwarded-Host SSRF in SAML callback, alg=none JWT, audience confusion, scope claim manipulation, refresh-token replay, device-code flow). Run them all per surface.
3. **"Interesting constant token, not chased."** If you see a token, hash, ID, or fingerprint that's constant across what should be varying responses, that's a *lead*, not an *artifact*. `GET` it. Decode it. Pass it back. A `view.php?view=<constant-md5>` redirect is a session/auth/error-key signal, not noise.
4. **Reading robots.txt for cross-template signals and NOT READING the Disallow lines.** A 469-line robots.txt is a developer-curated map of every path they don't want public. Every Disallow line is a probe target.
5. **Treating soft-404 as "noted."** A 37 KB body inside a 404 status is leaking the home page or worse. Read it. Grep it. Diff it against the home page.
6. **"OpenAPI exposed → finding logged"** with only 4 of N endpoints probed. Every endpoint × every relevant test class. The OpenAPI spec is the attack-surface map handed to you; not running it is throwing away a free recon.
7. **"APK retest deferred — needs tooling."** `brew install jadx`, apkpure direct download, `apk-redteam-pipeline` already documents the flow. Five minutes of setup, not "another session."
8. **Volume framed as a problem.** For an authorized engagement, 3,000 well-tagged requests through Burp is normal cadence. Bug-bounty hunters at full pace exceed that per *hour*. The question to ask is *"have I run every test class on every live surface,"* not *"have I sent too many requests."*
9. **Inserting `AskUserQuestion` at any decision point inside an active engagement loop.** If the user picked a mode at start, that mode is in effect until revoked. Choosing operationally between e.g. SAML acs raw POST vs SAML acs replay is a *technical* decision the operator can make and document — it does not require user pre-approval.
10. **Skill-gap-as-stop-condition.** "No `hunt-zoho` skill exists, so I logged a v1.1 gap and moved on." NO. If a hunt-* skill doesn't exist for a discovered tech stack, do the same work *manually* using the vendor's public check matrix. Log the gap in v1.1 roadmap *and* run the checks now.

### Real-engagement cadence — what a complete sweep per live host actually looks like

Per live host, before declaring the host complete:

- Top-100 path probe (admin, api, login, /.git, /.env, server-status, swagger, openapi.json, /docs, /actuator, /healthz, /metrics, /debug, /trace, /env, /heapdump, /threaddump, robots.txt, sitemap.xml, /.well-known/*, common-CMS-paths per fingerprint)
- robots.txt content **read** — every Disallow becomes a probe target
- sitemap.xml content **read** — every entry becomes a probe target
- JS bundles harvested — grep'd with the FULL secret-regex catalogue (Firebase, AWS, GCP, JWT, Stripe, GitHub, generic high-entropy strings), route extraction, API-endpoint extraction
- Source-map variant paths checked (`/*.js.map`, `/static/js/*.js.map`, `/_next/static/*.js.map`, `/build/*.js.map`)
- For every form discovered: full SQLi marker-discipline sweep (12+ classes), auth-bypass class sweep (12+ classes), CSRF, parameter pollution, mass-assignment, race condition on state-changing submission
- For every API endpoint discovered (from JS, OpenAPI, swagger, network capture): HTTP method tampering, content-type tampering, JWT alg=none, alg=HS256-with-RS256-key, audience confusion, prototype pollution, race conditions on idempotency-violating ops
- For every SaaS tenant identified: vendor-specific check matrix from the vendor's known-vuln catalogue — even when no dedicated hunt-* skill exists yet
- Identity fabric: GetUserRealm, OpenID well-known, autodiscover-v2, federation behavior testing, sister-brand-TLD pivot for shared tenant, OneDrive-based user enum (no lockout risk)
- Mobile apps: pull every APK in the developer's catalogue, jadx decompile, secret + endpoint + cert-pin grep, exported-component enum

**If you've done less than this per host, you have not finished the host. The engagement is not done until every host is finished.**

---

## Mindset correction #1 — The blocker is data, not the stop sign

**Anti-pattern (what I did wrong):**
> "Recheck under load showed no timing differential — recanting the SQLi as indeterminate."

**The correct frame:**
> "The original 3-sample baseline (σ = 32 ms) with three distinct SLEEP payloads each adding +6 s is statistically definitive. The recheck failure is data — investigate the *delta*, not retract the finding."

When a defense suddenly appears mid-engagement:
1. **Original PoC artifacts are forever** — capture them BEFORE recheck. Screenshots, request/response pairs, timing samples.
2. **Diff the response** — body size, headers, cookies, response time. The change tells you what the client deployed (WAF rule? Hotfix? Geo block?).
3. **The deployed mitigation is itself a finding** — positive operational observation about IR responsiveness.
4. **Try alternative vectors** — slower-paced timing, encoded keywords, different injection contexts, cookie injection, header injection.
5. **Document both states** — "vulnerable at T0, mitigated at T0+30min, mitigation likely at WAF (bypassable)".

**Rule:** never retract a finding on first reproducibility failure. Investigate why before declaring false positive.

---

## Mindset correction #2 — Sister-app pattern recognition

**Anti-pattern:**
> "I confirmed SQLi on /app-a/. Moving on to other tasks."

**The correct frame:**
> "Same backend, same code template likely → /app-b/, /app-c/, /app-d/, /app-e/ (sibling apps on the same employee-portal host) are all probable. Test them with the SAME payload."

When you confirm a vuln on app A:
1. **Identify shared infrastructure** — same IP, same load balancer, same TLS cert, same response headers, same session cookie name, same login form HTML.
2. **Identify shared code template** — same form fields, same error messages, same view structure, same framework version.
3. **Sweep all sisters with the SAME exploit payload immediately.**
4. **Document the class of vulnerability** — "vulnerability is in shared form-handler template across N apps", not just one finding.
5. **Recommend class-fix** — fix the shared template, not just one app.

The authorized-engagement case: SQLi confirmed on one sub-app (`<app-A>`); four sibling sub-apps (`<app-B>`, `<app-C>`, `<app-D>`, `<app-E>`) sit on the same employee-app host with similar form patterns — likely all share the same vulnerable template. Should have been a multi-app finding.

---

## Mindset correction #3 — WAPT vs Red Team scope discipline

**Skip these in red team scope (they don't yield access):**
- IDOR (cross-user read/write *without* an ATO / privilege-escalation / sensitive-data-exfil chain — WAPT class). NOTE: IDOR is NOT a blanket-skip. IDOR on admin/role-assignment endpoints yields privilege escalation; on password-reset or session objects yields ATO; on file-download or PII handlers yields exfil; on token/key endpoints yields credential capture. Pursue those per the decision rule below.
- CSRF (state-change-via-tricked-user — WAPT class)
- Reflected XSS (without account-takeover chain — WAPT class)
- Missing security headers (WAPT class)
- Cookie hardening flags (WAPT class)
- Verbose error messages without sensitive data (WAPT class)
- DoS (out of scope per engagement rules typically)
- Username enumeration (intel-gathering, not access)

**Pursue these (they DO yield access):**
- SQL injection (data exfil → DB creds → lateral)
- Command injection / RCE (foothold)
- File upload → webshell (foothold)
- LFI/RFI (config reads → DB creds → access)
- SSRF (cloud metadata → IAM → cloud access)
- Authentication bypass (parameter manipulation, JWT alg=none, header injection)
- Hardcoded credentials in mobile/JS bundles
- Default credentials on admin panels
- SAML XSW / signature stripping (session hijack)
- Cisco ASA / Citrix / Pulse / Fortinet SSL VPN CVEs (network foothold)
- ManageEngine / Confluence / Atlassian RCE CVEs (foothold)
- Kerberoasting / AS-REP roasting (post-foothold, but enumerate from outside if possible)

**Decision rule:** if the bug, exploited fully, doesn't lead to a session/token/foothold or sensitive data exfil, it's WAPT-class — note it briefly but don't burn time on it.

---

## Mindset correction #4 — Aggressive default, not conservative default

**Anti-pattern:**
> "Tested 30% of the websites and called it comprehensive."

**The correct frame:**
> "Until I've actively probed every login form, every API endpoint, every parameter, every CVE-matched version, the engagement is not done."

Aggressive defaults:
1. **Probe every live host** for top 20 paths (admin, api, login, /.git, /.env, server-status, swagger, openapi.json, robots.txt, /actuator, /healthz, etc.)
2. **For every login form discovered**, attempt 1-of-leaked + 1-of-spray-pattern + SQLi + auth-bypass-via-parameter-tampering
3. **For every JS bundle**, grep for hardcoded API keys, JWT, base URLs, hidden endpoints, admin paths
4. **For every API endpoint**, check OPTIONS preflight, missing-auth response, alg=none JWT, X-Forwarded-User header injection
5. **For every mobile app**, decompile + grep for secrets + check pinned certs + identify exported components
6. **For every "out of scope" SaaS** that's on a corp subdomain, confirm with client — vendor-managed doesn't mean immune (CVE-2022-47966 went unpatched on many on-prem ME-SDP installs)

**Rule:** if you haven't actively probed *every* live surface (not just a sampled fraction), you haven't done red team yet — you've done recon.

---

## Mindset correction #5 — Persistence beats elegance

**Anti-pattern:**
> "Tesseract failed on 3 captchas, gave up, declared captcha bypass not feasible."

**The correct frame:**
> "Tesseract failed. Decision tree: try preprocessing (binarize, denoise, upscale, multi-PSM), then trained-model OCR, then paid solving service ($5/mo for engagement-grade volume), then session-bound captcha replay attack. A real attacker WILL invest the $5."

Decision-tree for blockers:

**Captcha:**
1. Omit field → check if required
2. Empty value → check validation
3. Reuse value across multiple submits → check session-bind
4. Tesseract with preprocessing
5. Trained-model OCR (deep-text-recognition-benchmark, calamari-OCR)
6. Paid solving service (2captcha API, anti-captcha)
7. Audio captcha if available (much weaker)

**WAF:**
1. Slower pace
2. Encode the payload (URL, hex, base64, mixed case)
3. Different injection context (cookie, header, JSON)
4. Different HTTP verb
5. Different content-type (multipart, application/json)
6. Bypass at the host level (X-Forwarded-Host, X-Original-URL)
7. Probe origin server directly (find via certificate transparency)

**Rate limit:**
1. IP rotation (multiple cloud regions)
2. User-Agent rotation
3. Slower pace + jitter
4. Distribute across multiple TLS sessions

**Slow target (timing-based exfil too slow):**
1. Different injection point (avoid per-row SLEEP context)
2. BENCHMARK or GET_LOCK as alternate timing oracle
3. Error-based extraction
4. OOB DNS callback (interactsh)
5. Faster network (cloud VM in same region as target)
6. Run dumper unattended overnight; deliver partial results

**Rule:** when one path fails, the next move is "another vector to the same goal", not "documented as not vulnerable". A real adversary doesn't have an engagement window.

---

## Mindset correction #6 — Real-time IR observation is a finding

When the client SOC patches mid-engagement (you observe a vulnerability disappear during your test):
- **Treat it as evidence**, not as a failure
- **Capture timestamps** before and after the change
- **Document as positive operational finding** — "client SOC detected and mitigated within X minutes; mitigation deployed at WAF/code level"
- **Verify the mitigation depth** — WAF rule (bypassable) vs code fix (real)
- **The original PoC remains th

<!-- Truncated for OpenGAP token limits -->
