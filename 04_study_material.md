# Study Material: Netcompany Hackathon Athens 2026

> Goal: bring the whole team up to speed on **all three tracks** before kick-off (**12 Nov 2026**). Netcompany makes the final track allocation, so we may not get our first choice.
> Legend: ⭐ = start here / highest value per hour · 📄 = paper · 🎥 = video · 📘 = book · 🛠️ = tool/docs · 📊 = dataset
> Research and learning before the hackathon is allowed. **Writing the actual project code before 12 Nov is not** (pre-existing code needs approval and disclosure).

---

## 0. Suggested study plan (8 Oct → 12 Nov, about 5 weeks)

| Week | Everyone | Track deep-dives (split the team) |
|---|---|---|
| **1** (8–14 Oct) | §1 Foundations: LLM basics, tool calling, RAG. Email the organisers (see overview §7). | Each person skims the "⭐ start here" items of all 3 tracks |
| **2** (15–21 Oct) | §1 Engineering: FastAPI, Docker Compose, pytest, CI | Pair A → Track 1 · Pair B → Track 2 · Pair C → Track 3 |
| **3** (22–28 Oct) | Throwaway spikes (not project code): RAG over your own notes; tool-calling demo | Continue deep-dive; collect datasets and APIs and get keys |
| **4** (29 Oct–4 Nov) | Each pair gives a **30-min teach-back** of its track to the rest. **Rank track preferences.** | Domain one-pager per track: key terms, data sources, architecture sketch |
| **5** (5–11 Nov) | Register (deadline **6 Nov**). Set up the dev environment, repo template (disclosed if used), roles. Pitch-skills material (§5). | Final reading on the likely-allocated track |

---

## 1. Foundations (all tracks)

