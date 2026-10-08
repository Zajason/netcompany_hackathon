# Track 2: Applications Development · "Smart Maritime Operations Assistant"

> **Official brief:** *"Build a smart maritime operations assistant that combines operational data and domain expertise to support better decisions across voyage planning, fuel use, port operations, compliance, and vessel performance."*
> The full problem statement and evaluation focus areas will be revealed at kick-off (12 Nov). This is the **Applications Development** stream, so expect the senior engineers (**Davelos, Saperas**) and the testing director (**Charpidi**) to weigh architecture and delivery quality heavily. **Pitching takes place at the University of Piraeus**, home turf of Greek shipping.

---

## 1. Decoding the brief

Five decision areas are named. Each maps to real stakeholders and data:

| Area | Key decisions | Who decides | Data |
|---|---|---|---|
| **Voyage planning** | Route, speed profile, ETA, weather avoidance | Master and the office (operations) | AIS, weather/ocean forecasts, port positions |
| **Fuel use** | Optimal speed (slow steaming), trim, fuel type choice and bunkering | Technical and operations superintendents | Noon reports, engine data, fuel prices |
| **Port operations** | Arrival time (Just-in-Time), berth waiting, laytime/demurrage risk | Operations, chartering | Port calls, congestion, berth schedules |
| **Compliance** | CII rating, EU ETS allowances, FuelEU Maritime, ECA/SECA fuel sulphur, MARPOL | Compliance and HSQE | Fuel consumption, distance, cargo |
| **Vessel performance** | Hull fouling, engine degradation, when to clean or maintain | Technical superintendent | Speed-power deviation over time |

"Application Development" means judges want a **real, well-engineered app**: a backend with domain services, data pipelines, and a polished UI. An LLM chat is not enough. The LLM is the *interface*. The *engines* (fuel model, compliance calculators, optimiser) must be deterministic code.

---

## 2. Domain primer (must know before kick-off)

- **AIS (Automatic Identification System):** ships broadcast position, speed, course, and heading. This gives free historical and live tracks.
- **Noon report:** a daily report from the ship covering position, distance, speed, fuel consumed (ME/AE/boiler), ROB (remaining on board), weather, and draft. It is the core operational dataset, and it is notoriously **messy and error-prone**.
- **Speed vs fuel:** fuel consumption scales roughly with the **cube of speed** (the admiralty law). Slowing down 10% saves about 27% fuel per day, about 19% per voyage. This is the most important lever.
- **CII (Carbon Intensity Indicator, IMO, since 2023):** an annual A–E rating for ships ≥5,000 GT. CO₂ / (capacity × distance). Required thresholds tighten each year. A "D" for 3 consecutive years or a single "E" requires a corrective action plan in the SEEMP.
- **EEXI:** a one-time technical efficiency requirement on design. It is less relevant for operations.
- **EU ETS for shipping (since 2024):** owners must surrender emission allowances for voyages to, from, or within the EU (100% intra-EU or in port, 50% for extra-EU legs). Phase-in: 40% of 2024 emissions, 70% of 2025, **100% from 2026**. Each tonne of CO₂ costs roughly €60–90.
- **FuelEU Maritime (since 1 Jan 2025):** limits the GHG intensity of energy used on board (well-to-wake), −2% vs baseline in 2025, tightening to −80% by 2050. There are penalties for deficits, and surplus can be pooled or banked.
- **ECA/SECA:** sulphur emission control areas with a 0.10% S fuel limit. **The Mediterranean SECA has been in force since 1 May 2025.** This is directly relevant to Greek shipping and the fuel switching decision.
- **Laytime / demurrage:** charter-party terms. If the ship waits too long at port, the charterer pays demurrage. This creates the "hurry up and wait" problem, which **Just-in-Time arrival** solves.
- **Hull fouling:** marine growth increases resistance. A speed-power curve drifting over time signals when to clean the hull or polish the propeller.

---

## 3. What we would implement (realistic 8-day scope)

### 3.1 Architecture

