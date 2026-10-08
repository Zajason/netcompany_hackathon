# Track 1: AI · "Your Digital Self"

> **Official brief:** *"Develop an AI-powered digital twin that can learn from a person's digital footprint and emulate their knowledge, communication style, preferences, and decision-making patterns."*
> The full problem statement and evaluation focus areas will be revealed at kick-off (12 Nov). Likely judge lead: **Nikos Kousiadis** (AI Lead).

---

## 1. Decoding the brief

The brief names four things the twin must emulate. Each needs a different technique, so design them as separate, measurable components:

| Dimension | What it means | Core technique |
|---|---|---|
| **Knowledge** | What the person knows: facts, projects, opinions, history | RAG over the personal corpus, plus a structured memory or knowledge graph |
| **Communication style** | How they write: tone, length, vocabulary, emoji, language mix (Greek, English, Greeklish) | Stylometric profile, plus few-shot exemplar retrieval, plus optional LoRA fine-tune |
| **Preferences** | What they like or dislike: tools, food, meeting times, tech choices | Extracted preference store (typed facts with confidence and evidence) |
| **Decision-making patterns** | How they choose: risk appetite, trade-offs, recurring heuristics | Decision log mining (past choice → context → outcome), plus a "would-I" reasoning prompt grounded in examples |

Most teams will build "RAG chatbot over my WhatsApp export". To win, treat it as a **system with a measurable fidelity score**.

---

## 2. What we would implement (realistic 8-day scope)

### 2.1 Architecture

```
 ┌───────────── Ingestion ─────────────┐     ┌──────── Profile Builder ────────┐
 │ Connectors: Gmail/mbox, WhatsApp    │     │ • Stylometry extractor          │
 │ export, Telegram/Slack/Discord JSON,│ ──► │ • Preference & fact extractor   │
 │ GitHub, Google Takeout, calendar    │     │   (LLM → typed JSON + evidence) │
 │ (.ics), notes (Markdown/Notion),    │     │ • Decision-log miner            │
 │ CV/LinkedIn PDF                     │     │ • Topic/timeline clustering     │
 └──────────────┬──────────────────────┘     └──────────────┬──────────────────┘
                │ PII scrub + consent tags                  │
                ▼                                           ▼
     ┌─────────────────── Memory layer ───────────────────────────┐
     │ Vector store (pgvector/Qdrant): episodic chunks            │
     │ Fact/preference store (Postgres) with evidence & timestamp │
     │ Style exemplar index (messages grouped by recipient/context)│
     └────────────────────────────┬───────────────────────────────┘
                                  ▼
     ┌──────────────── Twin Runtime (agent) ──────────────────────┐
     │ Router → {answer-as-me, draft-reply, decide-for-me,        │
     │           explain-myself}                                  │
     │ Retrieval + style conditioning + confidence/abstention     │
     │ Tool use: calendar lookup, email draft, search own notes   │
     └────────────────────────────┬───────────────────────────────┘
                                  ▼
     ┌──────────── UI ────────────┐   ┌──────── Evaluation ─────────┐
     │ Chat w/ "why I said this"  │   │ Fidelity score dashboard    │
     │ citations, Persona card,   │   │ (style, knowledge, decision)│
     │ Memory editor (forget/fix) │   │ Blind "real vs twin" test    │
     └────────────────────────────┘   └─────────────────────────────┘
```

### 2.2 Components in detail

1. **Connectors and normalisation.** Write parsers to a unified `Message{author, recipients, channel, timestamp, text, thread_id, lang}` schema. Start with 2–3 sources that give rich *self-authored* text, such as WhatsApp `.txt` export, Gmail Takeout mbox, and Slack or Discord JSON. Add GitHub commits and PR comments for "decision" signals.
2. **Privacy layer.** Run PII detection on *third parties* in the corpus (Microsoft Presidio plus Greek regex: phone numbers, AMKA, AFM). Pseudonymise other people, and add per-source consent toggles. A "forget this" button re-indexes without that data.
3. **Profile builder** (runs offline, once per ingest):
   - **Stylometry:** average sentence length, punctuation and emoji rates, formality score, Greek/English/Greeklish ratio, greeting and sign-off patterns, and how style shifts **per audience** (boss vs friend).
   - **Fact and preference extraction:** the LLM reads chunks and outputs typed JSON (`{type: preference, subject: "IDE", value: "VS Code", confidence: .8, evidence: [msg_ids], first_seen, last_seen}`).
   - **Decision mining:** find messages where a choice was made ("let's go with…", "I'd rather…", accept or decline invites) and store context → choice → stated rationale.
4. **Twin runtime:**
   - Hybrid retrieval (BM25 + embeddings, multilingual model such as `bge-m3` or `multilingual-e5`, since Greek matters).
   - **Style conditioning:** retrieve 5–10 of the person's real messages to a *similar recipient in a similar context* as few-shot exemplars, plus a persona card in the system prompt.
   - **Confidence and abstention:** if retrieval support is weak, the twin says *"I'm not sure. Real me would need to answer this."* This is a big trust win.
   - **Modes:** *Answer as me* · *Draft my reply* (to an incoming email) · *What would I decide?* (shows similar past decisions) · *Explain my reasoning*.
5. **Optional stretch: fine-tune.** Apply a LoRA on a small open model (Qwen2.5-7B, Llama-3.1-8B, or a Greek model such as Meltemi or Krikri from ILSP) on the person's outgoing messages. Compare the fine-tuned model vs RAG + few-shot on the fidelity benchmark. That is a strong "technical complexity" story, even if RAG wins.
6. **Evaluation harness.** This is the differentiator, see §4.
7. **UI:** a chat with expandable "evidence" chips, a persona card (auto-generated "who I am"), a memory editor, and the fidelity dashboard.

