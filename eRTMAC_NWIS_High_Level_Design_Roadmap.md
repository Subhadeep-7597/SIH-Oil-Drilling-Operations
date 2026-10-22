# eRTMAC-NWIS: High-Level Design & Technical Roadmap
## Core 12-Step Architecture & Tech Stack (MCP-Enabled)

> **Problem Statement:** eRTMAC-NWIS (Nearby Wells Intelligence System)  
> **Client:** Oil India Limited (OIL)  
> **Event:** Smart India Hackathon (SIH)  
> **Scope:** Core MVP Functionality (Zero Scaling Bloat, 100% Functional Completeness)

---

## 1. End-to-End Pipeline Table (12 Core Steps)

| Step | Description (in short) | Function | Tech Stack |
| :--- | :--- | :--- | :--- |
| **01. Document Ingestion & OCR** | Clean raw scanned/digital reports and extract text/tables. | Auto-detect scan vs. digital PDF; deskew and denoise pages; run OCR; extract multi-column mud and casing tables cell-by-cell. | `PyMuPDF`, `OpenCV`, `Tesseract OCR 5` (`--oem 3`), `pdfplumber` |
| **02. Domain NLP & Quality Gate** | Extract structured entities, standardize units, and verify accuracy. | Custom petroleum NER (Well, Formation, Depth, Mud Weight, Events); normalize 70-year synonyms and units; score confidence (flag <70% for human review). | `spaCy`, `Pydantic`, Custom Synonym/Unit Rules, PostgreSQL review queue |
| **03. 3D Spatial & Knowledge Storage** | Store well records and compute true subsurface proximity in 3D. | Store directional survey trajectories; calculate 3D distance at current TVD (`ST_3DDistance`) to dynamically re-rank nearby wells; index narrative chunks in vector DB. | `PostgreSQL 16` + `PostGIS 3.4`, `sentence-transformers` (`all-MiniLM-L6-v2`), `ChromaDB` (embedded) |
| **04. Live Stream & Sensor Validation** | Stream real-time rig parameters and filter out hardware noise. | Ingest live 5s sensor feeds (ROP, WOB, RPM, Torque, SPP, Flow); filter physical bound violations, rate-of-change spikes (>300%), and sensor freeze errors. | `FastAPI WebSocket`, `TimescaleDB` (hypertables), Python sensor validation rules |
| **05. Subsurface Formation Correlation** | Correlate offset wells by geological layer rather than absolute depth. | Map active bit depth to formation tops; align lithology across wells while accounting for dip, faults, and unconformities; highlight historical hazards in the same formation. | `Pandas`, `NumPy`, `PostGIS` spatial queries |
| **06. Predictive Risk Analytics** | Forecast stuck pipe, mud loss, and kicks 15–30 minutes ahead. | Compute 80+ rolling and rate-of-change features; execute ensemble ML (XGBoost + LSTM); validate predictions against physical constraints ($ECD \ge MW$, pore/fracture gradients). | `XGBoost`, `PyTorch` (LSTM), `scikit-learn`, `imbalanced-learn` (SMOTE) |
| **07. Explainable AI (XAI)** | Break down model predictions into transparent engineering factors. | Compute SHAP values for real-time predictions; display top-3 contributing parameters (e.g., Torque +42%, ROP -58%) with magnitude and direction on every alert. | `SHAP` (`TreeExplainer`), `Joblib` |
| **08. Proactive Alerting & Throttling** | Prioritize, de-duplicate, and dispatch early hazard warnings. | Scan look-ahead depth buffer (±25m); calculate multi-factor priority score (severity × proximity × offset frequency); throttle to 5–8 alerts/shift to eliminate alert fatigue. | Python priority engine, `FastAPI WebSocket`, `Redis` Pub/Sub |
| **09. NWIS MCP Server & Grounded RAG** | Expose well memory via standardized MCP tools for audited LLM reasoning. | Host an MCP server exposing read-only tools (`get_nearby_wells`, `search_reports`, `get_mitigations`); the LLM calls these tools sequentially to synthesize answers with verified page citations. | `mcp` (Official Python SDK / `FastMCP`), `Ollama` (local Mistral/Llama-3) / `Gemini 1.5 Flash` |
| **10. Multilingual Voice Processing** | Enable hands-free querying for field crews in regional languages. | Convert spoken audio to text; translate Hindi and Assamese queries into English while preserving technical drilling terms; route to MCP server and translate answer back. | `faster-whisper` (Speech-to-Text), `IndicTrans2` (AI4Bharat), Web Audio API |
| **11. Unified Operations Dashboard** | Deliver real-time situational awareness on a single screen. | Dark-mode interface featuring an interactive Mapbox 2D/3D map, real-time parameter gauges, D3.js side-by-side lithology correlation columns, and an alert timeline. | `React 18`, `TypeScript`, `Mapbox GL JS`, `D3.js`, `Recharts`, `Tailwind CSS`, `shadcn/ui` |
| **12. Offline PWA & Audit Governance** | Ensure field continuity during network drops and log all actions. | Pre-cache field well packs and look-ahead hazards locally; queue engineer acknowledgements offline; collect feedback (*"Helpful"* / *"False Alarm"*); maintain an immutable audit trail. | `Workbox` (Service Worker), `IndexedDB`, `PostgreSQL` (immutable `audit_log`), `python-jose` (JWT) |

