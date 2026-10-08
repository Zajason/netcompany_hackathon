# Track 3: Cybersecurity · "Shadow AI Detection & Data Protection"

> **Official brief:** *"Building a tool to Detect Rogue AI and Protect Corporate Data. This event invites students to design and implement a Shadow AI Detection & Data Loss Prevention (DLP) platform that continuously monitors user activity, application traffic, and API calls to identify unauthorized or 'rogue' AI tools operating outside corporate governance."*
> The full problem statement and evaluation focus areas will be revealed at kick-off (12 Nov). Likely judge leads: **Sokratis Ioannidis** (Head of Cybersecurity Technology & **Red Team**) and **Periklis Nikolaidis** (Lead OT Security Engineer). Expect them to try to **evade** your detection.

---

## 1. Decoding the brief

There are two halves, and teams that only do one will lose:

| Half | Question it answers | Signals |
|---|---|---|
| **Shadow AI detection** | *Which* AI tools are employees using, *who*, *how much*, and are they sanctioned? | DNS, proxy/SNI, TLS fingerprints, endpoint processes, browser activity, API keys in traffic or code, SaaS OAuth grants |
| **Data Loss Prevention** | *What data* is being sent to those tools, and should it be allowed? | Prompt and upload content, files pasted, code, PII, secrets, classified docs |

The three named monitoring surfaces map to sensors: **user activity** (browser extension and endpoint agent), **application traffic** (network/proxy), and **API calls** (egress gateway and code or secrets scanning).

"Rogue AI" also covers **local models** (Ollama, LM Studio), **AI features embedded in sanctioned SaaS** (e.g., an AI assistant inside a note-taking app), **AI browser extensions**, and **autonomous agents / MCP servers** calling external APIs. This is the modern frontier, and a good place to differentiate.

---

## 2. What we would implement (realistic 8-day scope)

### 2.1 Architecture

```
  ┌──────────────────────── Sensors ───────────────────────────────┐
  │ (1) Network: explicit proxy (mitmproxy) + DNS logger (Zeek /    │
  │     CoreDNS / Pi-hole) → SNI, domains, JA4 TLS fingerprints,    │
  │     request sizes, SSE streaming patterns                      │
  │ (2) Endpoint agent (Python/Go): running processes, listening   │
  │     ports (11434 Ollama, 1234 LM Studio), installed AI apps &  │
  │     browser extensions, osquery-style                          │
  │ (3) Browser extension (Chrome MV3): paste / file-upload / form │
  │     submit events on AI web UIs → content to DLP before send   │
  │ (4) API/egress gateway: outbound API calls from services,      │
  │     detects AI SDK traffic & API key patterns                  │
  │ (5) Repo & SaaS scanner: secrets/API keys in Git (AI keys),    │
  │     OAuth grants to AI apps (Google Workspace / M365 audit)    │
  └───────────────────────────┬────────────────────────────────────┘
                              ▼  normalised events (OCSF/ECS JSON) via queue (Redis/Kafka-lite)
  ┌───────────────────── Analytics core ────────────────────────────┐
  │ AI Service Catalog: domains/endpoints/fingerprints → vendor,    │
  │   risk tier, sanctioned? (policy-driven, updatable)             │
  │ Detection engine: rules (Sigma-like YAML) + anomaly (baseline   │
  │   per user: new AI domain, volume spike, off-hours uploads)     │
  │ DLP classifier pipeline:                                        │
  │   regex+checksum (IBAN GR, AFM, AMKA, cards/Luhn, emails),      │
  │   secrets (gitleaks rules), NER PII (Presidio),                 │
  │   doc fingerprinting / exact-data-match (hashes of confidential │
  │   files), local LLM semantic classifier ("is this source code / │
  │   contract / customer data?")                                   │
  │ Risk scoring: user × app × data sensitivity → score             │
  │ Policy engine (OPA/Rego or YAML): ALLOW / WARN / REDACT / BLOCK │
  └───────────────────────────┬────────────────────────────────────┘
                              ▼
  ┌────────────── Response & UI ───────────────────────────────────┐
  │ Real-time block/redact in proxy & extension; user coaching pop-up│
  │ SOC dashboard: AI usage inventory, top risky users/apps, data   │
  │   types leaked, timeline, MITRE ATLAS/ATT&CK mapping            │
  │ Alerts → SIEM (Wazuh/Elastic) / webhook (Teams/Slack)           │
  │ Compliance report: AI inventory for EU AI Act / ISO 42001       │
  └─────────────────────────────────────────────────────────────────┘
```