### 1.1 LLMs and AI engineering
- ⭐🎥 **Andrej Karpathy – "Intro to Large Language Models"** (1h talk): https://www.youtube.com/watch?v=zjkBMFhNj_g
- 🎥 Andrej Karpathy – "Deep Dive into LLMs like ChatGPT" (3.5h, for the curious): https://www.youtube.com/watch?v=7xTGNNLPyMI
- ⭐📄 **Anthropic – "Building effective agents"**: when to use workflows vs agents, and simple patterns that work: https://www.anthropic.com/engineering/building-effective-agents
- 🛠️ Anthropic docs: tool use: https://docs.anthropic.com/en/docs/build-with-claude/tool-use/overview
- 🛠️ OpenAI Cookbook (many practical RAG, tool and eval notebooks): https://cookbook.openai.com/
- 📘 **Chip Huyen – *AI Engineering*** (O'Reilly, 2025): the best single book on building LLM products (evals, RAG, agents, fine-tuning, deployment).
- 🎓 Hugging Face LLM Course (free): https://huggingface.co/learn/llm-course
- 🎓 DeepLearning.AI short courses (free, 1–2h each; good ones: RAG, agents, LLM evaluation, function calling): https://www.deeplearning.ai/short-courses/
- 🎥 Jay Alammar – The Illustrated Transformer: https://jalammar.github.io/illustrated-transformer/
- 📄 ReAct: Synergizing Reasoning and Acting in Language Models (Yao et al., 2022): https://arxiv.org/abs/2210.03629
- 📄 Toolformer (Schick et al., 2023): https://arxiv.org/abs/2302.04761

### 1.2 RAG (used in all three tracks)
- ⭐📄 **Retrieval-Augmented Generation for Knowledge-Intensive NLP** (Lewis et al., 2020): the original RAG paper: https://arxiv.org/abs/2005.11401
- 📄 Lost in the Middle: How Language Models Use Long Contexts (Liu et al., 2023): https://arxiv.org/abs/2307.03172
- 📄 Self-RAG (Asai et al., 2023): retrieval plus self-critique: https://arxiv.org/abs/2310.11511
- 📄 BGE M3-Embedding (multilingual, hybrid dense + sparse; **good for Greek**): https://arxiv.org/abs/2402.03216
- 📄 RAGAS: automated evaluation of RAG (Es et al., 2023): https://arxiv.org/abs/2309.15217
- 🛠️ pgvector (vector search in Postgres): https://github.com/pgvector/pgvector
- 🛠️ Qdrant docs: https://qdrant.tech/documentation/
- 🛠️ Ollama (run open models locally): https://ollama.com/

### 1.3 Evaluation of LLM systems
- ⭐📄 **Judging LLM-as-a-Judge with MT-Bench and Chatbot Arena** (Zheng et al., 2023): https://arxiv.org/abs/2306.05685
- 📄 G-Eval: NLG evaluation using GPT-4 (Liu et al., 2023): https://arxiv.org/abs/2303.16634
- 📄 Hamel Husain – "Your AI Product Needs Evals" (practical blog): https://hamel.dev/blog/posts/evals/

### 1.4 AI security basics (needed for Track 1 privacy and core to Track 3)
- ⭐🛠️ **OWASP Top 10 for LLM Applications**: https://genai.owasp.org/llm-top-10/
- 📄 Not what you've signed up for: Indirect Prompt Injection (Greshake et al., 2023): https://arxiv.org/abs/2302.12173

### 1.5 Engineering and delivery (the "quality of delivery" criterion, and the testing-director judge)
- ⭐🛠️ **FastAPI tutorial**: https://fastapi.tiangolo.com/tutorial/
- 🛠️ Docker Compose docs: https://docs.docker.com/compose/
- 🛠️ pytest docs: https://docs.pytest.org/
- 🛠️ GitHub Actions quickstart: https://docs.github.com/en/actions/quickstart
- 🛠️ Playwright (end-to-end UI tests, also used for the Track 3 browser scenarios): https://playwright.dev/
- 🛠️ shadcn/ui (fast, good-looking React components): https://ui.shadcn.com/
- 🛠️ The Twelve-Factor App (config, logs, deployability): https://12factor.net/

### 1.6 EU regulation basics (shows up in every track)
- ⭐🛠️ **EU AI Act explorer** (readable article-by-article text): https://artificialintelligenceact.eu/
- 🛠️ GDPR full text (readable): https://gdpr-info.eu/
- 🛠️ NIST AI Risk Management Framework: https://www.nist.gov/itl/ai-risk-management-framework

---

## 2. Track 1: AI · "Your Digital Self" (digital twin of a person)

### 2.1 Start here ⭐
- 📄 **Generative Agents: Interactive Simulacra of Human Behavior** (Park et al., 2023): memory stream, reflection, planning. The architecture template for a "self": https://arxiv.org/abs/2304.03442
- 📄 **Generative Agent Simulations of 1,000 People** (Park et al., 2024): digital twins of *real* people built from 2-hour interviews, validated against their actual survey answers. This is closest to our brief, and the evaluation method is gold: https://arxiv.org/abs/2411.10109
- 📄 **MemGPT: Towards LLMs as Operating Systems** (Packer et al., 2023): long-term memory management: https://arxiv.org/abs/2310.08560

### 2.2 Memory and personalisation
- 📄 Mem0: Building Production-Ready AI Agents with Scalable Long-Term Memory (2025): https://arxiv.org/abs/2504.19413
- 🛠️ Letta (MemGPT's framework): https://docs.letta.com/
- 🛠️ mem0: https://github.com/mem0ai/mem0
- 📄 LaMP: When Large Language Models Meet Personalization (Salemi et al., 2023): a benchmark for personalised generation, including personalised email subject and tweet writing: https://arxiv.org/abs/2304.11406

### 2.3 Persona and role-play emulation
- 📄 Character-LLM: A Trainable Agent for Role-Playing (Shao et al., 2023): https://arxiv.org/abs/2310.10158
- 📄 RoleLLM: Benchmarking, Eliciting, and Enhancing Role-Playing Abilities (Wang et al., 2023): https://arxiv.org/abs/2310.00746

### 2.4 Communication style and stylometry
- 📄 Stamatatos – "A Survey of Modern Authorship Attribution Methods" (JASIST, 2009): the classic stylometry survey. Use its features (function words, character n-grams) for the "fidelity score": https://doi.org/10.1002/asi.21001
- 🛠️ `faststylometry` Python library (Burrows' Delta): https://github.com/fastdatascience/faststylometry

### 2.5 Fine-tuning (optional stretch goal)
- ⭐📄 **LoRA: Low-Rank Adaptation of LLMs** (Hu et al., 2021): https://arxiv.org/abs/2106.09685
- 📄 QLoRA (Dettmers et al., 2023): fine-tune on one consumer GPU: https://arxiv.org/abs/2305.14314
- 🛠️ Unsloth (fast LoRA, free Colab notebooks): https://github.com/unslothai/unsloth
- 🛠️ Hugging Face PEFT: https://huggingface.co/docs/peft

### 2.6 Greek language
- 📄 **Meltemi: The first open LLM for Greek** (ILSP / Athena RC, 2024): https://arxiv.org/abs/2407.20743
- 🛠️ ILSP models on Hugging Face (Meltemi, Krikri): https://huggingface.co/ilsp
- 🛠️ Greek BERT (AUEB NLP group): https://huggingface.co/nlpaueb/bert-base-greek-uncased-v1

### 2.7 Data ingestion and privacy
- 🛠️ Google Takeout (export your own Gmail, Calendar, etc. **Start now**, it can take days): https://takeout.google.com/
- 🛠️ WhatsApp chat export (per-chat `.txt`): https://faq.whatsapp.com/1180414079177245
- ⭐🛠️ **Microsoft Presidio** (PII detection and anonymisation, extensible with Greek recognisers): https://microsoft.github.io/presidio/
- 🛠️ EU AI Act **Art. 50**, transparency obligations (users must know they're interacting with AI; deepfake labelling): https://artificialintelligenceact.eu/article/50/

---

## 3. Track 2: Applications Development · "Smart Maritime Operations Assistant"

### 3.1 Start here ⭐ (domain crash course)
- ⭐🛠️ **IMO – EEXI & CII FAQ** (official explainer of the carbon intensity rating): https://www.imo.org/en/MediaCentre/HotTopics/Pages/EEXI-CII-FAQ.aspx
- ⭐🛠️ **EU Commission – Reducing emissions from the shipping sector** (EU ETS for shipping, MRV): https://climate.ec.europa.eu/eu-action/transport/reducing-emissions-shipping-sector_en
- ⭐🛠️ **EU Commission – FuelEU Maritime**: https://transport.ec.europa.eu/transport-modes/maritime/decarbonising-maritime-transport-fueleu-maritime_en
- 🛠️ IMO – Index of MEPC resolutions and guidelines for MARPOL Annex VI (incl. the CII guidelines MEPC.352–355(78), SECA): https://www.imo.org/en/OurWork/Environment/Pages/Index-of-MEPC-Resolutions-and-Guidelines-related-to-MARPOL-Annex-VI.aspx
- 🛠️ Classification society explainers are very readable and practical. Search the **DNV**, **ABS**, and **Lloyd's Register** sites for "CII explained", "EU ETS shipping", "FuelEU Maritime", "Mediterranean ECA".
- 📘 Martin Stopford – *Maritime Economics* (3rd ed.): chapters on shipping markets, costs, and charter types. Skim for vocabulary.
- 🎥 YouTube: search "how ship chartering works", "noon report explained", "CII rating explained". Plenty of 10-minute explainers.

### 3.2 Speed optimisation, fuel and weather routing (papers)
- ⭐📄 **Psaraftis & Kontovas (2013) – "Speed models for energy-efficient maritime transportation: A taxonomy and survey"**, *Transportation Research Part C* 26: 331–351. NTUA authors, so cite them in Athens! https://doi.org/10.1016/j.trc.2012.09.012
- 📄 Psaraftis & Kontovas (2014) – "Ship speed optimization: Concepts, models and combined speed-routing scenarios", *Transportation Research Part C* 44: 52–69: https://doi.org/10.1016/j.trc.2014.03.001
- 📄 Zis, Psaraftis & Ding (2020) – "Ship weather routing: A taxonomy and survey", *Ocean Engineering* 213: https://doi.org/10.1016/j.oceaneng.2020.107697
- 📄 Fagerholt, Laporte & Norstad (2010) – "Reducing fuel emissions by optimizing speed on shipping routes", *J. Operational Research Society* 61(3): https://doi.org/10.1057/jors.2009.77
- 📄 Ronen (1982) – "The effect of oil price on the optimal speed of ships", *JORS* 33: the classic cubic speed-fuel model: https://doi.org/10.1057/jors.1982.235
- 📄 **Fourth IMO GHG Study 2020** (emissions methodology, fuel and CO₂ factors): https://www.imo.org/en/OurWork/Environment/Pages/Fourth-IMO-Greenhouse-Gas-Study-2020.aspx
- 🛠️ ISO 19030 (hull and propeller performance measurement): search for overview articles. The standard itself is paywalled.

### 3.3 Data sources 📊
- ⭐📊 **aisstream.io** (free, real-time global AIS over websocket): https://aisstream.io/
- 📊 Danish Maritime Authority, historical AIS CSVs (free, large): http://aisdata.ais.dk/
- 📊 NOAA / BOEM Marine Cadastre, US AIS archive (free): https://hub.marinecadastre.gov/pages/vesseltraffic
- 📊 Global Fishing Watch API (vessel tracks, port visits): https://globalfishingwatch.org/our-apis/
- ⭐📊 **EMSA THETIS-MRV**: **real annual fuel consumption and CO₂ per ship** (all ships >5,000 GT calling at EU ports). Excellent for realistic vessel profiles and calibrating the fuel model: https://mrv.emsa.europa.eu/
- ⭐📊 **Open-Meteo Marine Weather API** (free, no key: waves, swell, currents): https://open-meteo.com/en/docs/marine-weather-api
- 📊 Copernicus Marine Service (ocean currents, waves, forecasts): https://marine.copernicus.eu/
- 📊 World Port Index (NGA): https://msi.nga.mil/Publications/WPI
- 📊 UN/LOCODE (port codes): https://unece.org/trade/cefact/unlocode-code-list-country-and-territory
- 📊 Kaggle: search "ship fuel consumption" and "vessel noon report" for small public datasets useful to prototype the ML residual model.

### 3.4 Tools 🛠️
- `searoute` (Python, realistic sea routes between ports): https://github.com/genthalili/searoute-py
- `pyais` (decode raw AIS NMEA messages): https://github.com/M0r13n/pyais
- MovingPandas (trajectory analysis): https://movingpandas.org/
- PostGIS (geospatial SQL): https://postgis.net/workshops/postgis-intro/
- MapLibre GL JS (open-source map rendering): https://maplibre.org/
- deck.gl (large-scale geo visualisation, ship tracks): https://deck.gl/
- LightGBM (fuel residual model): https://lightgbm.readthedocs.io/
- SHAP (explain model predictions): https://shap.readthedocs.io/

### 3.5 Local expertise (people to talk to)
- **University of Piraeus – Department of Maritime Studies**: the pitching venue's own domain experts. A short chat with a lecturer or PhD student about real pain points (noon-report quality, CII pressure, ETS admin) is worth more than days of reading.
- **NTUA – Laboratory for Maritime Transport** (Psaraftis' former group).
- Any family or friends in Greek shipping (superintendents, ops managers). Ask: *"What decision do you make every week that a tool could make easier?"*

### 3.6 Maritime cyber (bonus for the security judges)
- IMO – Maritime cyber risk (MSC-FAL.1/Circ.3): https://www.imo.org/en/OurWork/Security/Pages/Cyber-security.aspx
- IACS UR E26 / E27 (cyber resilience of ships and on-board systems): search the IACS website.

---

## 4. Track 3: Cybersecurity · "Shadow AI Detection & Data Protection"

### 4.1 Start here ⭐
- ⭐🛠️ **MITRE ATT&CK** – Exfiltration Over Web Service (T1567) and the Exfiltration tactic: https://attack.mitre.org/techniques/T1567/
- ⭐🛠️ **MITRE ATLAS** (adversarial threat landscape for AI systems): https://atlas.mitre.org/
- ⭐🛠️ **OWASP Top 10 for LLM Apps** (see §1.4) plus the OWASP GenAI Security Project: https://genai.owasp.org/
- ⭐📊 Industry reports on shadow AI and data leakage (use for pitch statistics):
  - IBM – Cost of a Data Breach Report 2025 (includes shadow-AI findings): https://www.ibm.com/reports/data-breach
  - Netskope Threat Labs – Cloud & Threat Reports (GenAI data policy violations): https://www.netskope.com/netskope-threat-labs
  - Cyberhaven Labs – AI adoption and data risk research: https://www.cyberhaven.com/

### 4.2 DLP concepts
- 📄 Alneyadi, Sithirasenan & Muthukkumarasamy (2016) – "A survey on data leakage prevention systems", *J. Network and Computer Applications* 62: https://doi.org/10.1016/j.jnca.2016.01.008
- 🛠️ Microsoft Purview DLP docs (good conceptual overview of policies, sensitive info types, actions): https://learn.microsoft.com/en-us/purview/dlp-learn-about-dlp
- ⭐🛠️ **Microsoft Presidio** (PII detection, custom recognisers for Greek AFM/AMKA): https://microsoft.github.io/presidio/
- 🛠️ gitleaks (secret detection rules, reuse for API-key patterns): https://github.com/gitleaks/gitleaks
- 🛠️ TruffleHog (secret scanning with verification): https://github.com/trufflesecurity/trufflehog
- 🛠️ IBAN structure and mod-97 check (ISO 13616): https://en.wikipedia.org/wiki/International_Bank_Account_Number#Validating_the_IBAN

### 4.3 Network and endpoint monitoring
- ⭐🛠️ **mitmproxy** (interception proxy, Python addons): https://docs.mitmproxy.org/stable/
- 🛠️ Zeek (network security monitoring, DNS/TLS/SNI logs): https://docs.zeek.org/
- 🛠️ JA4+ TLS/client fingerprinting (FoxIO): https://github.com/FoxIO-LLC/ja4
- 🛠️ osquery (endpoint telemetry via SQL): https://osquery.io/
- 🛠️ Chrome Extensions (Manifest V3) docs: https://developer.chrome.com/docs/extensions
- 🛠️ Server-Sent Events (how LLM APIs stream tokens, for behavioural detection): https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events

### 4.4 Detecting LLM traffic from encrypted streams (research; underpins "behavioural detection")
- ⭐📄 **What Was Your Prompt? A Remote Keylogging Attack on AI Assistants** (Weiss et al., 2024): shows token-length side channels in encrypted LLM streaming. The same signal can *identify* LLM traffic: https://arxiv.org/abs/2403.09751
- 📄 Microsoft Security – "Whisper Leak" (2025): inferring LLM conversation topics from encrypted streaming metadata. Search the Microsoft Security blog.

### 4.5 Detection engineering and red teaming
- ⭐🛠️ **SigmaHQ** (generic detection rule format, thousands of examples): https://github.com/SigmaHQ/sigma
- ⭐🛠️ **Atomic Red Team** (small scripted attack tests, the model for our evasion-matrix test suite): https://github.com/redcanaryco/atomic-red-team
- 🛠️ MITRE Caldera (automated adversary emulation): https://caldera.mitre.org/
- 🛠️ OCSF (Open Cybersecurity Schema Framework, normalised event schema): https://ocsf.io/
- 🛠️ Wazuh (open-source SIEM/XDR): https://documentation.wazuh.com/
- 🛠️ Elastic Security / Elastic Common Schema: https://www.elastic.co/guide/en/ecs/current/index.html
- 🛠️ Open Policy Agent (policy-as-code, Rego): https://www.openpolicyagent.org/docs/latest/
- 📘 *The Practice of Network Security Monitoring* (Richard Bejtlich): the classic NSM book.
- 🎥 SANS Blue Team / Purple Team webcasts on YouTube: search "purple teaming SANS" and "detection engineering".

### 4.6 AI security research (for depth and credibility)
- 📄 Extracting Training Data from Large Language Models (Carlini et al., 2021): https://arxiv.org/abs/2012.07805
- 📄 Universal and Transferable Adversarial Attacks on Aligned LMs (Zou et al., 2023): https://arxiv.org/abs/2307.15043
- 📄 Indirect prompt injection (Greshake et al., 2023): see §1.4.
- 🛠️ Model Context Protocol (MCP) spec, to understand agent and tool traffic: https://modelcontextprotocol.io/

### 4.7 Governance, law and standards (employee monitoring must be lawful)
- 🛠️ EU AI Act (deployer obligations, AI literacy Art. 4): https://artificialintelligenceact.eu/
- 🛠️ **Hellenic Data Protection Authority (HDPA)** (Greek DPA; guidance on employee monitoring): https://www.dpa.gr/
- 🛠️ EDPB guidelines (e.g., on DPIAs and processing in the employment context): https://www.edpb.europa.eu/our-work-tools/general-guidance/guidelines-recommendations-best-practices_en
- 🛠️ NIS2 Directive (Greece transposed it with Law 5160/2024): https://eur-lex.europa.eu/eli/dir/2022/2555/oj
- 🛠️ ISO/IEC 42001 (AI management system) overview: https://www.iso.org/standard/81230.html
- 🛠️ ENISA Threat Landscape reports: https://www.enisa.europa.eu/topics/cyber-threats/threat-landscape

---

## 5. Pitching and demo skills
- ⭐ Practise the **3-minute story**: problem (with a number) → live demo (the "aha" in 30s) → how it works (one architecture slide) → why it's different → what's next.
- 🛠️ Y Combinator – "How to pitch your startup" (Kevin Hale): https://www.ycombinator.com/library/6q-how-to-pitch-your-startup
- 🛠️ Devpost – hackathon resources and judging tips: https://info.devpost.com/
- Always have a **recorded backup demo video**, seed data, and an offline mode.
- Prepare answers for each judge's lens: AI quality (Kousiadis), architecture (Davelos, Saperas), security (Ioannidis, Nikolaidis), testing (Charpidi).

---

## 6. Quick "first hour" for each track (if you only have one hour)

| Track | Watch / read | Do |
|---|---|---|
| 1 Digital Self | Karpathy "Intro to LLMs" (first 30 min) + Generative Agents paper abstract and Fig. 1 | Request your Google Takeout export now |
| 2 Maritime | IMO CII FAQ + EU ETS shipping page | Open aisstream.io and watch ships near Piraeus; browse THETIS-MRV for one ship's real fuel data |
| 3 Shadow AI | MITRE T1567 + OWASP LLM Top 10 | Install mitmproxy, route your browser through it, open an AI chat, and look at the SSE stream |