---

## 2. NWIS MCP Server Specification

By implementing an **MCP (Model Context Protocol) Server**, the AI reasoning engine interacts strictly through standardized, read-only tools rather than raw SQL or unconstrained prompts:

### Exposed MCP Tools Catalog

| Tool Name | Input Arguments | Data Source | Output / Purpose |
| :--- | :--- | :--- | :--- |
| `get_nearby_wells` | `lat: float, lng: float, tvd: float, radius_km: float` | `PostGIS` (`ST_3DDistance`) | Returns offset wells ranked by 3D subsurface proximity at current TVD. |
| `get_formation_events` | `well_ids: list[int], formation: str, event_type: str` | `PostgreSQL` (`drilling_events`) | Returns historical events (mud losses, stuck pipes) in that formation. |
| `search_reports` | `query: str, well_id: int | None, top_k: int` | `ChromaDB` | Returns relevant verbatim text chunks with document name and exact page numbers. |
| `get_historical_mitigations` | `formation: str, event_type: str` | `Knowledge Graph` (PostgreSQL) | Returns actions taken in offset wells (e.g., LCM pill recipes) and their recorded outcomes. |
| `predict_drilling_risk` | `well_id: int, window_minutes: int` | `TimescaleDB` + `XGBoost` | Returns risk probability, confidence score, and top-3 SHAP factors. |
| `get_current_vitals` | `well_id: int` | `TimescaleDB` | Returns latest validated real-time drilling parameters (ROP, WOB, RPM, Torque, SPP, Flow). |

---

## 3. High-Level Architectural Flow

```
[ Legacy Documents ] ──► [ Step 01: OCR / Tables ] ──► [ Step 02: NLP / Quality Gate ]
                                                                   │
                                                                   ▼
[ Live Sensor Feed ] ──► [ Step 04: Validation ]      [ Step 03: 3D Spatial & Vector DB ]
            │                                                      │
            ▼                                                      ▼
[ Step 06: Predictive ML ] ◄── [ Step 05: Correlation ] ◄──────────┘
            │
            ├───────────────► [ Step 07: SHAP XAI ] ──► [ Step 08: Alert Throttling ]
            │                                                      │
            │                                                      ▼
            │                                         [ Step 11: React Dashboard ]
            │                                                      ▲
            ▼                                                      │
[ Step 09: NWIS MCP SERVER ] ◄── [ Step 10: Voice / Hindi ] ───────┤
(get_nearby_wells, search_reports, mitigations)                    │
            │                                                      │
            ▼                                                      │
[ Step 12: Offline PWA & Audit Log ] ──────────────────────────────┘
```