### 2.2 Components in detail
1. **AI service catalog.** A curated YAML/DB of AI services: web UIs and API endpoints for the major vendors (OpenAI, Anthropic, Google Gemini, Mistral, DeepSeek, Perplexity, Hugging Face, Character.ai, Poe, and others), local runtimes (Ollama, LM Studio, llama.cpp server), and AI browser extensions (by extension ID). Each entry has a risk tier (data retention or training policy, jurisdiction) and a sanctioned status. Make it **updatable**, and possibly auto-classify unknown domains with an LLM ("is this an AI service?").
2. **Network sensor.** Run mitmproxy as an explicit proxy in the lab, with addon scripts in Python. Log domain, SNI, method, size, and content-type. Detect the **streaming (SSE) response pattern** typical of LLM APIs, even on unknown domains (behavioural detection). Add a DNS log for devices not using the proxy.
3. **Endpoint agent.** Check process names and hashes, listening ports, installed apps, browser extensions, and env vars containing API keys (`OPENAI_API_KEY`, `ANTHROPIC_API_KEY`). Report every N seconds.
4. **Browser extension.** Intercept paste, file upload, and submit on AI sites. Send content to the DLP service *before* it leaves, then show an inline **coaching banner**: "This text contains 3 customer IBANs. Redact and continue? / Use the company-approved assistant instead."
5. **DLP pipeline.** Layered for precision:
   - Deterministic: regex + validators (Luhn, IBAN mod-97, **Greek AFM checksum**, **AMKA**), secret patterns (AWS keys, private keys, AI API keys).
   - NER-based PII: Microsoft Presidio, with custom Greek recognisers.
   - Exact Data Match / fingerprints: hash shingles of "crown jewel" documents and code repos. This detects a pasted chunk of proprietary code even when partially edited.
   - Semantic: a **local** small LLM or classifier labels confidentiality (source code, legal, HR, financial). It must be local, otherwise the DLP itself leaks data.
6. **Policy engine.** Rules like *"Sanctioned AI + PII → redact"*, *"Unsanctioned AI + source code → block + alert"*, *"Local LLM → allow + log"*. Express them in OPA/Rego or a simple YAML DSL editable from the UI.
7. **Safe AI gateway (the "yes, and" option).** A sanctioned proxy endpoint that **auto-redacts** prompts (replacing PII with tokens), forwards them to an approved LLM, and **re-hydrates** the response. This turns security from blocker into enabler.
8. **SOC dashboard.** An AI usage inventory (apps × users × volume), a risk leaderboard, an incident timeline with evidence (redacted), and MITRE ATLAS / ATT&CK technique tags (e.g., T1567 Exfiltration Over Web Service).
9. **Integrations.** Forward alerts to Wazuh or Elastic (open source SIEMs), and send webhook notifications.

### 2.3 Lab / demo environment
- Docker Compose with a proxy, DNS, backend, DB, dashboard, a sample "employee" VM or container, plus the browser extension loaded in Chrome.
- **Scripted attack scenarios** (benign, sandboxed, no malicious code per the T&C) replayed live:
  1. An employee pastes a customer list into ChatGPT's web UI → detected, redacted, coached.
  2. A developer's script calls an AI API with a personal key → detected via egress gateway and key pattern.
  3. Someone runs Ollama locally → endpoint detection (allowed by policy, logged).
  4. **Evasion attempts:** base64-encode the data, split it across 5 prompts, use an unknown AI domain or reverse proxy, upload a screenshot instead of text. Show which ones are caught and **honestly** which aren't, with mitigations.