```
 ┌──────── Data ingestion ────────┐   ┌─────────── Domain engines (deterministic, tested) ───────────┐
 │ • AIS (historical CSV / live   │   │ Fuel model: physics baseline (admiralty) + ML residual        │
 │   websocket)                   │   │   (gradient boosting on speed, draft, trim, wind, waves)     │
 │ • Weather & sea state          │──►│ Voyage optimiser: route (sea-route graph) + speed profile     │
 │   (Open-Meteo Marine,          │   │   s.t. ETA window / JIT, min cost = fuel + ETS + charter hire │
 │   Copernicus Marine, NOAA GFS) │   │ Compliance: CII calc & projection A–E, EU ETS € liability,    │
 │ • Noon reports (synthetic but  │   │   FuelEU balance, SECA fuel switch checker                    │
 │   realistic generator + CSV    │   │ Performance: speed-loss / fouling trend, anomaly detection    │
 │   upload)                      │   │ Port: congestion & waiting-time estimate, demurrage risk      │
 │ • Port list (UN/LOCODE, WPI)   │   └───────────────────────────┬───────────────────────────────────┘
 │ • Fuel prices (manual/table)   │                               │ exposed as typed tools / REST API
 └────────────────────────────────┘                               ▼
           │ (TimescaleDB/PostGIS)       ┌──────── AI Copilot ("Ask the Superintendent") ───────────┐
           └────────────────────────────►│ LLM w/ tool calling over the engines (never computes      │
                                         │ numbers itself) + RAG over regulations (MARPOL Annex VI,  │
                                         │ CII guidelines, EU ETS/FuelEU texts, charter-party clauses)│
                                         └──────────────────────────┬────────────────────────────────┘
                                                                    ▼
          ┌──────────────────────────── Web app ──────────────────────────────────────┐
          │ Fleet map (live positions, weather layers, SECA zones)                   │
          │ Vessel page: CII gauge & projection, fuel/speed curves, fouling trend     │
          │ Voyage planner: what-if sliders (speed, route, departure) → fuel, CO₂,   │
          │   CII, ETS €, ETA, demurrage risk                                         │
          │ Alerts & recommendations feed, compliance report export (PDF)             │
          └───────────────────────────────────────────────────────────────────────────┘
```

### 3.2 Components in detail
1. **Data layer.** Use Postgres + PostGIS + TimescaleDB, or plain Postgres. Ingest:
   - **AIS:** Danish Maritime Authority free historical AIS (CSV), or live from **aisstream.io** (free websocket API). Use AIS for realistic tracks and for port congestion (count anchored ships near a port).
   - **Weather:** **Open-Meteo Marine API** (free, no key: wave height, swell, wind) and Copernicus Marine (currents).
   - **Noon reports:** build a **synthetic generator** grounded in physics: pick 3–5 vessel profiles (tanker, bulker, container), simulate voyages along real AIS tracks, and inject realistic errors (typos, missing ROB, impossible consumption) to showcase data-quality detection.
   - **Ports:** UN/LOCODE + World Port Index (NGA). Sea routing: the `searoute` Python package.
2. **Fuel model.** Baseline: `FOC = k · speed³ · displacement^(2/3)`, plus a weather correction. Residual ML (LightGBM) trained on the synthetic or real noon data. Show the model explains *why* (feature contributions).
3. **Voyage optimiser.** Given origin, destination, a laycan/ETA window, and fuel price, choose a speed profile minimising `fuel cost + ETS cost + time charter cost`. Optionally avoid bad weather segments (simple graph cost). Output 3 options: *fastest*, *cheapest*, *greenest*.
4. **Compliance engine.**
   - **CII:** attained CII per vessel, year-to-date rating, **end-of-year projection** under the current plan, and "what speed keeps us at C?".
   - **EU ETS:** € liability per voyage, using the 50% rule for extra-EU legs and the 2026 phase-in of 100%.
   - **FuelEU:** GHG-intensity compliance balance, and the penalty estimate if in deficit.
   - **SECA:** alert when a planned route enters the Med SECA and fuel isn't compliant, then plan the changeover point.
5. **Performance monitoring.** Fit the speed-power reference curve and track the speed-loss % over time. A fouling alert comes with a **cleaning ROI** ("cleaning costs €X, saves €Y/month fuel + improves CII from D→C").
6. **Port operations.** A congestion indicator from AIS anchorage counts. JIT recommendation: "Arriving at 12 kn means 36h waiting at anchorage. Reduce to 10.5 kn: arrive just in time, save 28 t fuel, €X ETS."
7. **AI Copilot.** A chat interface where the LLM *calls the engines as tools*:
   - "*Why is Aegean Star's CII trending to D?*" → pulls data, explains drivers, and suggests actions.
   - "*Plan Piraeus → Rotterdam departing Friday, arrival window Tue–Wed*" → returns optimiser options with numbers.
   - "*Does this voyage fall under EU ETS and how much will it cost?*" → calculator plus cited regulation.
   - Use RAG over regulation PDFs for compliance Q&A with citations.
8. **Role-based views:** Master (ship), Operations, Technical superintendent, and Compliance officer, each with tailored dashboards.

### 3.3 Suggested stack
- **Backend:** Python FastAPI (domain engines in pure, unit-tested modules), or Java Spring Boot if the team is strong there (Netcompany is a heavy Java/.NET shop, so that may resonate).
- **Data:** Postgres + PostGIS (+ TimescaleDB).
- **Frontend:** React/Next.js + **MapLibre GL or deck.gl** for the map, and Recharts or ECharts for charts.
- **LLM:** a hosted model with function calling.
- **Infra:** Docker Compose, GitHub Actions CI (unit tests on calculators against hand-worked examples), seed script for demo data.