### 2.3 Suggested stack
- **Backend:** Python + FastAPI. Use LangGraph or plain code for the agent (keep it simple and debuggable).
- **LLM:** a hosted API (Claude, GPT, or Gemini) for quality, plus a local option (Ollama) for a "privacy mode" story.
- **Stores:** Postgres + pgvector (one DB for facts and vectors).
- **Frontend:** Next.js or React + Tailwind + shadcn/ui.
- **Infra:** `docker compose up` one-liner, GitHub Actions running pytest and the eval suite.

### 2.4 Demo data
**Don't use real third-party private chats in the demo.** Options:
- A team member donates their *own* data and consents on stage. This is the most compelling, but scrub third parties.
- A public figure's public corpus: blog posts, tweets, interviews, GitHub. This is great for a blind test with the audience.
- A synthetic persona generated with an LLM. It is safe but less impressive. Use it only for CI tests.

---

## 3. What we need to study (pre-kick-off reading list)

| Area | Topics | Pointers |
|---|---|---|
| RAG fundamentals | chunking, hybrid search, re-ranking, multilingual embeddings | pgvector docs, `bge-m3`, Cohere/BGE rerankers |
| Agent memory | episodic vs semantic vs procedural memory, memory consolidation | MemGPT/Letta paper, mem0, Generative Agents (Park et al. 2023), "Generative agent simulations of 1,000 people" (Park et al. 2024, a digital-twin-of-people study) |
| Style and stylometry | authorship attribution features, style transfer | Classic stylometry (function-word frequencies), authorship verification models |
| Fine-tuning | LoRA/QLoRA, chat-format SFT data prep | HF PEFT, Unsloth, Axolotl |
| Greek NLP | Greek LLMs, Greeklish transliteration, tokenisation issues | Meltemi / Krikri (ILSP), Greek BERT |
| LLM evaluation | LLM-as-judge, pairwise comparison, human blind tests | G-Eval, MT-Bench style pairwise judging |
| Security | **prompt injection from ingested data** (an email saying "ignore instructions…"), data poisoning | OWASP Top 10 for LLM Apps |
| Legal and ethics | GDPR (lawful basis, third-party data in your chats, right to erasure), **EU AI Act Art. 50** transparency (users must know they're talking to an AI; applies from Aug 2026), deepfake and impersonation risks | EU AI Act text, EDPB guidance |

---

## 4. How to differentiate

1. **"Twin Fidelity Score" with a real evaluation harness.** Hold out 10–20% of the person's messages. For each held-out reply, have the twin generate a response to the same incoming message, then measure:
   - *Style:* stylometric distance, plus a classifier trained to tell "real vs twin" (lower accuracy means a better twin).
   - *Knowledge:* QA accuracy on facts the person knows, checked against the corpus.
   - *Decision:* agreement rate on held-out decisions.
   - *Live blind test at the pitch:* show the judges 5 messages and ask "real or twin?". This is memorable and directly evidences quality.
2. **Knows what it doesn't know.** Calibrated abstention plus citations ("I said this because of these 3 messages from March"). Judges from an enterprise company care about hallucination.
3. **Audience-aware style.** The twin writes differently to a professor, a teammate, and a parent, just like the real person. Show the same question answered for 3 recipients side by side.
4. **Temporal self.** Memories are time-stamped, so you can ask "*me in 2023*" vs "*me now*". Show how opinions and preferences evolved, which is a unique visual.
5. **Greek-first.** Handle Greek, English, and **Greeklish** code-switching natively. Most global tools fail at this. It's a local, judge-relatable differentiator.
6. **Privacy by design as a feature.** Run locally (Ollama mode), pseudonymise third parties, add a "forget" button, and keep a consent ledger. Pitch it as "*a twin you can trust*", and mention the EU AI Act and GDPR compliance explicitly.
7. **Enterprise angle (speaks to Netcompany's business).** Netcompany builds large systems for the public sector and enterprises, so frame one use case as **"Expert Knowledge Twin"**: capture a retiring or departing senior engineer's know-how from tickets, docs, and code reviews so new joiners can ask "how would Maria approach this?". It turns a consumer gimmick into a knowledge-retention product.
8. **Human-in-the-loop delegation.** The twin drafts replies to real incoming emails, and the human approves, edits, or rejects. Each correction is fed back as a training signal, and the fidelity score improves live during the demo.

**Avoid:** voice or video cloning without strong safeguards (deepfake optics), and claiming it "is" the person.

---

## 5. Pitch angle (map to judging criteria)

| Criterion | Our evidence |
|---|---|
| Innovation | Fidelity score, temporal self, audience-aware style, expert-knowledge-twin use case |
| Technical complexity | Multi-source ingestion, profile extraction pipeline, hybrid RAG, optional LoRA vs RAG comparison, eval harness |
| Usability / ease of use | One-click ingest of an export file, a persona card, a chat with evidence chips |
| Quality of delivery | Tests (parsers, extractors), eval suite in CI, docker compose, README, privacy doc |

## 6. Risks
- **Data:** Getting enough self-authored text quickly. Start exporting your own WhatsApp and Gmail data *now*. Google Takeout can take hours to days.
- **Crowding:** This will likely be the most popular track. Differentiation is mandatory.
- **Cost:** Fine-tuning compute. Use Colab or Kaggle GPUs, or skip it.
- **Ethics questions from judges:** Prepare answers on consent, third-party data, and misuse (impersonation).