### 2.4 Suggested stack
- **Python** (FastAPI, mitmproxy addons, Presidio), **Go** optionally for the endpoint agent.
- **Postgres** (or OpenSearch for events), Redis streams.
- **React** dashboard. Grafana is a quick win for time-series panels.
- **Chrome MV3** extension (TypeScript).
- **OPA** for policy.
- **Wazuh/Elastic** for the SIEM integration demo.

---

## 3. What we need to study

| Area | Topics | Pointers |
|---|---|---|
| Network security monitoring | Proxies and TLS interception, SNI, DNS logging, **JA3/JA4** fingerprints, Zeek | mitmproxy docs, Zeek docs, FoxIO JA4 |
| DLP concepts | Content inspection, EDM/IDM fingerprinting, false-positive management, classification labels | Vendor whitepapers (Microsoft Purview DLP, Netskope, Zscaler) |
| PII detection | Regex + checksums, NER, Greek identifiers (AFM, AMKA, ΑΔΤ), IBAN | Microsoft Presidio, gitleaks/trufflehog rules |
| Endpoint telemetry | Process and port enumeration, osquery, browser extension management | osquery docs, Chrome MV3 docs |
| Detection engineering | Sigma rules, behavioural baselining, alert fatigue, MITRE ATT&CK & **MITRE ATLAS** | SigmaHQ, atlas.mitre.org |
| AI security | OWASP Top 10 for LLM Apps, prompt injection, agent/MCP risks | OWASP GenAI project |
| Governance & law | **EU AI Act** (AI inventory, deployer obligations), **GDPR** (employee monitoring must be proportionate; DPIA; transparency), **NIS2** (Greek transposition Law 5160/2024), **ISO/IEC 42001** (AI management system), Greek DPA (HDPA) guidance on employee monitoring | Official texts, ENISA reports |
| Shadow AI landscape | Industry stats on employee AI use and data pasted into AI tools | Reports from Cyberhaven, Netskope Threat Labs, Microsoft Work Trend Index |

---

## 4. How to differentiate

1. **Enable, don't just block.** The Safe AI Gateway with auto-redaction and re-hydration, plus in-the-moment coaching. Pitch line: *"Employees will use AI anyway. We make the safe path the easiest path."* Productivity + security is a CISO's dream story.
2. **Behavioural detection of unknown AI.** Don't rely only on a domain blocklist. Detect LLM-like traffic (SSE token streaming cadence, request/response size ratios, JA4 of AI SDKs). Show detection of a **brand-new or self-hosted AI endpoint** that isn't in any list.
3. **Red-team-aware design.** One judge leads a Red Team. Present an **evasion matrix** (see §5) that is backed by automated tests. Honesty about limits scores better than claiming 100%.
4. **Agentic and embedded AI.** Detect AI coding agents and **MCP servers** calling external APIs, AI features inside sanctioned SaaS, and AI browser extensions. Most teams will only cover chat UIs.
5. **Privacy-preserving monitoring.** Pseudonymise users by default. De-anonymisation requires a justified incident (four-eyes approval). Store only redacted snippets and hashes, not raw prompts. This makes it **GDPR/HDPA-defensible** in Greece and shows maturity.
6. **Local-only DLP intelligence.** The semantic classifier runs on-prem (a small local model). *"Our DLP doesn't leak the data it's protecting."*
7. **Governance output, not just alerts.** Auto-generate an **AI inventory and risk register** for EU AI Act and ISO 42001 audits, plus a weekly "AI posture score" for management.
8. **OT angle (for Nikolaidis).** Show one policy for an OT/engineering network segment, e.g., an engineer pasting PLC configurations or network diagrams into an AI tool. Flag industrial file types (`.L5X`, `.s7p`, `.ap*` project files) as high sensitivity.
9. **Mapped to standards.** Every detection is tagged with MITRE ATT&CK/ATLAS IDs and exported in standard formats (OCSF/ECS, Sigma), making it SOC-ready on day one.

---

## 5. Red team, evasion matrix & detection-as-code testing