---

## 4. What we need to study

| Area | Topics | Pointers |
|---|---|---|
| Shipping basics | Vessel types, charter types (voyage vs time charter), who pays fuel, laytime/demurrage, noon reports | Any "shipping 101" or Maritime Economics (Stopford) intro chapters |
| Regulations | **IMO CII** (MEPC.352–355 guidelines and reduction factors), **EU ETS maritime**, **FuelEU Maritime**, **MARPOL Annex VI**, **Med SECA (May 2025)** | IMO, EU DG CLIMA, and EMSA pages, DNV/ABS/Lloyd's Register explainers (very readable) |
| Ship performance | Admiralty coefficient, speed-power curves, weather correction, ISO 19030 (hull & propeller performance) | ISO 19030 overviews, DNV performance articles |
| Weather routing | Great-circle vs rhumb line, wave and wind penalties, isochrone method | Isochrone routing tutorials, `searoute` |
| Data | AIS message types, data cleaning, geospatial queries | aisstream.io docs, DMA AIS data, PostGIS tutorial |
| Optimisation | Speed optimisation under time windows, multi-objective trade-offs | Psaraftis & Kontovas papers on speed optimisation (Greek researchers at NTUA, cite them!) |
| LLM tool use | Function calling, structured outputs, grounding numbers in tools | Provider docs on tool use |
| Maritime cyber/OT (bonus) | IMO MSC-FAL.1/Circ.3, IACS UR E26/E27 | Relevant because judge Nikolaidis is an OT security lead |

---

## 5. How to differentiate

1. **"What-if" decision simulator, with money as the common unit.** One screen with sliders for speed, departure time, and route. It instantly shows *fuel t, CO₂, CII rating, ETS €, FuelEU €, ETA, demurrage risk*. Most teams will show a chatbot. We show a **decision cockpit**, and the copilot explains it.
2. **Numbers you can trust.** The LLM never does maths. Every number in a copilot answer links to a **calculation trace** (inputs → formula → output, with the regulation clause). Pitch line: *"An auditor can follow every euro."* This fits Netcompany's public-sector and regulated-industry DNA.
3. **Noon report data-quality guardian.** Automatically flag impossible or inconsistent reports (ROB doesn't add up, speed vs distance mismatch) before they corrupt CII and ETS reporting. This is a real, unglamorous industry pain point, and judges from delivery and testing will appreciate it.
4. **Just-in-Time arrival for Piraeus.** Combine AIS anchorage congestion with the optimiser to recommend arrival speed. Demo a live Piraeus anchorage count *at the University of Piraeus pitch*. That makes it local and tangible.
5. **Fouling → cleaning ROI.** Turn a performance trend into a business decision with a payback period. It speaks to the owner's wallet.
6. **Proactive alerts, not just Q&A.** For example, "*Vessel X will drop to CII D by November at current speed; reduce 0.8 kn on the next 3 voyages to stay C.*" This mirrors the Thessaloniki 2026 winner (risk map plus ready-made response plans).
7. **Bridge-ready, low-bandwidth mode.** A ship-side lightweight view or PWA that works offline on VSAT/Starlink and syncs when connected. It shows real-world empathy.
8. **Greek shipping relevance in the pitch.** Greek owners control about 20% of world tonnage, the largest fleet by ownership. Frame the product for a mid-size Greek management company with 20–40 vessels that can't afford enterprise suites like DNV Veracity or StormGeo.
9. **Security hook (optional).** A short slide on OT/IT segregation and data integrity of ship-to-shore feeds (IACS UR E26/E27). It earns points with the cyber judges.

---

## 6. Pitch angle (map to judging criteria)

| Criterion | Our evidence |
|---|---|
| Innovation | What-if cockpit, trustable calc traces, JIT + congestion from AIS, data-quality guardian |
| Technical complexity | Physics + ML fuel model, optimiser, geospatial pipeline, multi-regulation compliance engine, tool-calling copilot |
| Usability / ease of use | Map-first UI, role-based dashboards, plain-language recommendations |
| Quality of delivery | Calculators unit-tested against worked regulation examples, CI, seed data, docker compose, docs |

## 7. Risks
- **Domain depth:** We are not mariners. Mitigate with the study list, and try to talk to a shipping professional (e.g., University of Piraeus Maritime Studies dept.) before kick-off.
- **Data realism:** Real noon or sensor data is proprietary. Be upfront that the noon data is synthetic but physics-grounded, while AIS and weather data are real.
- **Scope creep:** Five areas is a lot. Do **compliance + fuel/speed + voyage what-if** deeply, and port and performance as lighter modules.