### 5.1 Background: red, blue, purple
- **Red team:** security professionals who **play the attacker**. They emulate real adversaries end to end (phishing → access → escalation → lateral movement → exfiltration). Their goal is to reach an objective **without being detected**, which shows whether defences actually work. They differ from pentesters: a pentest is scoped and noisy ("find all the vulns in this app"), while a red team is goal-driven and stealthy ("can you steal X without the SOC noticing?").
- **Blue team:** the defenders: SOC analysts, detection engineers, incident responders.
- **Purple team:** red and blue working together in a loop: *run attack → did the blue team detect it? → fix the gap → re-run.*

Judge **Sokratis Ioannidis heads Netcompany's Red Team**. Expect the question *"How would I get around this?"* The evasion matrix is our prepared answer, and it is our own purple-team loop.

### 5.2 What an evasion matrix is
A table of **every realistic way to bypass our detection**. For each technique it records whether we catch it, which layer catches it, the residual risk, and the mitigation. It is not a formal standard. It is a coverage map, similar to mapping detections against MITRE ATT&CK, and every row is backed by an automated test.

**How to build it:**
1. **Threat-model the actors:** (a) a *careless employee* who just wants to get work done, (b) a *savvy employee* who knows there is monitoring and works around it, (c) a *malicious insider* deliberately exfiltrating, (d) *software*: scripts, agents, and MCP servers calling AI APIs.
2. **Enumerate techniques per surface** (network, endpoint, browser, API, content), using MITRE ATT&CK exfiltration and defense-evasion techniques as a checklist.
3. **Run each technique against the lab, record the result, and turn it into a test** (§5.4).
4. **Show the matrix in the pitch and on the dashboard**, with ✅ / ⚠️ / ❌ and the reasons.

### 5.3 The matrix (initial version, to be filled in with real results during the build)

| ID | Actor | Evasion technique | ATT&CK | Expected result | Detecting layer(s) | Residual risk / mitigation |
|---|---|---|---|---|---|---|
| EV-01 | Careless | Paste customer list (names, IBANs, AFM) into ChatGPT web UI | T1567 | ✅ Detect + redact + coach | Browser ext → DLP regex/checksum + NER | Low |
| EV-02 | Careless | Upload a confidential `.docx`/`.pdf` to an AI chat | T1567 | ✅ Detect + block | Browser ext (upload hook) → doc fingerprint (EDM) | Heavily edited docs reduce fingerprint match |
| EV-03 | Careless | Paste proprietary source code into an AI assistant | T1567 | ✅ Detect + warn | Code fingerprint (shingle hashes of repos) + local LLM classifier | Very short snippets are ambiguous |
| EV-04 | Savvy | **Base64 / hex / URL-encode** sensitive data before pasting | T1027, T1132 | ✅ Detect | DLP normalisation step decodes common encodings recursively before scanning | Custom ciphers / XOR not decoded; entropy alert as a fallback |
| EV-05 | Savvy | **Split** data across many small prompts or sessions | T1030 | ⚠️ Partial | Per-user/session aggregation window of DLP hits | A slow drip over days slips under thresholds; long-window scoring |
| EV-06 | Savvy | **Paraphrase / summarise** confidential info in own words | — | ⚠️ Partial | Local semantic classifier (topic = confidential project names, client names list) | Fundamentally hard; log + risk score, don't block |
| EV-07 | Savvy | Upload a **screenshot / photo** of the data | T1567 | ⚠️ Partial | OCR (Tesseract) on image uploads → DLP | Low-quality/handwritten images; flag "image upload to unsanctioned AI" anyway |
| EV-08 | Savvy | Put data in a **password-protected ZIP** before upload | T1560 | ⚠️ Partial | Detect encrypted archive (can't inspect) → policy: block to unsanctioned AI | Can't see content; rely on policy |
| EV-09 | Savvy | Use a **new / obscure AI site** not on the catalog | T1567 | ✅ Detect | Behavioural network detection (SSE token-streaming cadence, req/resp size pattern) + LLM auto-classifying unknown domains | Novel transport (WebSockets, custom protocols) |
| EV-10 | Savvy | Access AI via a **personal reverse proxy / self-hosted front-end** on a generic domain (e.g., a cloud VM) | T1090 | ⚠️ Partial | Behavioural streaming signature, JA4 fingerprint of AI SDKs, new-domain-with-high-upload alert | Fully custom clients mimicking normal browsing |
| EV-11 | Savvy | Use a **personal phone / home network / hotspot** | — | ❌ Out of scope | — | Corporate device policy, MDM, training; acknowledge openly |
| EV-12 | Savvy | Use an **AI browser extension** (sidebar assistant) instead of a website | T1176 | ✅ Detect | Endpoint agent enumerates installed extensions vs catalog | Sideloaded/unknown extensions; flag by permissions (read all sites) |
| EV-13 | Savvy | Run a **local LLM** (Ollama, LM Studio) | — | ✅ Detect (allowed by policy, logged) | Endpoint agent: process names/hashes, listening ports 11434/1234 | Renamed binary or custom port; hash + model-file (`.gguf`) detection |
| EV-14 | Software | Script calls an AI API with a **personal API key** | T1567 | ✅ Detect | Egress gateway (AI endpoints) + API key regex (`sk-…`, `sk-ant-…`) | Keys fetched at runtime from a vault aren't visible in code |
| EV-15 | Software | AI API key **committed to a Git repo** | T1552 | ✅ Detect | Repo scanner (gitleaks rules) | Private repos outside our org |
| EV-16 | Software | **Coding agent / MCP server** sends repository contents to an external LLM | T1567 | ⚠️ Partial | Egress gateway + DLP on API payloads, process attribution (which tool made the call) | Agents using sanctioned endpoints with sensitive context; needs per-agent policy |
| EV-17 | Malicious | **Disable or uninstall** the browser extension / endpoint agent | T1562 | ✅ Detect | Heartbeat monitoring: missing heartbeat → tamper alert | Short offline windows; correlate with network logs |
| EV-18 | Malicious | Bypass the proxy via **DNS-over-HTTPS** or direct IP | T1572 | ⚠️ Partial | Block known DoH resolvers, alert on direct-IP TLS to AI ASN ranges | IP ranges change; maintain list |
| EV-19 | Malicious | Use **AI features embedded in a sanctioned SaaS** (e.g., AI assistant inside a docs tool) | — | ⚠️ Partial | Catalog of SaaS AI-feature endpoints + browser ext on known SaaS AI UI elements | Vendors add AI features silently; periodic catalog updates |

### 5.4 Turning each row into an automated test

Each evasion is a **scenario file** plus a **benign attack script**. The test harness runs it against the docker-compose lab and asserts the expected detection. This is **detection-as-code**, the same idea as Red Canary's *Atomic Red Team* (small, scripted, repeatable attack tests).

**Repo layout**
```
tests/
  evasion/
    scenarios/
      EV-01_paste_pii_webui.yaml
      EV-04_base64_encoded_pii.yaml
      ...
    payloads/                  # synthetic data only (fake IBANs, fake AFMs, fake code)
    runners/                   # how to "perform" the action: browser (Playwright), http, endpoint
    test_evasion_matrix.py     # parametrised over all scenario files
  conftest.py                  # spins up docker compose lab, clears alert store
scripts/
  build_matrix_report.py       # runs suite → generates EVASION_MATRIX.md + JSON for dashboard
```

**Scenario file example** (`EV-04_base64_encoded_pii.yaml`)
```yaml
id: EV-04
title: Base64-encoded PII pasted into an AI web UI
actor: savvy
attack_technique: [T1027, T1132]
runner: browser                     # Playwright drives Chrome with our extension loaded
target: https://mock-chatgpt.lab    # local mock of an AI chat UI. Never a real service.
action:
  type: paste_and_submit
  payload_file: payloads/customers_pii.txt
  transform: base64
expect:
  detected: true
  layers: [browser_extension, dlp_decoder]
  min_findings: { iban: 3, afm: 3 }
  action: redact                    # policy outcome
  alert_severity: high
  max_latency_ms: 2000
```

**Test harness** (`test_evasion_matrix.py`, simplified)
```python
import pathlib, yaml, pytest
from runners import run_scenario
from lab import alerts_api

SCENARIOS = sorted(pathlib.Path(__file__).parent.glob("scenarios/*.yaml"))

@pytest.mark.parametrize("path", SCENARIOS, ids=lambda p: p.stem)
def test_evasion(path, lab):
    sc = yaml.safe_load(path.read_text())
    started = run_scenario(sc)                       # performs the benign "attack"
    alerts = alerts_api.wait_for(scenario_id=sc["id"], since=started, timeout=10)

    exp = sc["expect"]
    if not exp["detected"]:
        pytest.xfail(f"{sc['id']} documented gap: {sc.get('residual_risk')}")
    assert alerts, f"{sc['id']} not detected"
    a = alerts[0]
    assert set(exp["layers"]) <= set(a["layers"])
    assert a["policy_action"] == exp["action"]
    for kind, n in exp.get("min_findings", {}).items():
        assert a["findings"].get(kind, 0) >= n
    assert a["latency_ms"] <= exp.get("max_latency_ms", 5000)
```

**Rules for the suite**
- **Synthetic data only:** Faker-generated names, valid-checksum but fake IBAN/AFM/AMKA, and dummy code repos. **Mock AI endpoints** run locally (a tiny FastAPI app that streams SSE like a real LLM API). No real data leaves the lab, and nothing is malicious. This stays compliant with the "no malicious code" rule.
- **Known gaps are `xfail`, not deleted.** The ❌ and ⚠️ rows stay in the suite as documented, expected failures. If we later close a gap, the test "unexpectedly passes" and we update the matrix.
- **Negative tests too:** benign actions (e.g., asking an AI a generic coding question, visiting a news site) **must not** alert. This measures the **false-positive rate**.
- **Run in CI** (GitHub Actions with docker compose) on every push.
- **Generate the report:** `build_matrix_report.py` turns the results into `EVASION_MATRIX.md` and a JSON feed for a **"Detection Coverage" dashboard panel** (green/amber/red grid by technique, detection latency, precision/recall).

### 5.5 Metrics to show in the pitch
- **Coverage:** e.g., "14/19 techniques fully detected, 4 partial, 1 out of scope".
- **DLP precision / recall** on a labelled set of about 200 benign and 200 sensitive prompts.
- **False-positive rate** on benign traffic.
- **Mean detection latency** (time from action to alert).
- **Purple-team story:** "Our first run caught 8/19. Here's what we changed to reach 14." This shows iteration and engineering rigour.

### 5.6 Live demo choreography
1. Run EV-01 live: paste PII and show the coaching banner plus redaction.
2. Run EV-04 live: "a smarter user base64-encodes it". Still caught.
3. Show EV-09: an unknown AI domain caught behaviourally.
4. Show the **coverage grid**, pointing at the ❌ and ⚠️ rows: *"Here's where we can't see, and why."*
5. Invite the red-team judge: *"Which technique would you try next?"* Have 2–3 extra scenarios ready to run on demand.

---

## 6. Pitch angle (map to judging criteria)

| Criterion | Our evidence |
|---|---|
| Innovation | Safe AI gateway with redaction/re-hydration, behavioural unknown-AI detection, agent/MCP coverage, privacy-preserving monitoring |
| Technical complexity | Multi-sensor architecture, layered DLP (regex → EDM → NER → local LLM), policy engine, SIEM integration |
| Usability / ease of use | Inline coaching UX for employees, a clean SOC dashboard, policy editor |
| Quality of delivery | Reproducible lab (docker compose), **evasion matrix as automated tests** (§5.4) running in CI, precision/recall metrics for the DLP on a labelled test set |

## 7. Risks
- **TLS interception complexity:** Use an explicit proxy plus an installed CA in the demo machine only, and document it. The browser extension avoids TLS issues for web UIs.
- **False positives:** Measure and show precision/recall on a labelled dataset. Have the testing director in mind.
- **Ethics and law:** Employee surveillance concerns. Prepare the proportionality, transparency, and DPIA answer.
- **"No malicious code" rule:** Keep attack simulations clearly benign (synthetic data, local endpoints, no real exfiltration tooling).
- **Scope:** Five sensors is too many to perfect. Do **proxy + browser extension + DLP + dashboard** deeply, and the endpoint agent and repo scanner lightly.
