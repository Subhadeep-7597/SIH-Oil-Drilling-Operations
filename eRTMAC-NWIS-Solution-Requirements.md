# eRTMAC-NWIS — Nearby Wells Intelligence System

Solution Requirements & Product Specification · Smart India Hackathon · Problem owner: Oil India Limited

Sep 27, 2026 · @Manav

## Title Page & Document Control

**eRTMAC-NWIS (Nearby Wells Intelligence System)** is an AI-powered offset-well knowledge and decision-support platform that works alongside Oil India Limited's eRTMAC real-time monitoring system. It does not replace eRTMAC.

| Item | Detail |
| --- | --- |
| Document type | Master Solution Requirements & Product Specification (PRD + SRS + solution architecture) |
| Competition | Smart India Hackathon (SIH) |
| Problem owner | Oil India Limited (OIL) |
| Problem title | eRTMAC-NWIS: An AI-Powered Offset Well Knowledge and Decision Support Platform for Drilling Operations |
| Team | 6 students (roles defined in Section 39) |
| Status | Proposal draft for hackathon submission |
| Intended readers | SIH judges, OIL drilling engineers, geologists, managers, and the team itself |
| Source of truth | The problem statement issued by OIL. Nothing in this document describes OIL's internal systems beyond what that statement says. |

### How to read this document

- **Two-minute read:** Section 1 (Executive Summary) and Section 52 (Judges' View).
- **Operational readers:** Sections 2–6, 13–17, 29–30 and 34.
- **Technical readers:** Sections 7–12, 18–27 and 41–42.
- **Team planning:** Sections 38–40 and 51.

Wherever the solution depends on data OIL may or may not hold, the text says: *Subject to availability of corresponding OIL data/access.* Numbers marked *pilot target* or *illustrative target* are goals to be validated, not promises.

### Design principles used throughout

| # | Principle | What it means in NWIS |
| --- | --- | --- |
| 1 | Human-centred | Simplifies an engineer's work; never adds another complicated screen to learn. |
| 2 | Explainable | Every alert or recommendation shows why it appeared. |
| 3 | Traceable | Every insight links back to the original report page or data record. |
| 4 | Proactive | Relevant history surfaces before the bit reaches a risky interval. |
| 5 | Context-aware | The same event is read differently by well, depth, formation, trajectory and conditions. |
| 6 | Enterprise-ready | Security, audit, governance, scale and integration are designed in from the start. |
| 7 | Modular | Each engine (documents, search, risk, alerts) can be upgraded on its own. |
| 8 | Human-in-the-loop | AI advises; qualified drilling professionals decide. |
| 9 | Practical MVP | Everything in the hackathon scope is buildable by six students. |
| 10 | Extensible | New fields, wells, risk types, data types and models can be added later. |

## 1. Executive Summary

> **One-line value proposition:** NWIS gives every drilling engineer the combined experience of every nearby well OIL has already drilled, surfaced at the right depth, with the evidence attached.

**What NWIS is.** NWIS is a standalone, AI-assisted decision-support platform that sits beside eRTMAC. eRTMAC shows what is happening in the active well right now. NWIS answers a different question: *what happened in the wells around us when they were at this depth, in this formation, and what did the crews do about it?*

**The problem.** OIL's drilling history is rich but scattered. It lives in Well Completion Reports (WCRs), Daily Drilling Reports (DDRs), scanned PDFs, mud logging records, databases and the memories of experienced staff. Finding the right lesson takes hours of manual searching, and depends on knowing whom to ask. Useful warnings are often found after the problem has already occurred, or not at all.

**Why offset-well knowledge matters.** An *offset well* is a previously drilled well near the one being drilled. Wells in the same field often pass through the same rock layers (formations) and meet similar hazards: mud losses into weak or fractured rock, stuck pipe in swelling shale, pressure surprises, or cement jobs that fail. Knowing that three neighbouring wells lost mud at roughly the same formation top is one of the most practical pieces of information a drilling team can have.

**How NWIS works.**

1. **Reads the archive.** OCR (turning scanned pages into text) and NLP (software that understands written language) extract depths, formations, events, mud weights, casing and cementing details, and mitigation actions from historical reports.
2. **Builds a knowledge repository.** Extracted facts become structured, linked records: *Well → Formation → Depth → Event → Action → Outcome → Source page.*
3. **Maps the neighbourhood.** A map shows the active well and all offset wells within a user-chosen radius, with their history one click away.
4. **Correlates and compares.** Wells are compared by depth and by formation, and the most similar wells are ranked, with the reason shown.
5. **Watches live context.** Using eRTMAC data (subject to access), NWIS knows the current depth and parameters, and checks them against historical patterns.
6. **Warns early, with evidence.** When the well approaches an interval where neighbours had trouble, or current parameters resemble historical pre-incident behaviour, NWIS raises a graded alert, explains why, and links to the source records.

**Where AI is used, and where it is not.** OCR and NLP read documents. Semantic search finds meaning, not just keywords. Machine learning scores risk and spots unusual parameter behaviour. A large language model (LLM), run inside OIL's own environment, answers questions by calling read-only NWIS tools over the Model Context Protocol (MCP), so every answer is traceable. Self-hosted Whisper and IndicTrans2 let field staff ask by voice in Hindi or Assamese, and an offline well pack keeps essential history available when rig-site connectivity drops. Simple, well-understood rules handle depth and formation look-ups, because rules are transparent and reliable for those jobs.

**What makes it valuable.**

- It turns *data that exists* into *knowledge that is at hand when needed*.
- Every answer and alert is explainable and traceable to its source document.
- It is proactive: history surfaces before the bit reaches the risky interval.
- It preserves institutional memory as experienced staff retire or move.

**Who benefits.** Drilling engineers, wellsite supervisors, geologists, drilling managers, data teams, senior management, and future engineers who inherit a searchable record rather than a pile of reports.

**Expected impact.** NWIS is expected to cut the time spent searching historical reports from hours to minutes, help identify recurring risks earlier, support reduction of avoidable non-productive time (NPT), and make planning and training more consistent. Actual gains are to be measured in a pilot (Section 32); no specific reduction is guaranteed.

**How it complements eRTMAC.** eRTMAC remains the system for real-time monitoring. NWIS reads from it (read-only), adds historical context, and never writes to it or to any drilling control system. Final operational decisions stay with qualified drilling professionals.

## 2. Problem Understanding

The core problem is not missing data. It is that the right historical knowledge cannot be found fast enough to influence a decision on the rig.

### 2.1 Current workflow (typical, as understood from the problem statement)

1. During well planning, engineers gather offset-well information by opening WCRs, DDRs and mud logs of wells they know or are told about.
2. They note formation tops, problem intervals, mud weights and casing points, often in personal spreadsheets or slide decks.
3. During drilling, eRTMAC shows live parameters. When something unusual appears, the team relies on memory, phone calls to experienced colleagues, or a quick search through folders.
4. After the well, lessons are written into reports that join the same archive, where they are hard to find again.

### 2.2 Information silos

| Source | Typical format | Why it is hard to use |
| --- | --- | --- |
| Well Completion Reports | Long PDFs, often scanned | Key facts buried in narrative text and tables |
| Daily Drilling Reports | Daily PDFs or forms, hundreds per well | Events spread across many days; no single view |
| Mud logging records | Logs, databases, charts | Different formats and units across years |
| Drilling databases | Structured tables | Separate from the narrative explaining the numbers |
| Geological data | Formation tops, logs, maps | Held by different teams, different naming |
| Individual memory | Nowhere written | Leaves when the person retires or transfers |

### 2.3 Why PDF-heavy documentation hurts

- A scanned page is an image; a computer cannot search it until OCR converts it.
- The same event can be described in many ways ("partial losses", "lost 40 bbl", "loss of returns").
- Depths may be recorded in metres or feet, measured depth or true vertical depth.
- Tables, handwritten notes and stamps confuse simple text extraction.

### 2.4 Specific difficulties

| Difficulty | What happens today |
| --- | --- |
| Finding relevant wells | Engineers rely on memory or lists; nearby wells drilled long ago may be missed. |
| Comparing wells | Manual side-by-side reading of several reports; slow and inconsistent. |
| Linking history to current depth | No system says "you are 150 m above the interval where Well B lost circulation". |
| Detection delay | Similar past events are often discovered after the current problem starts. |
| Knowledge transfer | Lessons stay with individuals; new engineers repeat old learning. |
| Field engineers | Little time, limited connectivity, need a quick answer, not a 200-page report. |
| Office teams | Spend planning time collecting data instead of analysing it. |

### 2.5 "Data exists" vs "actionable knowledge is easily accessible"

This distinction is the heart of the problem.

| Data exists | Actionable knowledge is accessible |
| --- | --- |
| A DDR from years ago mentions losses at a depth. | The engineer sees, before reaching that depth, that 3 of 5 nearby wells lost mud in the same formation. |
| A WCR records the mud weight used. | The engineer can compare that mud weight with the current plan in one view. |
| A report describes the fix that worked. | The mitigation is shown next to the alert, linked to the page it came from. |
| An expert remembers a stuck-pipe event. | The event is recorded, searchable and available to everyone, permanently. |

NWIS exists to close the gap between the left column and the right column.

## 3. Current-State vs Proposed-State Comparison

| Aspect | Current approach | With NWIS |
| --- | --- | --- |
| Finding offset wells | From memory, lists or asking colleagues | Map shows every well within a chosen radius, with filters |
| Historical report search | Open PDFs one by one | Plain-English search across all reports, with page links |
| Event identification | Read the full narrative to find incidents | Events already extracted, tagged by type, depth and formation |
| Cross-well comparison | Manual spreadsheets | Side-by-side depth and formation comparison views |
| Risk identification | Depends on individual experience | Risk indicators from historical patterns, with evidence |
| Real-time awareness | eRTMAC shows live data, without history | Live depth and parameters matched against offset-well history |
| Institutional knowledge | In people's heads and scattered files | A permanent, searchable, growing repository |
| Decision support | Informal, slow, inconsistent | Evidence-backed alerts and recorded mitigations; engineer decides |

## 4. Solution Overview

NWIS is one pipeline: it reads OIL's historical records once, keeps them as linked knowledge, and uses that knowledge continuously to put the right history in front of the engineer at the right depth.

```text
Historical Documents + Historical Well Data + Geological Data + eRTMAC Real-Time Data
                                   |
                                   v
 1. Data Ingestion & Processing ........ collects files, tables and live feeds; records origin
                                   |
                                   v
 2. OCR + NLP + Information Extraction . pulls out depth, formation, event, action, outcome
                                   |
                                   v
 3. Structured Well Knowledge Repository  linked records with source page   <----------+
                                   |                                                  |
                                   v                                                  |
 4. Offset Well Intelligence Engine .... finds and ranks similar nearby wells, with reasons
                                   |                                                  |
                                   v                                                  |
 5. Analytics + Correlation Engine ..... aligns wells by depth and by formation       |
                                   |                                                  |
                                   v                                                  |
 6. Risk Prediction Engine ............. risk score + confidence                      |
                                   |                                                  |
                                   v                                                  |
 7. Real-Time Alert & Recommendation ... graded alerts, evidence, past mitigations    |
                                   |                                                  |
                                   v                                                  |
 8. User Dashboard ..................... map, search, comparison, alerts, evidence    |
                                   |                                                  |
                                   v                                                  |
    ENGINEER REVIEWS EVIDENCE AND DECIDES  --------- user feedback -------------------+
```

History flows down through eight stages into an alert or answer; the engineer's decision is the last step, and their feedback flows back to improve the repository.

| Stage | In simple terms | Example output |
| --- | --- | --- |
| Inputs | Historical documents, well data, geological data and (subject to access) eRTMAC real-time data | A scanned DDR; a formation-tops table; the current bit depth |
| 1. Data ingestion & processing | Gathers files and data feeds in one place, records where each came from and when | "DDR, Well B, Day 14, uploaded from archive folder X" |
| 2. OCR + NLP + extraction | OCR turns page images into text; NLP reads the text and pulls out facts | "Partial losses of 60 bbl at 2,480 m in Formation F2; mud weight reduced" |
| 3. Knowledge repository | Stores each fact as a structured record linked to its well, depth, formation and source page | Event record: Well B · 2,480 m · F2 · mud loss · moderate · LCM pumped · losses cured · DDR p.3 |
| 4. Offset well intelligence | Finds wells near the active well and ranks the most relevant ones | "Well B: 2.1 km, same field, same formation sequence, similar trajectory" |
| 5. Analytics & correlation | Lines up wells by depth and by formation so like is compared with like | "F2 top is 2,450 m in Well B and 2,510 m in the active well" |
| 6. Risk prediction | Combines offset history and live parameters into a risk indication with a confidence level | "Mud loss risk: elevated, confidence moderate" |
| 7. Alert & recommendation | Raises a graded alert, explains why, lists recorded past mitigations | "Approaching F2 in \~120 m; 3 of 5 offset wells had losses here" |
| 8. User dashboard | Shows it all on a map, timeline, comparison and alert screens | At-a-glance panel on a rig-site tablet |

*Well names, depths and formation labels in examples throughout this document are illustrative, not OIL data.*

## 5. Key Differentiating Concept: Institutional Memory

NWIS acts as OIL's institutional memory for drilling: what the organisation has learned once, it should never have to learn again the hard way.

### 5.1 From individual memory to organisational knowledge

| Instead of… | NWIS lets the user… |
| --- | --- |
| Relying on one person's recollection | Search a record of every extracted event, available to all authorised users |
| Searching folders | Search by meaning across every ingested report |
| Opening many PDFs | Read a one-line event summary, then jump to the exact source page |
| Asking experienced colleagues | Ask the system first, then discuss with colleagues using shared evidence |
| Manually comparing wells | Open a ready-made side-by-side comparison by depth or formation |

### 5.2 Questions NWIS is designed to answer

| Question | How NWIS answers |
| --- | --- |
| What happened when previous wells entered this formation? | Lists events recorded at or near that formation top in offset wells, by well |
| Which nearby wells encountered mud losses around this depth? | Filters loss events within a depth window and radius; shows them on the map |
| Were there stuck-pipe incidents in this formation? | Shows stuck-pipe events, their hole section, suspected cause and outcome |
| What mud weight was used? | Shows the recorded mud weight by depth for each offset well |
| What casing strategy was used? | Shows casing sizes and setting depths relative to formation tops |
| What mitigation was applied? | Lists recorded actions and their recorded outcomes, with sources |
| How much NPT resulted? | Shows recorded non-productive time per event, where reported |
| Which nearby wells behaved similarly? | Ranks similar wells and states why each was chosen |

### 5.3 How records become reusable knowledge

1. **Capture:** every report, old or new, is ingested.
2. **Extract:** facts are pulled out and tagged: depth, formation, event type, severity, cause, action, outcome.
3. **Link:** facts are connected (this event, in this formation, in this well, fixed by this action).
4. **Verify:** low-confidence extractions are flagged for expert review; experts can correct them.
5. **Reuse:** the verified knowledge feeds search, comparison, risk scoring and alerts.
6. **Grow:** each new well adds to the memory; user feedback improves it.

The result is a memory that does not retire, transfer or forget, and that gets more useful with every well drilled.

## 6. Target Users / Personas

NWIS serves six user groups; each gets a view shaped to the decisions they make.

| Persona | Goals | Problems today | Information needed | Key actions | Main dashboard needs |
| --- | --- | --- | --- | --- | --- |
| **Drilling Engineer** | Drill the current section safely and efficiently; anticipate problems | Slow access to offset history; history not linked to current depth | Nearby well context, historical incidents, live risk indicators, recorded mitigations | Select well, review map, search, compare, acknowledge alerts, open evidence | Active well panel, map, risk and alert view, comparison, search |
| **Drilling Supervisor / Wellsite Personnel** | Act quickly on the rig; keep the crew informed | Little time, limited connectivity, long reports | Short alerts, current-vs-historical comparison, the one relevant past event | Read at-a-glance panel, acknowledge alert, call the office with evidence | At-a-glance panel, simple alerts, large readable text, tablet layout |
| **Geologist / Geoscientist** | Predict formation tops and hazards; correlate geology across wells | Formation data and drilling events held separately | Formation tops, lithology, pressure indications, nearby well behaviour | Correlate formations, review events by formation, annotate | Formation correlation view, map with formation filters, cross-section style comparison |
| **Drilling Manager** | Oversee several wells; reduce NPT; plan better wells | No consolidated view of risk and NPT across wells | Risk summaries, NPT by cause and formation, performance comparison | Review dashboards, compare wells, export reports | Portfolio overview, NPT analytics, risk trends, reports |
| **Data / Technical Team** | Keep data complete, correct and current; keep models healthy | Many formats, OCR errors, unknown data quality | Ingestion status, extraction confidence, validation queue, model metrics | Upload documents, review extractions, fix data, monitor models | Administration and data-health screens |
| **Management / Decision Makers** | Understand operational risk and value from digital investments | Little visibility into lessons learned or repeated incidents | High-level KPIs, risk trends, recurring issues, adoption | Review summaries, set priorities | KPI summary, trend reports |

## 7. Functional Requirements

NWIS has 58 functional requirements across six capability areas; the Must Have items alone deliver the full value chain the problem statement asks for.

**Priority key:** **Must Have** = needed for the core value chain and the MVP. **Should Have** = strongly expected in a pilot. **Could Have** = valuable later. IDs are stable so design, test cases and the traceability matrix (Section 51) can refer to them.

### 7.1 Active Well Selection

| ID | Requirement | Description | Priority | Primary user | Expected outcome |
| --- | --- | --- | --- | --- | --- |
| FR-AW-01 | Select active well | Choose the active well from a searchable list of wells the user is authorised to see | Must Have | Drilling Engineer | Every view is anchored to one active well |
| FR-AW-02 | Well metadata | Show well name, field, well type, rig, spud date, planned total depth | Must Have | All | Basic identity at a glance |
| FR-AW-03 | Coordinates | Show surface location and, where available, bottom-hole location | Must Have | Engineer, Geologist | Position known for proximity search |
| FR-AW-04 | Current depth | Show current measured depth and, where available, true vertical depth, with last-update time | Must Have | Engineer, Supervisor | User knows how fresh the depth value is |
| FR-AW-05 | Current formation | Show the formation the bit is estimated to be in, from the well plan or prognosis and nearby tops | Must Have | Engineer, Geologist | Formation context without manual look-up |
| FR-AW-06 | Trajectory | Show planned and actual trajectory (vertical, deviated, horizontal) in plan and section view | Should Have | Engineer | Trajectory comparable with offset wells |
| FR-AW-07 | Operational status | Show status such as drilling, tripping, casing, cementing, circulating, where data allows | Should Have | Supervisor, Manager | Alerts can be tuned to the operation |
| FR-AW-08 | Well plan reference | Show planned casing points, mud weight window and formation prognosis, if provided | Could Have | Engineer | Plan vs history comparison |

### 7.2 Nearby Well Geospatial Map

| ID | Requirement | Description | Priority | Primary user | Expected outcome |
| --- | --- | --- | --- | --- | --- |
| FR-MAP-01 | Show active well | Active well shown with a distinct symbol at the map centre | Must Have | All | Clear reference point |
| FR-MAP-02 | Show nearby wells | All wells within the chosen radius shown with symbols by type and status | Must Have | All | Complete neighbourhood view |
| FR-MAP-03 | User-defined radius | Radius adjustable (for example 1, 2, 5, 10 km, or custom), drawn as a circle | Must Have | Engineer, Geologist | User controls how wide to look |
| FR-MAP-04 | Distance display | Show distance from the active well to each offset well | Must Have | All | Proximity visible at once |
| FR-MAP-05 | Filters | Filter by distance, field, formation reached, well type, drilling year, total depth, completion status, operational event, incident type | Must Have | All | Focus on the wells that matter |
| FR-MAP-06 | Event markers | Colour or badge wells by historical events (losses, kicks, stuck pipe) | Must Have | Engineer | Problem wells stand out |
| FR-MAP-07 | Data availability indicator | Show which data exists for each well (WCR, DDRs, logs, trajectory) | Should Have | Engineer, Data team | User knows how complete the history is |
| FR-MAP-08 | Well pop-up | Clicking a well shows a summary card: name, distance, depth, year, key events | Must Have | All | Quick triage without leaving the map |
| FR-MAP-09 | Open well history | From the pop-up, open the full Offset Well Details screen | Must Have | All | One click from map to history |
| FR-MAP-10 | Trajectory overlay | Show well paths projected on the map, not just surface points | Should Have | Engineer, Geologist | Deviated wells compared correctly |
| FR-MAP-11 | Base layers | Satellite, terrain and field-boundary layers, subject to OIL GIS availability | Could Have | Geologist | Richer spatial context |

**Filter definitions**

| Filter | Values | Why it helps |
| --- | --- | --- |
| Distance | Slider or preset radius | Nearer wells usually share more geology |
| Field | Field names | Keep to the same field or compare across fields |
| Formation | Formations penetrated | Only wells that drilled the formation of interest |
| Well type | Exploration, appraisal, development; vertical, deviated, horizontal | Compare like with like |
| Drilling year | Range | Recent wells reflect current practices |
| Depth | Total depth range | Exclude wells that never reached the interval |
| Completion status | Completed, suspended, abandoned | Context on outcome |
| Operational event / incident type | Mud loss, kick, stuck pipe, fishing, cementing issue, NPT | Find wells with a particular history |

### 7.3 Offset Well Intelligence

The offset well view should let a user understand a well's history in under a minute: a summary card first, a timeline and depth chart second, full reports only when needed.

| ID | Requirement | Description | Priority | Primary user | Expected outcome |
| --- | --- | --- | --- | --- | --- |
| FR-OW-01 | Well overview card | Name, field, coordinates, distance, type, spud and completion dates, total depth, drilling duration, total NPT | Must Have | All | Instant summary |
| FR-OW-02 | Formation tops | Formations encountered with top depths | Must Have | Engineer, Geologist | Basis for correlation |
| FR-OW-03 | Trajectory | Survey-based path, where available | Should Have | Engineer | Geometry comparison |
| FR-OW-04 | Major events list | Mud losses, kicks, stuck pipe, torque/drag issues, cementing problems, fishing, other NPT, each with depth, formation, date, severity, source | Must Have | Engineer | All problems in one list |
| FR-OW-05 | Depth-based event chart | Events drawn against depth next to formation column and casing points | Must Have | Engineer, Geologist | See where problems cluster |
| FR-OW-06 | Casing details | Casing sizes, grades if recorded, setting depths | Must Have | Engineer | Compare casing strategy |
| FR-OW-07 | Cementing details | Cement jobs, recorded problems (losses during cementing, poor bond, remedial jobs) | Should Have | Engineer | Plan cement jobs with history in mind |
| FR-OW-08 | Mud program | Mud type and weight by depth or section | Must Have | Engineer | Compare mud weights to events |
| FR-OW-09 | Lessons learned | Extracted or curated lessons with source links | Must Have | All | Reusable learning |
| FR-OW-10 | Mitigation measures | Actions recorded against each event and their recorded outcome | Must Have | Engineer, Supervisor | See what was tried and what worked |
| FR-OW-11 | Plain-language summary | Short AI-generated summary of the well's drilling history, each sentence linked to sources | Should Have | Supervisor, Manager | Understand without reading reports |

**Making history understandable without reading full reports:** each event is shown as a single line ("2,480 m · Formation F2 · Partial mud loss, 60 bbl · LCM pumped, losses cured · DDR Day 14"). Colour indicates event type, an icon indicates severity, and a "view source" link opens the original page with the passage highlighted.

### 7.4 Historical Document Search

| ID | Requirement | Description | Priority | Primary user | Expected outcome |
| --- | --- | --- | --- | --- | --- |
| FR-SR-01 | Natural-language search | Type a question in plain English; results interpret intent (event type, depth, formation, radius) | Must Have | All | No query syntax to learn |
| FR-SR-02 | Keyword search | Exact-word and phrase search, including well names and codes | Must Have | All | Precise look-ups |
| FR-SR-03 | Semantic search | Finds passages with the same meaning in different words ("loss of returns" ≈ "lost circulation") | Must Have | All | Fewer missed results |
| FR-SR-04 | Structured filters | Narrow by well, field, radius, formation, depth range, event type, date, document type | Must Have | All | Focused results |
| FR-SR-05 | Ranking | Rank by relevance, then by proximity and similarity to the active well | Should Have | Engineer | Most useful results first |
| FR-SR-06 | Source traceability | Every result shows document, well, date, page and the matching passage | Must Have | All | Results can be verified |
| FR-SR-07 | Result grouping | Group results by well or by event | Should Have | Engineer | Easier scanning |
| FR-SR-08 | Saved searches | Save and re-run searches | Could Have | Engineer, Geologist | Faster repeat work |

Example queries the search must handle:

- "Show wells with mud losses in this formation."
- "What happened in nearby wells at 3150 m?"
- "Show stuck pipe incidents within 5 km."
- "Which wells encountered overpressure?"
- "What mitigation worked?"
- "Show cementing issues in this field."

### 7.5 AI-Powered Document Understanding (Document Intelligence)

NWIS reads each report once, pulls out the facts engineers care about, and keeps a link from every fact back to the page it came from.

**Documents handled:** WCRs, DDRs, other PDF drilling reports, scanned documents, historical logs, mud reports and incident reports.

| ID | Requirement | Description | Priority | Primary user | Expected outcome |
| --- | --- | --- | --- | --- | --- |
| FR-DI-01 | Document intake | Accept PDFs, scanned images, spreadsheets and text files, singly or in bulk | Must Have | Data team | Archive can be loaded |
| FR-DI-02 | Document classification | Recognise document type (WCR, DDR, mud report, incident report) | Should Have | Data team | Right extraction rules per type |
| FR-DI-03 | OCR | Convert scanned pages to text, including tables | Must Have | Data team | Old scans become searchable |
| FR-DI-04 | Entity extraction | Extract formation, depth, mud weight, pressure, incident, equipment, casing, cementing, NPT, mitigation, operational action | Must Have | Data team | Facts become data |
| FR-DI-05 | Event extraction | Assemble entities into events with context and outcome | Must Have | Data team | Usable event records |
| FR-DI-06 | Unit and term normalisation | Convert units and synonyms to a standard form | Must Have | Data team | Consistent comparison |
| FR-DI-07 | Confidence score | Attach a confidence to each extracted item | Must Have | Data team | Weak extractions flagged |
| FR-DI-08 | Human review queue | Low-confidence items go to experts to accept, correct or reject | Should Have | Data team, Engineer | Trusted knowledge |
| FR-DI-09 | Source linking | Store document ID, page and text location for every fact | Must Have | All | Every fact verifiable |

**What each technique does**

| Technique | Plain explanation | Example |
| --- | --- | --- |
| **OCR** (Optical Character Recognition) | Turns a picture of a page into computer text | A scanned 1990s DDR becomes searchable |
| **NLP** (Natural Language Processing) | Lets software understand sentences, not just words | Understands that "returns lost while drilling at 2480m" describes a mud-loss event |
| **Named entity extraction** | Finds and labels specific items in text | Depth = 2,480 m; Formation = F2; Mud weight = 1.18 sg; Casing = 9 5/8 in |
| **Event extraction** | Groups those items into one incident with cause and response | Mud loss · 60 bbl · LCM pumped · cured |
| **Information structuring** | Stores the event in a standard format | See the record chain below |

**Entities extracted:** formation, depth (MD/TVD), mud weight and type, pressure (pore, fracture, standpipe, where recorded), incident type, equipment involved, casing size and setting depth, cementing job details, NPT hours, mitigation, and operational action.

**Standard record chain:**

> Depth → Formation → Event → Severity → Cause/Context → Action Taken → Outcome → Source

| Field | Example |
| --- | --- |
| Depth | 2,480 m MD |
| Formation | F2 (illustrative) |
| Event | Partial mud loss |
| Severity | Moderate (60 bbl lost) |
| Cause/Context | Drilling ahead with 1.20 sg mud; suspected fractured zone |
| Action taken | LCM pill pumped; mud weight reduced to 1.17 sg |
| Outcome | Losses cured after 4 hours; drilling resumed |
| Source | Well B, DDR Day 14, page 2, paragraph 3 |

**Keeping the link to the original.** Every record stores the document, page and position of the text it came from. In the interface, any fact has a "view source" action that opens the original page with the passage highlighted, so users can verify it themselves.

### 7.6 AI Tool Layer, Voice and Low-Connectivity Access

Three capabilities make NWIS distinctive in the field: an AI assistant that works only through audited MCP tools, voice questions in Indian languages, and a field view that keeps working when the connection drops.

| ID | Requirement | Description | Priority | Primary user | Expected outcome |
| --- | --- | --- | --- | --- | --- |
| FR-MCP-01 | Tool-based assistant | The assistant gets all facts by calling read-only NWIS tools over the Model Context Protocol (MCP), never from the model's own memory | Must Have | All | Answers built only from NWIS data |
| FR-MCP-02 | Tool trace | Each answer shows which tools were called, with what inputs, and what they returned | Must Have | All, Data team | Users and auditors see how an answer was built |
| FR-MCP-03 | Role-scoped tools | Each role can use only an approved list of tools; no tool can change data or operations | Must Have | Administrator | Access rules enforced for AI as for people |
| FR-VM-01 | Voice questions | Push-to-talk voice input, converted to text by a self-hosted speech model (Whisper) | Should Have | Supervisor, Field staff | Hands-busy users can ask questions |
| FR-VM-02 | Indian-language questions and answers | Ask and receive answers in Hindi or Assamese (more languages later), translated by self-hosted IndicTrans2 | Should Have | Field staff, new engineers | Language is not a barrier to history |
| FR-VM-03 | Transcript confirmation | The recognised question is shown for confirmation or correction before it runs | Must Have (when voice is on) | Voice users | Misheard questions are caught |
| FR-VM-04 | Protected technical terms | Numbers, units, depths, well and formation names are never altered by translation; English original shown alongside | Must Have (when translation is on) | All | No technical meaning is lost |
| FR-OF-01 | Offline well pack | While connected, the device stores a compact pack of the active well, its ranked offset wells, look-ahead events, mitigations and key source pages | Must Have | Field staff | Essential history readable without a connection |
| FR-OF-02 | Connection status | A clear indicator: online, degraded or offline, with last-sync time on every panel | Must Have | Field staff | Users always know how fresh the data is |
| FR-OF-03 | Offline actions queue | Acknowledgements, notes and feedback made offline are queued and synchronised on reconnect | Must Have | Field staff | No work is lost |
| FR-OF-04 | Low-bandwidth mode | Text-first pages, compressed data, cached map tiles, images on demand, only changed records synced | Should Have | Field staff | Usable over slow or costly links |

## 8. Knowledge Repository

The knowledge repository is the single, structured home for everything NWIS knows about past wells, and every item in it is linked to its source.

### 8.1 What it stores

| Category | Contents |
| --- | --- |
| Well profiles | Identity, location, type, dates, depth, trajectory, status |
| Drilling events | Every extracted operational event, typed and located by depth and formation |
| Geological observations | Lithology notes, shows, formation descriptions |
| Formation information | Formation names, tops per well, known hazards |
| Mud-related events | Mud weight changes, contamination, gains |
| Pressure-related events | Kicks, overpressure indications, well control actions |
| Stuck pipe events | Depth, hole section, suspected mechanism, freeing method, outcome |
| Lost circulation | Volumes, rates, cure method, outcome |
| Torque/drag | Abnormal torque or drag episodes, tight spots, reaming |
| Casing | Sizes, setting depths, running problems |
| Cementing | Job details, losses during cementing, remedial jobs |
| Fishing | Fish description, time, outcome |
| NPT | Duration, cause category, linked event |
| Lessons learned | Curated statements with context and source |
| Mitigation measures | Actions taken and recorded effectiveness |
| Recommendations | Practices derived from history, reviewed by experts |
| Source documents | Original files, pages, extraction metadata |

### 8.2 Well Knowledge Graph (relationship layer)

A knowledge graph is a way of storing facts as things connected by named relationships, like a map of who-is-linked-to-what. It lets NWIS answer questions that cross several categories at once.

**Core chain:** Well → Formation → Depth interval → Event → Risk type → Action → Outcome, with every link pointing to a Source document.

| Relationship | Meaning | Example |
| --- | --- | --- |
| Well *penetrates* Formation | The well drilled through this formation, top at a given depth | Well B penetrates F2 at 2,450 m |
| Event *occurred in* Well at Depth | Event location | Mud loss at 2,480 m in Well B |
| Event *occurred within* Formation | Geological context | Mud loss within F2 |
| Event *is a type of* Risk | Classification | Mud loss → Lost circulation risk |
| Event *was addressed by* Action | Response | LCM pill pumped |
| Action *led to* Outcome | Effectiveness | Losses cured in 4 hours |
| Event *caused* NPT | Cost of the problem | 4 hours NPT |
| Well *is near* Well | Distance relationship | Well B is 2.1 km from active well |
| Well *is similar to* Well | Similarity with reasons | Same formation sequence and trajectory type |
| Any fact *is supported by* Source | Traceability | DDR Day 14, page 2 |

**Why this matters to users.** A question such as "What worked when nearby wells lost mud in F2?" becomes a walk along these links: nearby wells → events in F2 of type mud loss → actions → outcomes → sources. The answer is precise and every step is checkable.

## 9. Cross-Well Correlation Engine

The correlation engine lines wells up so that like is compared with like: the same depth, the same formation, the same hole section.

### 9.1 Correlation dimensions

| Dimension | What is compared | Why it matters |
| --- | --- | --- |
| Geographic proximity | Surface and subsurface distance | Nearby wells usually share geology and pressure regime |
| Formation | Formations penetrated and their tops | Hazards tend to follow rock type, not absolute depth |
| Depth | Measured and true vertical depth | Pressure and temperature change with depth |
| Trajectory | Vertical, deviated, horizontal; inclination and azimuth | Hole cleaning, torque, drag and stuck-pipe risk depend on angle |
| Reservoir characteristics | Pressure, fluid type, permeability, where available | Influences kicks and losses |
| Geological properties | Lithology, faults, fractures, where available | Losses and instability often follow fractures and shales |
| Mud parameters | Mud type, weight, rheology | Many incidents relate to the mud window |
| Drilling parameters | ROP, WOB, RPM, torque, pump rate | Parameter patterns can precede incidents |
| Historical incidents | Event types and densities by interval | Direct evidence of problem zones |
| Casing program | Casing points relative to formations | Exposed open hole affects risk |
| Cementing strategy | Slurry and job practice, recorded problems | Informs future cement jobs |
| Well type | Exploration, appraisal, development | Different uncertainty and objectives |

*Subject to availability of corresponding OIL data/access for reservoir, geological and parameter data.*

### 9.2 Depth-based correlation

Depth-based correlation asks: **what happened in other wells at about this depth?** It uses a depth window (for example ± 50 m, user-adjustable) around the current or planned depth. It is simple, fast and useful in fields where layers are nearly flat, and for depth-driven effects such as pressure and temperature.

### 9.3 Formation-based correlation

Formation-based correlation asks: **what happened in other wells in this same rock layer, wherever it was?** Formations rise and fall across a field, so the same formation can be 100 m shallower in one well than in the next. NWIS aligns wells by formation top and expresses events as "metres below the top of F2", so a loss 30 m into F2 in Well B is matched with the active well's position 30 m into F2, even if absolute depths differ.

### 9.4 Why both are needed

| Situation | Depth-based | Formation-based |
| --- | --- | --- |
| Flat, layered field | Works well | Works well |
| Dipping or faulted layers | Can match the wrong rock | Matches the right rock |
| Formation tops uncertain in the active well | Still usable | Less reliable until the top is confirmed |
| Pressure-driven hazards | Strong signal | Useful supporting signal |
| Lithology-driven hazards (swelling shale, fractured carbonate) | Weaker | Strong signal |

NWIS shows both views and flags when they disagree, which is itself useful information for the geologist.

## 10. Similar Well Identification

NWIS ranks offset wells by how relevant they are to the active well and always shows why each well was chosen.

### 10.1 Similarity dimensions

| Dimension | How it is judged (conceptually) | Default weight |
| --- | --- | --- |
| Distance | Closer is more similar | High |
| Formation sequence | Same formations penetrated, in the same order | High |
| Geological / reservoir properties | Similar lithology, pressure regime (where available) | Medium |
| Trajectory | Same well profile and similar inclination | Medium |
| Hole section | Same hole size and casing design around the interval | Medium |
| Drilling parameters | Similar ROP, WOB, RPM, mud weight ranges | Medium |
| Historical events | Similar event pattern (useful once the active well has history) | Medium |
| Well type | Same purpose and design | Low |
| Depth | Reached the interval of interest | Required |

Weights are starting points, adjustable by administrators and refined with engineer feedback. In the MVP this is a transparent weighted score; in production, weights can be learned from which wells engineers find useful.

### 10.2 "Why was this well considered similar?"

Every ranked well shows its reasons as short tags, for example:

> **Well B — similarity 0.82 (high)** Same field · Same formation sequence · 2.4 km away · Similar trajectory (deviated, 30–35°) · Same 12¼ in hole section · Similar mud-loss history in F2

Showing reasons lets the engineer disagree: a well that is close but was drilled with a very different design can be set aside with one click, and that feedback improves future ranking.

## 11. Risk Identification & Predictive Analytics

NWIS gives early warning and risk indication based on historical similarity; it does not claim to predict incidents with certainty.

### 11.1 What every risk output contains

| Output | Meaning |
| --- | --- |
| Risk score | Low / Moderate / High / Critical, backed by a 0–100 score |
| Confidence level | How much evidence supports the score (data volume, agreement between signals) |
| Supporting historical evidence | The offset events behind the score |
| Similar wells | Which wells contributed, and why they are similar |
| Relevant formation and depth | Where the risk applies |
| Historical occurrence rate | For example, "3 of 5 offset wells that drilled F2 recorded losses" |
| Recommended checks | What to review, drawn from past practice (Section 30) |

### 11.2 Risk catalogue

| Risk | What it means | Why it matters | Historical indicators | Real-time indicators (subject to eRTMAC access) | Offset-well signals | Possible prediction approach | Alert type | Recommended user action |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| **Mud loss (lost circulation)** | Drilling fluid flows into the rock instead of returning to surface | Costs mud and time; can lead to well-control problems | Loss events by formation and depth; fractured or depleted zones | Drop in return flow, pit volume loss, pressure drop | Losses in same formation in nearby wells | Formation/depth occurrence rate + anomaly detection on flow and pit volume | Depth, formation, parameter, combined | Review mud weight vs offset losses; check LCM readiness; review past cures |
| **Stuck pipe** | Drill string cannot be moved or rotated freely | Can lead to fishing or sidetrack; major NPT source | Stuck events, tight spots, reaming, swelling-shale notes | Rising torque and drag, overpull, erratic hook load, pressure spikes | Stuck events in same section or formation | Pattern classifier on torque/drag/hook-load trends + historical rate | Parameter, formation, combined | Review hole cleaning and trip practices used in offset wells |
| **Kick** | Formation fluid enters the wellbore | Well-control safety event | Kicks, gas shows, mud weight increases | Pit gain, flow increase, gas readings, drilling break | Kicks at similar depth/formation | Occurrence rate + anomaly detection on flow and pit volume | Depth, parameter, combined (high priority) | Follow established well-control procedures; review offset mud weights |
| **Overpressure zone** | Pore pressure higher than normal | Drives kicks and mud-weight decisions | Pressure data, mud weight step-ups, gas levels | Changes in ROP trend, gas, background indicators | Overpressure reported at similar formation | Offset pressure profile + trend indicators | Depth, formation | Review offset pressure profile and mud weights used |
| **Torque spike** | Sudden rise in rotating torque | Early sign of stuck pipe, poor hole cleaning or bit issues | Torque issues recorded by interval | Torque above expected band for depth and RPM | High torque in same formation or angle | Anomaly detection against expected torque band | Parameter | Compare with offset torque behaviour at the same interval |
| **Drag anomaly** | Hook load differs from expected when tripping | Tight hole, ledges, cuttings beds | Tight spots, reaming, overpull records | Overpull or slack-off beyond expected | Tight spots at similar depth | Anomaly detection on hook load vs expected | Parameter | Review offset tripping and reaming practices |
| **Cementing issue** | Poor cement placement or losses during the job | Affects well integrity; remedial work costs time | Losses during cementing, poor bond, squeeze jobs | Losses while cementing (if streamed) | Cement problems at same casing point | Historical rate by casing point and formation | Formation, planning | Review offset slurry and job practices before the job |
| **NPT (general)** | Time not spent making progress | Main cost driver | NPT by cause and interval | Operation stalled, repeated circulation | High-NPT intervals in offset wells | Statistical profile of NPT by interval | Planning, depth | Plan contingency time and resources |
| **Formation instability** | Borehole wall collapses or swells | Causes stuck pipe, cavings, reaming | Cavings, washouts, reaming notes | Rising torque, pack-off signs, cavings reports | Instability notes in same formation | Formation occurrence rate + parameter pattern | Formation, combined | Review offset mud type and weight in that formation |
| **Other hazards (extensible)** | Shallow gas, bit balling, H₂S, others as data allows | Operation-specific | Recorded events | As available | As recorded | Added as new risk modules | As configured | As defined by experts |

### 11.3 Prediction approaches, from simplest to most advanced

| Level | Approach | Plain explanation | Where used |
| --- | --- | --- | --- |
| 1 | Historical occurrence rate | Counts how often an event happened in similar wells at this formation or depth | MVP and production; always shown |
| 2 | Rule-based proximity | "Within X m of an interval with recorded events" | MVP and production |
| 3 | Anomaly detection | Learns what normal parameters look like for this depth and flags unusual behaviour | MVP (simple), production (refined) |
| 4 | Pattern similarity | Compares the recent parameter trend with trends seen before past incidents | Production; MVP demo with simulated data |
| 5 | Supervised ML classifier | Learns from labelled history which conditions came before incidents | Production, once enough labelled events exist |

The final risk score combines these signals. When the model has little data, it says so through a lower confidence, rather than guessing.

## 12. Real-Time eRTMAC Integration

NWIS reads live context from eRTMAC in a read-only way and adds history to it; it never writes back to eRTMAC or to any rig system.

### 12.1 Potential eRTMAC inputs vs historical NWIS knowledge

The fields below are *potential* inputs. Which ones are actually available, at what frequency and through which interface must be agreed with OIL. *Subject to availability of corresponding OIL data/access.*

| Potential eRTMAC real-time inputs | Historical NWIS knowledge |
| --- | --- |
| Current bit depth and hole depth | Formation tops of offset wells |
| Rate of penetration (ROP) | Offset events by depth and formation |
| Weight on bit (WOB) | Mud weights and mud programs used |
| Rotary speed (RPM) | Casing and cementing history |
| Torque | Recorded mitigations and outcomes |
| Hook load | Historical parameter profiles (where digitised) |
| Standpipe pressure | NPT by interval |
| Pump rate / flow in | Lessons learned |
| Flow out / return flow | Similar-well rankings |
| Mud weight in/out, pit volume | Historical pre-incident patterns |
| Gas readings, other mud logging data | Source documents |

**Plain-language glossary:** ROP = drilling speed; WOB = downward force on the bit; RPM = rotation speed; torque = twisting effort; hook load = weight hanging from the rig's hook; standpipe pressure = pump pressure pushing mud down the pipe; pit volume = mud in the surface tanks (a rise can mean a kick, a fall can mean losses).

### 12.2 How live data and history are combined

1. **Current depth = X** is received from eRTMAC (or from manual entry if the feed is unavailable).
2. **Current formation is estimated** from the well prognosis, offset formation tops and, when available, geologist confirmation.
3. **Relevant offset wells are identified** using the similarity ranking (Section 10).
4. **Historical incidents near this depth and formation are checked**, including those just ahead of the bit (a look-ahead window, for example the next 100–200 m).
5. **Current drilling parameters are analysed** against the expected range for this depth and against historical pre-incident patterns.
6. **Risk indicators are calculated** per risk type, with confidence.
7. **A warning is displayed** when a threshold is crossed, graded by severity.
8. **Evidence and historical context are shown**: which wells, which events, which parameters, which documents.
9. **The engineer reviews and decides** the appropriate action. NWIS records the acknowledgement and any feedback.

### 12.3 Integration modes

| Mode | Description | When used |
| --- | --- | --- |
| Live stream (preferred) | NWIS subscribes to a read-only real-time feed | Production, if eRTMAC exposes one |
| Periodic pull | NWIS requests the latest values every few seconds or minutes | If only a request-based interface exists |
| File drop | Periodic export files (for example WITSML or CSV, if available) | Interim or pilot |
| Manual entry | User enters current depth and key parameters | Fallback; always available |
| Simulated stream | Replays historical or synthetic data | Hackathon MVP and testing |

## 13. Proactive Alert System

Alerts are few, graded and explained, so that each one is worth reading.

### 13.1 Alert types

| Type | Trigger | Example message |
| --- | --- | --- |
| **Depth-based** | Active well is within the look-ahead window of an interval where offset wells had events | "The active well is approximately 120 m above an interval where 3 nearby wells experienced mud losses." |
| **Formation-based** | Active well is entering, or predicted to enter, a formation with recorded hazards | "The active well is entering Formation F3, associated with stuck-pipe events in 2 of 4 offset wells." |
| **Parameter-based** | Live parameters deviate from the expected band or resemble a historical pre-incident pattern | "Torque over the last 30 minutes shows a rising pattern similar to that recorded before stuck pipe in Well C." |
| **Combined intelligence** | Historical and live signals agree | "Current conditions and offset-well history together indicate elevated mud-loss risk." |

### 13.2 Alert content

| Field | Example |
| --- | --- |
| Alert type | Combined intelligence |
| Severity | High |
| Current condition | Depth 2,410 m; return flow down 8% in 15 min |
| Historical evidence | Losses at 2,450–2,520 m in 3 of 5 offset wells |
| Nearby wells involved | Well B (2.1 km), Well D (3.4 km), Well E (4.8 km) |
| Depth | 2,410 m now; historical events at 2,450–2,520 m |
| Formation | Approaching F2 top (predicted 2,445 m) |
| Relevant historical event | Well B: 60 bbl loss at 2,480 m, cured with LCM |
| Confidence | Moderate: 3 consistent offset events; flow signal short |
| Recommended checks | Review mud weight vs offset loss cases; confirm LCM availability; review recorded cures |
| Source information | Well B DDR Day 14 p.2; Well D WCR p.37; Well E DDR Day 21 p.1 |

### 13.3 Severity levels

| Level | Meaning | Presentation |
| --- | --- | --- |
| Information | Useful context, no action implied | In the feed only |
| Advisory | Worth reviewing during normal work | Yellow badge |
| Warning | Elevated risk; review soon | Orange banner, sound optional |
| Critical | Strong combined evidence of a safety-relevant risk (for example kick indicators) | Red banner and sound; acknowledgement required |

### 13.4 Avoiding alert overload

- **Prioritisation:** only the highest-severity alert per risk type is shown on the main view; others are grouped.
- **De-duplication:** one alert per interval and risk, updated rather than repeated.
- **Cool-down:** after acknowledgement, the same alert is not raised again unless severity increases.
- **Context filters:** parameter alerts are suppressed during operations where they do not apply (for example torque while not rotating), where status data allows.
- **Thresholds:** administrators can tune thresholds by field and risk type.
- **Acknowledgement:** users acknowledge with an optional note ("Reviewed; mud weight already adjusted").
- **Alert history:** every alert, its evidence and acknowledgement are kept for review and audit.
- **Alert explanation:** a "Why?" button on every alert (Section 14).
- **False-alert control:** feedback ("not relevant") and periodic review of alert precision tune the thresholds (Sections 35 and 37).

## 14. Explainable AI

Every NWIS alert and recommendation answers the question **"Why am I seeing this?"** in plain language, with evidence the user can open.

### 14.1 Example explanation

> **Why am I seeing this alert?**
>
> - Current depth (2,410 m) is approaching an interval where problems were recorded (2,450–2,520 m).
> - 3 of the 5 most similar nearby wells had mud losses in this interval.
> - 2 of those wells lost mud within 40 m of the F2 formation top.
> - Return flow has dropped 8% in the last 15 minutes, similar to the pattern recorded before losses in Well B.
> - Relevant reports: Well B DDR Day 14, Well D WCR p.37. *\[Open\]*

### 14.2 Key concepts in plain terms

| Concept | Plain meaning | How NWIS applies it |
| --- | --- | --- |
| Explainability | The system can say why it produced a result | Every alert lists the reasons, in order of importance |
| Evidence-based recommendation | Advice rests on recorded facts, not opinion | Recommendations quote past actions from named wells |
| Confidence score | How sure the system is, given the evidence | Shown as Low / Moderate / High with the reason ("only 1 offset well") |
| Source traceability | Every fact can be followed back to its origin | One click to the original report page |
| Signal contribution | Which inputs mattered most | "History: strong · Live flow: moderate · Torque: not involved" |
| Human verification | A person checks before acting | Acknowledge, agree or disagree; feedback recorded |

### 14.3 Explainability rules

1. No alert without at least one piece of listed evidence.
2. No generated sentence without a linked source; if there is no source, the sentence is not shown.
3. Confidence is always shown next to the risk level.
4. When data is thin, the system says so plainly ("Limited history: only 1 offset well drilled this formation").
5. Machine-learning outputs are always shown alongside the historical facts, never alone.

## 15. Dashboard / UI Requirements

NWIS has 13 screens; the first five cover most daily use, and every screen is reachable from the active well in two clicks or fewer.

| # | Screen | Purpose | Main widgets | Data shown | User actions | UX considerations |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | **Login / Access Control** | Secure entry and role-based view | Single sign-on button, role indicator | User name, role, assigned fields | Sign in, choose default well | Use OIL's existing sign-in; no separate passwords |
| 2 | **Main Operations Dashboard** | Starting point after login | At-a-glance panel, active alerts, well list, mini-map | Active wells, highest alerts, recent activity | Pick well, open alert, search | Role-tailored: supervisor sees the panel, manager sees portfolio |
| 3 | **Active Well Overview** | Everything about the current well | Header card, depth/formation strip, trajectory, parameter snapshot, plan vs actual | Metadata, depth, formation, status, casing points | Change well, open map, open comparison | Depth and last-update time always visible |
| 4 | **Nearby Wells Map** | See the neighbourhood | Map, radius control, filter panel, well pop-ups, legend, ranked list | Wells, distances, statuses, event badges, data availability | Set radius, filter, click well, toggle trajectories | Map and ranked list stay in sync; readable symbols on tablets |
| 5 | **Offset Well Details** | Understand one well quickly | Overview card, event list, depth chart with formation column, casing/mud tables, lessons, AI summary | All offset data for the well | Open source, compare with active, add note, give feedback | Summary first, detail on demand |
| 6 | **Historical Event Timeline** | See how a well's drilling unfolded | Time or depth axis with event markers, NPT bars | Events by day or depth, NPT, operations | Zoom, filter by event type, open DDR for a day | Switch between time and depth views |
| 7 | **Natural Language Knowledge Search** | Ask questions, find passages | Search box, suggested questions, filters, results with snippets, AI answer panel | Answers, matching passages, sources, confidence | Ask, refine, filter, open source, save, give feedback | Example questions shown; "no evidence found" stated clearly |
| 8 | **Cross-Well Comparison** | Compare active well with offsets | Side-by-side depth tracks (formations, events, mud weight, casing), depth/formation alignment toggle | Aligned data from 2–6 wells | Choose wells, align by depth or formation, export | Consistent scales; highlight current depth line |
| 9 | **Risk & Alert Dashboard** | Manage current and past alerts | Risk gauges by type, alert feed, alert detail with "Why?", history | Risk scores, confidence, evidence, acknowledgements | Acknowledge, comment, mark useful/not useful, escalate | Severity colours plus icons (not colour alone) |
| 10 | **Real-Time Drilling Monitoring** | Live parameters with history overlaid | Parameter tracks vs depth, expected band from offsets, anomaly markers | ROP, WOB, RPM, torque, hook load, SPP, flow, pit volume (as available) | Pick parameters, overlay offset well, zoom | Complements, not duplicates, eRTMAC; shows only what adds context |
| 11 | **Document Viewer / Evidence** | Verify facts in the original | Page viewer with highlighted passage, extracted fields side panel | Original page, extracted record, confidence | Navigate pages, confirm/correct extraction (if permitted) | Opens straight to the right page and passage |
| 12 | **Analytics & Reports** | Trends and exports | NPT charts, risk-by-formation heat map, report builder | NPT by cause/formation, incident frequencies, alert statistics | Filter, generate report, export PDF/Excel | Clear chart titles stating the finding |
| 13 | **Administration / Data Health** | Keep data and models healthy | Ingestion queue, extraction review queue, data completeness per well, model metrics, user management | Document counts, error rates, confidence distribution, model performance | Upload, review, approve, retrain request, manage roles | Only for data team and admins |

## 16. "At a Glance" User Experience

A field engineer should understand the situation within about five seconds: where the well is, what lies ahead, why it matters, and what to check.

```text
+----------------------------------------------------------------------------+
| ACTIVE WELL                                                                |
| Well NW-07 (illustrative)                                                  |
| Depth 2,410 m MD | Formation F1, F2 top expected ~2,445 m | 12-1/4 in      |
+-------------------------------------+--------------------------------------+
| NEARBY INTELLIGENCE                 | CURRENT RISK                         |
| Nearby wells within 5 km: 9         | HIGH - Mud loss                      |
| Relevant (ranked similar): 5        | Moderate: stuck pipe | Low: kick     |
| Risk events in next 200 m: 7        | Confidence: moderate                 |
+-------------------------------------+--------------------------------------+
| WHY THIS ALERT?                                                            |
| - Approaching the F2 interval where 3 of 5 similar wells lost mud          |
| - Return flow down 8% in 15 min, like the pattern before losses in Well B  |
+----------------------------------------------------------------------------+
| HISTORICAL EVIDENCE                                                        |
| Well B | 2.1 km | 2,480 m | F2 | 60 bbl loss, cured with LCM | DDR Day 14  |
| Well D | 3.4 km | 2,505 m | F2 | total losses, MW reduced    | WCR p.37    |
| Well E | 4.8 km | 2,470 m | F2 | seepage losses, drilled ahead | DDR Day 21|
+----------------------------------------------------------------------------+
| RECOMMENDED CHECKS                                                         |
| - Review current mud weight against the offset loss cases                  |
| - Confirm LCM stock and readiness, as recorded in Wells B and D            |
| - Open the recorded cures and assess whether they apply here               |
+----------------------------------------------------------------------------+
| [Acknowledge]  [Why?]  [Open evidence]              The engineer decides   |
+----------------------------------------------------------------------------+
```

The panel reads top to bottom as four questions: where am I, how risky is it, why, and what did others do. The engineer acts on it; NWIS does not.

**Design rules for the panel**

- One risk level dominates; secondary risks are smaller.
- No more than three lines per zone; everything else is one tap away.
- Colour is always paired with a word ("HIGH") and an icon, for colour-blind users and bright outdoor light.
- Large type and touch targets for tablets and gloves.
- Same layout on every well, so users learn it once.
- Last-update time is shown; stale data is greyed and labelled.

## 17. Natural Language AI Assistant

The assistant lets any user ask questions in plain English and get short answers built only from NWIS records, each with its sources.

### 17.1 Example questions

- "Show me wells within 5 km."
- "Which wells had stuck pipe?"
- "What happened near this depth?"
- "Give me the drilling history of Well X."
- "What formations caused the most NPT?"
- "Show all wells with severe mud losses."
- "Why am I receiving this alert?"
- "Which offset wells are most similar to the current well?"
- "What mitigation measures were used previously?"

### 17.2 What every answer contains

| Element | Example |
| --- | --- |
| Clear answer | "Two of the nine wells within 5 km recorded stuck pipe." |
| Relevant wells | Well C, Well F |
| Depth / formation | 2,980 m in F3; 3,040 m in F3 |
| Supporting evidence | Short quotes from the records |
| Source links | Well C DDR Day 22 p.1; Well F WCR p.51 |
| Confidence | High: both events explicitly recorded |
| Date / context | 2014 and 2019; both in 8½ in hole section |

### 17.3 How it avoids making things up

A large language model on its own can produce fluent but false statements ("hallucinations"). NWIS uses a method called **retrieval-augmented generation (RAG)**: the system first retrieves matching records from the repository, and the language model may only summarise what was retrieved.

- Every sentence in an answer must be backed by a retrieved record; unsupported sentences are removed.
- If nothing relevant is found, the assistant answers: *"I could not find evidence for this in the NWIS repository."*
- Numbers (depths, volumes, NPT hours) are taken from structured records, not generated by the model.
- The assistant cannot change data, trigger actions or contact the rig.
- Questions and answers are logged for audit and quality review.

### 17.4 Voice and Indian-language access

Field staff can ask the assistant a question by voice, in English, Hindi or Assamese, and get the answer in the same language, with every model running inside OIL's network.

1. **Speak:** the user presses and holds a talk button; a headset is recommended at the rig.
2. **Transcribe:** a self-hosted **Whisper** speech-to-text model converts the audio into text.
3. **Confirm:** the transcript is shown; the user confirms or edits it before anything runs.
4. **Translate in:** if the question is not in English, self-hosted **IndicTrans2** translates it into English.
5. **Answer:** the assistant answers through its MCP tools (Section 18.3), with sources.
6. **Translate out:** IndicTrans2 translates the answer back, with the English original one tap away.

| Design point | Approach |
| --- | --- |
| Protected terms | Numbers, units, depths, well names, formation names and abbreviations (LCM, NPT, ROP) are masked before translation and restored afterwards, so they cannot be altered |
| Drilling glossary | An OIL-reviewed glossary keeps terms such as "mud loss" and "stuck pipe" translated consistently, or left in English where field staff prefer it |
| Rig noise | Push-to-talk, noise-robust model settings, and mandatory transcript confirmation |
| Assamese accuracy | Whisper's Assamese support is weaker than its Hindi support; AI4Bharat's Indian-language speech models are the planned alternative if tests show a gap |
| Privacy | Audio is processed on OIL servers (or the rig-site edge node) and is not stored unless the user opts in for quality improvement |
| Hardware | A small Whisper model runs on a CPU for the demo; production uses a shared GPU for faster response |

Why it matters: many experienced field staff think and speak most comfortably in Hindi or Assamese. Letting them question the full drilling archive in their own language widens who benefits from institutional memory. It is also a visible fit with national priorities for Indian-language digital services.

## 18. AI / ML Architecture

NWIS uses four kinds of intelligence, each where it is most reliable: rules for clear facts, statistics for rates, machine learning for patterns, and generative AI only for reading and summarising language.

### 18.1 The four kinds of intelligence

| Kind | Plain explanation | Where NWIS uses it | Why this choice |
| --- | --- | --- | --- |
| **Rule-based logic** | Fixed, human-written if-then rules | Depth window checks, formation look-ahead, unit conversion, alert de-duplication, safety gating | Fully transparent, predictable, easy to audit |
| **Statistical analytics** | Counting, averages, rates, ranges | Historical occurrence rates, NPT statistics, expected parameter bands | Simple, explainable, works with modest data |
| **Traditional machine learning** | Models that learn patterns from labelled or unlabelled historical data | Anomaly detection on parameters, risk classifiers, similarity weighting, extraction models | Finds patterns people cannot spot by eye; measurable accuracy |
| **Generative AI (LLM)** | Language models that read and write text | Understanding difficult report text, answering plain-English questions over retrieved evidence, drafting summaries | Best at language; always grounded in retrieved records, never used to compute numbers or risk scores |

### 18.2 Components

| Component | What it does, simply | MVP | Production |
| --- | --- | --- | --- |
| Data ingestion | Collects files and feeds, records origin | Folder upload + simulated stream | Connectors to OIL repositories and eRTMAC |
| Document processing | Splits documents into pages, text blocks and tables | Digital and scanned PDFs | Plus layout models for complex forms |
| OCR | Reads scanned pages into text | Open-source OCR engine | Tuned OCR with domain vocabulary and quality scoring |
| NLP | Understands sentences | Rules + pre-trained language model | Domain-adapted models trained on OIL-labelled text |
| Information extraction | Finds entities and events | Patterns + model, drilling vocabulary | Fine-tuned extractor + LLM-assisted extraction with human review |
| Data normalisation | Standardises units, names, depths | Conversion tables, synonym lists | Managed reference data with data stewards |
| Embeddings / semantic representation | Turns text into numerical "meaning fingerprints" so similar meanings sit close together | Open-source embedding model | Domain-tuned embeddings |
| Search | Finds records by keyword and meaning | Hybrid keyword + semantic search | Plus learned ranking from feedback |
| Knowledge graph / relationship layer | Links wells, formations, events, actions, sources | Relationship tables in the main database | Dedicated graph store if query needs justify it |
| ML models | Learn from history | Occurrence rates, simple classifier | Model family per risk, retrained on schedule |
| Anomaly detection | Flags unusual live behaviour for this depth | Expected-band and isolation-style detector on simulated data | Per-field models on real eRTMAC history |
| Risk prediction | Combines signals into a score with confidence | Weighted combination of rate + proximity + anomaly | Calibrated ensemble per risk type |
| Recommendation engine | Retrieves past mitigations for similar situations | Retrieval of recorded actions and outcomes | Ranked by recorded effectiveness and feedback |
| Explainable AI | Shows reasons and evidence | Reason lists, feature contributions | Plus standard contribution methods (e.g. SHAP-style explanations) |
| LLM / conversational layer | Answers questions from retrieved evidence | Small open-weight model running locally | Larger open-weight model hosted inside OIL infrastructure |

### 18.3 MCP Tool Layer

The NWIS assistant reaches data only through a fixed set of read-only tools exposed over the **Model Context Protocol (MCP)**, so every AI answer is a logged, checkable sequence of tool calls.

**What MCP is, simply:** an open standard that lets an AI model call outside tools and data sources in a uniform way. The model decides which tool to call; the tool, not the model, fetches the facts.

| MCP server | Example tools | Backed by |
| --- | --- | --- |
| Well knowledge | Find events by type, formation and depth; get well summary; get recorded mitigations | Knowledge repository |
| Geospatial | Wells within a radius; distance between two wells | Geospatial database |
| Similarity | Rank offset wells for the active well, with reasons | Similarity engine |
| Search | Hybrid keyword and semantic search over documents | Search index |
| Risk | Current risk scores; explain an alert | Risk and alert engine |
| Documents | Fetch a source page and highlighted passage | Document store |
| Live context | Current depth and latest parameters (simulated in the MVP) | eRTMAC read-only feed |

**How a question is answered**

1. The user asks: "Which nearby wells had mud losses in F2?"
2. The model plans tool calls: wells within 5 km → find mud-loss events in F2 for those wells → fetch sources.
3. Each tool checks the user's role, runs, and returns structured results.
4. The model writes a short answer using only those results, citing each source.
5. The answer and its tool trace are shown to the user and written to the audit log.

**Why MCP for NWIS**

| Benefit | Explanation |
| --- | --- |
| Grounded answers | Numbers come from tool results, not from model memory |
| Built-in audit trail | Every call records user, tool, inputs, outputs and time |
| Same access rules for AI and people | Tools enforce role and field permissions |
| Modular | A new engine (e.g. geomechanics) becomes a new MCP server without changing the assistant |
| Future-ready | Any MCP-compatible AI assistant OIL adopts later can use NWIS knowledge without custom integration |

**Safety rules:** tools are read-only; each role has an allowlist; text inside documents is treated as data, never as instructions to the model; tool servers require authentication (OAuth via Keycloak for network transport); tool-call rate limits apply.

**MVP vs production:** in the MVP, the MCP servers run on the demo machine with a local open-weight model that supports tool calling, and the tool trace is shown under each answer. In production, the servers run as separate services behind OIL's identity system, with the same tools.

## 19. Proposed Technology Stack

The stack favours open-source, widely supported tools that a student team can use in a hackathon and that an enterprise can run on its own servers.

| Layer | Recommended (MVP) | Why | Production / alternative |
| --- | --- | --- | --- |
| Frontend | React with a component library | Large ecosystem, fast dashboard building | Same; Angular is an alternative if OIL standards require it |
| Charts | Apache ECharts or Plotly | Depth tracks and time series with little effort | Same |
| Backend | Python with FastAPI | Same language as the AI/ML work; quick to build APIs | Same, split into services as load grows |
| Database | PostgreSQL | Reliable, free, enterprise-proven | PostgreSQL cluster, or OIL's approved RDBMS |
| Geospatial | PostGIS + Leaflet map | Distance and radius queries; simple web maps | PostGIS + OpenLayers or OIL GIS services |
| Time-series | PostgreSQL (TimescaleDB extension) | Stores parameter streams in the same database | TimescaleDB or OIL's historian |
| Document processing | PyMuPDF / pdfplumber for PDFs | Extracts text and tables from digital PDFs | Plus layout-aware document models |
| OCR | Tesseract or PaddleOCR | Open-source, runs offline | Tuned PaddleOCR or an approved enterprise OCR |
| NLP | spaCy with custom drilling patterns, plus a pre-trained transformer model | Fast to set up, rules and models together | Fine-tuned transformer NER on OIL-labelled data |
| Embeddings | Open-source sentence-embedding model (e.g. BGE or E5 family) | Good quality, runs locally | Domain-tuned embeddings |
| Vector database | pgvector inside PostgreSQL | One database to run in the MVP | Qdrant or Milvus if volumes demand |
| Knowledge graph | Relationship tables in PostgreSQL | Keeps MVP simple | Neo4j if multi-hop queries grow |
| LLM | Small open-weight instruction model (e.g. Llama, Mistral or Qwen family) served locally via Ollama | No data leaves the machine; free | Larger open-weight model served with vLLM on OIL GPUs |
| ML | scikit-learn, XGBoost/LightGBM | Explainable, proven, fast on tabular data | Same, with MLflow model registry |
| Streaming | Replay service pushing simulated eRTMAC data over WebSockets | Demonstrates live behaviour without real access | Apache Kafka (or OIL's messaging layer) with read-only eRTMAC connector |
| Authentication | Keycloak (OpenID Connect) with roles | Free, standard single sign-on | Keycloak federated with OIL Active Directory/LDAP |
| Deployment | Docker Compose on one machine | One command to start the demo | Kubernetes or VMs on OIL on-premise / private cloud |
| Monitoring | Application logs + simple health page | Enough for a demo | Prometheus + Grafana; MLflow for model metrics |

**Components added for NWIS's distinctive capabilities**

| Layer | Recommended (MVP) | Why | Production / alternative |
| --- | --- | --- | --- |
| AI tool layer | Official MCP Python SDK; each NWIS engine exposed as a read-only MCP server | Standard, auditable way for the assistant to reach NWIS data | Same, over authenticated network transport (OAuth via Keycloak) |
| Tool-calling LLM | Open-weight instruction model with reliable tool use (e.g. Qwen or Llama family) via Ollama | Runs locally; calls MCP tools | Larger model served with vLLM on OIL GPUs |
| Speech-to-text | Whisper (small model, e.g. via faster-whisper), self-hosted | Multilingual, runs offline, no audio leaves the network | Larger Whisper on GPU; AI4Bharat Indian-language speech models if Assamese accuracy needs it |
| Translation | IndicTrans2 (AI4Bharat), self-hosted | Covers all 22 scheduled Indian languages, including Hindi and Assamese | Same, adapted with an OIL drilling glossary |
| Offline / low connectivity | Progressive Web App: service worker (e.g. Workbox) + IndexedDB on-device store | Installable on tablets; works from cache when offline | Plus optional rig-site edge node (Section 47.2) and device management |

**Hackathon MVP stack in one line:** React (as an offline-capable PWA) + FastAPI + PostgreSQL/PostGIS/pgvector + Tesseract/PaddleOCR + spaCy + local open-weight LLM behind MCP tool servers + Whisper + IndicTrans2 + scikit-learn, packaged with Docker Compose.

**Production stack in one line:** the same core, plus Kafka streaming, a dedicated vector/graph store if needed, GPU-hosted LLM inside OIL's network, Keycloak federated with OIL identity, Kubernetes, and full monitoring.

## 20. System Architecture

NWIS is built as eleven layers, each replaceable on its own, with security, audit and monitoring running across all of them.

```text
 1  Data Sources ............ WCRs, DDRs, PDFs, databases, geology, eRTMAC feed      | ACROSS ALL LAYERS
         |                                                                            |
 2  Integration Layer ....... secure connectors, batch import, read-only stream     | - SSO and roles
         |                                                                            | - Encryption
 3  Data Ingestion .......... intake queue, file storage, source metadata, lineage  | - Audit logs
         |                                                                            | - Data lineage
 4  Document Intelligence ... OCR, table reading, NLP entity and event extraction   | - Model registry
         |                                                                            | - Monitoring
 5  Data Processing ......... normalisation, validation, review queue               | - Backup and DR
         |                                                                            | - Feedback loop
 6  Data Storage ............ relational + geospatial DB, document store, time series| - On-premise hosting
         |                                                                            |
 7  Knowledge Layer ......... knowledge graph links, vector index (semantic search) |
         |                                                                            |
 8  Analytics / ML .......... correlation, similarity, anomaly detection, risk      |
         |                                                                            |
 9  Alert Engine ............ rules + model scores -> graded, de-duplicated alerts   |
         |                                                                            |
10  API / Service Layer ..... APIs + read-only MCP tool servers for UI and assistant |
         |                                                                            |
11  User Interface .......... offline-capable web app; voice in Hindi, Assamese     |
```

Data flows down from sources to the user interface; the knowledge layer (highlighted) is the heart that search, analytics, alerts and the assistant all draw on.

| Layer | Role | Talks to |
| --- | --- | --- |
| 1. Data sources | Historical documents, databases, geological data, eRTMAC | Integration layer |
| 2. Integration | Secure connectors; batch file import; read-only real-time subscription | Ingestion |
| 3. Data ingestion | Queues incoming items, stores originals, records lineage | Document intelligence, processing |
| 4. Document intelligence | OCR, text and table extraction, NLP entity and event extraction | Processing |
| 5. Data processing | Normalises units and names, validates, routes low-confidence items to review | Storage |
| 6. Data storage | Relational/geospatial store, document store, time-series store | Knowledge layer |
| 7. Knowledge layer | Relationship graph and semantic index | Analytics, search, assistant |
| 8. Analytics / ML | Correlation, similarity, anomaly detection, risk scoring | Alert engine, APIs |
| 9. Alert engine | Turns scores and rules into graded, explained, de-duplicated alerts | APIs, notification |
| 10. API / service layer | Secure interfaces for the UI, reports and assistant | User interface |
| 11. User interface | Web dashboard for office and field | Users |

## 21. Data Architecture

NWIS organises its data into 13 domains, joined by a common well identifier and depth reference.

| Domain | Key contents | Type | Typical source |
| --- | --- | --- | --- |
| Well master data | Well ID, name, field, type, dates, status, total depth | Structured | Well databases, WCRs |
| Well location data | Surface and bottom-hole coordinates, coordinate reference system | Structured, geospatial | GIS, WCRs |
| Trajectory data | Survey stations: depth, inclination, azimuth | Structured | Survey records |
| Geological data | Formation tops, lithology, pressure data, geological notes | Structured + text | Geology databases, WCRs |
| Drilling parameter data | ROP, WOB, RPM, torque, hook load, pressures, flow | Time/depth series | eRTMAC, historical logs |
| Mud data | Mud type, weight, properties by depth/day | Structured | Mud reports, DDRs |
| Incident/event data | Event type, depth, formation, severity, cause, action, outcome, NPT | Structured (extracted) | DDRs, incident reports |
| Casing data | Size, grade, setting depth, running notes | Structured | Casing records, WCRs |
| Cementing data | Job details, volumes, problems, remedial work | Structured + text | Cementing records |
| Document metadata | File, type, well, date, pages, OCR quality, lineage | Structured | Ingestion layer |
| Lessons learned | Statement, context, applicability, source, reviewer | Text + structured | Reports, expert input |
| Risk data | Risk type, score, confidence, contributing signals, model version | Structured | Analytics layer |
| Alert data | Alert, severity, evidence, acknowledgement, feedback | Structured | Alert engine |

**Structured vs unstructured.** Structured data already sits in rows and columns (a survey table). Unstructured data is free text or images (a DDR narrative, a scanned page). NWIS keeps the original unstructured file, extracts structured facts from it, and links the two, so that numbers can be analysed and the original can be read.

**Historical documents and streaming data together.** Historical knowledge is loaded in batches and changes slowly. Live data arrives continuously. Both are keyed by well and depth, so the risk engine can place the live bit depth against historical events on the same scale. Live data is stored in a time-series store; only summarised or event-relevant slices are kept long-term, to control volume.

## 22. Data Processing Pipeline

Every document passes through the same 13 steps, and each step records what it did so that any fact can be traced back.

| # | Step | What happens | Quality check |
| --- | --- | --- | --- |
| 1 | Document uploaded / received | Stored unchanged with a unique ID, date and origin | Duplicate check by file fingerprint |
| 2 | OCR if scanned | Image pages converted to text | OCR confidence per page; poor pages flagged |
| 3 | Text extraction | Text and tables pulled from digital pages | Empty or garbled pages flagged |
| 4 | Cleaning & normalisation | Headers, footers and noise removed; units and abbreviations standardised | Unit conversion logged |
| 5 | NLP extraction | Sentences analysed for meaning | Language model confidence |
| 6 | Entity extraction | Depths, formations, mud weights, equipment, casing found | Range checks (e.g. mud weight plausible) |
| 7 | Event extraction | Entities grouped into events with cause, action, outcome | Required fields present? |
| 8 | Depth extraction | Depth tied to each event; MD/TVD and units resolved | Depth within well's total depth? |
| 9 | Formation identification | Formation assigned from text or from formation tops at that depth | Text and tops agree? |
| 10 | Relation mapping | Event linked to well, formation, action, outcome, source | Orphan records flagged |
| 11 | Validation | Rules and confidence thresholds; low-confidence items to human review | Reviewer accepts, corrects or rejects |
| 12 | Storage | Records saved with lineage and version | Audit trail |
| 13 | Searchable knowledge | Indexed for keyword, semantic and graph queries | Sample search tests |

Data quality issues are caught as early as possible, marked with confidence rather than silently dropped, and routed to people when the system is unsure (see Section 23).

## 23. Data Quality & Validation

Historical drilling data will be imperfect; NWIS is designed to show how reliable each fact is rather than pretend every fact is equally certain.

### 23.1 Common issues and how they are handled

| Issue | Example | Handling |
| --- | --- | --- |
| Missing data | No mud weight recorded for a section | Shown as "not recorded"; never filled in by guesswork; completeness score per well |
| Duplicate data | Same event in a DDR and the WCR | Matched by well, depth, date and type; merged with both sources kept |
| Conflicting values | DDR says 2,480 m, WCR says 2,485 m | Both kept; source priority rule chooses the displayed value; conflict flag shown |
| Different units | Feet vs metres; ppg vs sg; bbl vs m³ | Converted to a standard unit; original value and unit retained |
| Different naming | "Fm-2", "F2", "Formation Two" | Synonym list managed by geologists |
| OCR errors | "2480" read as "Z480" | OCR confidence; pattern checks; low-confidence to review |
| Incorrect extraction | Planned depth captured as actual event depth | Context rules; human review sample; user "report error" button |
| Terminology changes over time | Older reports use different phrasing | Semantic search and synonym lists bridge old and new terms |
| Date inconsistencies | Day-first vs month-first formats | Date parsing checked against spud and completion dates |
| Depth inconsistencies | MD vs TVD mixed; depth beyond total depth | Depth-type detection; range check against well total depth |

### 23.2 Controls

- **Validation rules:** plausible ranges (mud weight, depth), required fields per event, date and depth consistency.
- **Confidence scores:** every extracted fact carries a confidence; the interface shows it and alerts weigh it.
- **Human review:** items below a threshold go to a review queue; reviewers accept, correct or reject; reviewed facts are marked "verified".
- **Source prioritisation:** a configurable order, for example verified database record > WCR > DDR > other report, agreed with OIL.
- **Data lineage:** every fact records its document, page, extraction method, model version, reviewer and time.

## 24. Security, Privacy & Enterprise Considerations

Drilling data is commercially and operationally sensitive, so NWIS is designed to run entirely inside OIL's own environment, with no dependence on public AI services.

| Area | Approach |
| --- | --- |
| Authentication | Single sign-on through OIL's identity system (e.g. Active Directory via OpenID Connect); multi-factor where OIL policy requires |
| Role-based access control | Roles such as Field User, Drilling Engineer, Geologist, Manager, Data Steward, Administrator |
| Authorisation | Access limited by role and by field/asset; sensitive wells can be restricted further |
| Data encryption | Encrypted in transit (TLS) and at rest (database and file storage) |
| Secure APIs | Authenticated, authorised, rate-limited; input validation; no anonymous access |
| Audit logs | Record of logins, searches, document views, alert acknowledgements, data edits, model changes |
| Data lineage | Every fact traceable to its source and processing steps |
| Access logging | Who viewed which well or document, and when |
| Network security | Deployed within OIL's internal network segments; read-only, one-way link from eRTMAC where possible |
| Secure document storage | Originals stored unchanged, access-controlled, with checksums |
| Backup | Scheduled backups of databases, documents and models |
| Disaster recovery | Documented restore procedure; recovery targets agreed with OIL IT |
| Model security | Models stored in a registry with versions; only approved models deployed |
| AI prompt/data security | Assistant limited to authorised data for that user; guards against instructions hidden in documents; prompts and answers logged |
| Protection of operational information | No data sent outside OIL's environment; exports watermarked and logged |

### 24.1 Why not public AI services

Sending well data, reports or questions to an external AI service would move sensitive operational and reservoir information outside OIL's control, where it may be stored, logged or processed under another organisation's terms. It could also conflict with public-sector data-handling rules.

**Enterprise-safe approaches:**

1. Run open-weight language models on OIL's own servers or approved private cloud.
2. Keep embeddings, vector indexes and documents inside the same environment.
3. If an external service is ever considered, use only approved, contractually bound providers with data-residency guarantees, and only for non-sensitive tasks.

### 24.2 Security for MCP tools, voice and offline caches

| Area | Control |
| --- | --- |
| MCP tools | Read-only tools only; per-role allowlists; authenticated tool servers; every call logged; rate limits |
| Prompt injection | Text retrieved from documents is passed to the model as data and cannot add tools or change permissions |
| Voice data | Processed on OIL servers or the edge node; not retained by default; opt-in retention for quality work only |
| Translation | Self-hosted; no text sent to public translation services |
| Offline caches | Encrypted on the device; limited to the user's authorised wells; expire after a set period; remotely wiped through device management |

## 25. Reliability & Availability

NWIS should be dependable during drilling, but because it is a decision-support tool, not a control system, a brief outage must never affect rig safety.

| Aspect | Expectation |
| --- | --- |
| High availability | Redundant application instances and database replicas in production; *proposed target: 99.5% availability during operations, to be agreed with OIL* |
| Fault tolerance | Failure of one engine (e.g. the assistant) does not stop the map, search or alerts |
| Backup | Daily database backups; document store replicated |
| Recovery | Tested restore procedures; *proposed target: service restored within a few hours* |
| Monitoring | Health checks on every service, data feed and model |
| Logging | Central logs for troubleshooting and audit |
| Alert delivery reliability | Alerts stored before display; unacknowledged critical alerts re-displayed; feed loss itself shown as a visible status |
| Field connectivity | Cached essential knowledge on field devices; clear "last updated" labels (Section 47) |

**Stale-data rule:** if the live feed stops, NWIS shows "live data unavailable since HH:MM" and stops parameter-based alerts, instead of silently showing old values.

## 26. Scalability

NWIS is designed to start in one field and grow to all of OIL's operational areas without redesign.

| Stage | Scope | What changes |
| --- | --- | --- |
| Pilot | One field, tens of wells, one team | Single server; manual document loading |
| Multi-field | Several fields, hundreds of wells | Automated ingestion; per-field models and thresholds |
| Organisation-wide | All operational areas, all active rigs | Clustered services, streaming platform, central model governance |

| Dimension | How it scales |
| --- | --- |
| Wells | Spatial indexing keeps radius searches fast as well count grows |
| Documents | Processing runs as parallel background jobs; new documents do not slow the UI |
| Users | Stateless application services can be multiplied behind a load balancer |
| Data volume | Time-series compression; long-term storage keeps summaries and event windows |
| Real-time streams | Streaming platform handles many rigs in parallel, one channel per well |
| ML models | Model registry tracks per-field and per-risk models |
| Fields and areas | Field is a first-class attribute; access and thresholds set per field |

**Adding new data types and risks.** Each risk is a module with its own definition, indicators, thresholds and explanation template. A new risk (for example shallow gas) is added by defining its event type, extraction patterns, indicators and alert rules, without changing the rest of the system. New data types join through the ingestion layer with a mapping to the common well/depth reference.

## 27. Integration Strategy

NWIS connects to existing OIL systems through standard, secure interfaces, and reads from them rather than replacing them. All integrations are subject to availability of corresponding OIL data/access and must be defined jointly with OIL.

| System | Integration approach | Direction |
| --- | --- | --- |
| eRTMAC | Read-only event stream or periodic API pull; file export as interim | Into NWIS |
| Existing drilling databases | Scheduled batch data pipelines | Into NWIS |
| GIS systems | Standard map services for base layers and well locations | Into NWIS |
| Reservoir / geological databases | Batch pipelines for formation tops, pressure data | Into NWIS |
| Document repositories | Batch ingestion with change detection | Into NWIS |
| Authentication systems | Single sign-on federation | Both (identity) |
| Enterprise data platforms | Publish curated event data and reports through secure APIs or data exports | Out of NWIS |

**Principles:** read-only access to operational systems; clearly documented data contracts (what fields, units and frequency); secure interfaces with authentication; retry and error handling; and industry data standards (such as WITSML for wellsite data) where OIL systems support them.

## 28. End-to-End User Workflows

These ten workflows cover the main ways people will use NWIS, from a rig-site check to a new report entering the repository.

| Workflow | Input | System processing | User view | Decision support output |
| --- | --- | --- | --- | --- |
| **A. Engineer opens active well** | Engineer signs in and selects well | Loads metadata, latest depth, formation estimate, nearby wells, current risks | Active Well Overview with at-a-glance panel | Immediate picture of position and upcoming risks |
| **B. Find nearby wells** | Sets radius (e.g. 5 km) and filters | Spatial query; similarity ranking; event badges | Map and ranked list with "why similar" tags | Short-list of the most relevant offset wells |
| **C. Investigate historical incident** | Clicks an event badge on Well B | Retrieves event record, related actions, outcome, source | Offset Well Details with event card and source link | Understanding of what happened, why and how it was handled |
| **D. Natural-language search** | Types "What happened in nearby wells at 3150 m?" | Interprets intent; hybrid search; retrieves records; LLM summarises with citations | Answer panel with wells, depths, events and links | Evidence-backed answer or a clear "no evidence" message |
| **E. Compare with offset wells** | Selects 3 offset wells and the active well | Aligns by depth or formation; overlays events, mud weights, casing | Cross-Well Comparison tracks | Visual view of where and how offsets differed from the current plan |
| **F. Receive proactive alert** | Live depth approaches a historical problem interval | Look-ahead check; parameter analysis; risk score; alert generation | Banner + at-a-glance panel update | Early warning with severity and confidence |
| **G. Investigate why an alert was generated** | Taps "Why?" | Retrieves contributing signals, events and documents | Explanation list with evidence links | User can judge whether the alert is relevant |
| **H. Review historical mitigations** | Opens "Recorded mitigations" on an alert | Retrieves actions and outcomes from similar events | List of actions with outcomes and sources | Options to consider; engineer judges applicability |
| **I. Management reviews insights** | Manager opens Analytics | Aggregates NPT, events, alerts by field, formation, period | KPI dashboard and trend charts | Where to focus planning, training and resources |
| **J. New document enters repository** | Data team uploads a WCR | Pipeline: OCR → extraction → validation → review → indexing | Data Health screen shows progress and review items | New knowledge available to all users after validation |

## 29. Example Real-World Scenarios

The four scenarios below show NWIS giving earlier, evidence-backed awareness; in each, the engineer makes the decision and no outcome is guaranteed. *All wells, depths and values are illustrative.*

### Scenario 1 — Mud loss approaching a fractured formation

| Step | What happens |
| --- | --- |
| Situation | Well NW-07 is drilling the 12¼ in section at 2,380 m. Formation F2, a fractured carbonate in this illustration, is expected at about 2,445 m. |
| NWIS finds | 9 wells within 5 km; 5 ranked as similar. 3 of those 5 recorded mud losses between 5 and 70 m below the F2 top. |
| Early signal | At 2,395 m, a formation-based **Advisory**: "Entering F2 in \~50 m; losses recorded in 3 of 5 similar wells." |
| Live data | At 2,410 m, return flow drops 8% over 15 minutes. The advisory becomes a **Warning** (combined intelligence). |
| Explanation | "Why?" lists the three offset events, the flow trend, and links to DDR pages. |
| History shows | Well B cured losses with an LCM pill and a small mud-weight reduction; Well D had total losses after continuing at the original mud weight. |
| Engineer decides | Reviews mud weight against the offset cases, confirms LCM readiness, and consults the office using the shared evidence view. |
| Value | The team sees the relevant history before entering F2, not after losses begin, and discusses it with facts in hand. |

### Scenario 2 — Stuck pipe in a reactive shale

| Step | What happens |
| --- | --- |
| Situation | A deviated well (about 35°) is drilling the 8½ in section through Formation F3, a shale described in offset reports as swelling. |
| NWIS finds | 2 of 4 similar wells had stuck pipe in F3; both were at high angle; one required fishing, with 60+ hours of NPT recorded. |
| Early signal | Formation-based **Advisory** on entering F3. |
| Live data | Over two connections, torque rises above the expected band and overpull on connections increases. A **Warning** appears: "Torque and drag trend resembles the pattern recorded before stuck pipe in Well C." |
| History shows | Recorded practices in the successful offset well included more frequent wiper trips and a different mud inhibition approach. |
| Engineer decides | Compares the current mud and trip practice with offset records and decides on hole-cleaning and tripping measures. |
| Value | A pattern that might otherwise be noticed late is placed next to a directly comparable past case. |

### Scenario 3 — Overpressure and kick indicators

| Step | What happens |
| --- | --- |
| Situation | An appraisal well approaches 3,150 m. Offset WCRs describe raised pore pressure below a sealing layer at a similar depth. |
| NWIS finds | 2 nearby wells increased mud weight in steps near 3,100–3,200 m; one recorded a kick at 3,180 m. |
| Early signal | Depth-based **Advisory** 150 m ahead: "Offset wells recorded overpressure indications and one kick in the next 150 m." |
| Live data | A drilling break (sudden ROP increase) and a rise in background gas. Combined with history, a **Critical** alert is raised. |
| Explanation | Lists the offset kick, the mud-weight profile of offset wells, and the live indicators, with confidence shown. |
| Engineer decides | The crew follows OIL's established well-control procedures. NWIS provides context; it does not replace those procedures. |
| Value | The team is already aware of the offset pressure history when the indicators appear. |

### Scenario 4 — Cementing issue at a casing point

| Step | What happens |
| --- | --- |
| Situation | The team is planning the 9⅝ in casing cement job in a weak formation. |
| NWIS finds | 3 offset wells lost returns during cementing at the same casing point; 2 needed remedial cement squeezes. |
| Early signal | Planning-stage **Advisory** on the Cementing tab: "Losses during cementing recorded in 3 similar wells at this casing point." |
| History shows | The offset well without problems used a lighter slurry design and a lower pump rate, as recorded in its cementing report. |
| Engineer decides | Cementing engineers review these records while finalising the job design. |
| Value | Historical cementing lessons are used at the planning stage, where changes cost least. |

## 30. Recommendations Engine

NWIS recommends by showing what was done before and what happened, not by issuing orders.

### 30.1 Qualities of every recommendation

| Quality | Meaning |
| --- | --- |
| Evidence-based | Built from recorded actions in named wells |
| Context-aware | Matched on formation, depth, hole section, trajectory and current conditions |
| Historical | Shows what was done and the recorded outcome |
| Explainable | States why those cases were chosen |
| Non-prescriptive | Leaves operational judgement to qualified personnel |

### 30.2 Prescriptive vs NWIS style

| Avoid (prescriptive) | NWIS style (evidence-based) |
| --- | --- |
| "Reduce mud weight immediately." | "Wells A, B and C experienced similar loss behaviour in F2. Their recorded actions included an LCM pill (A, B) and a mud-weight reduction of 0.02–0.03 sg (B, C). Losses were recorded as cured in A and B. Review these practices and assess applicability to the current operation." |
| "Pull out of hole now." | "In Well C, a similar torque trend preceded stuck pipe. The recorded response was a wiper trip and increased circulation. Consider reviewing hole-cleaning status." |

### 30.3 How recommendations are built

1. Find past events similar to the current situation (same risk type, similar formation, depth and context).
2. Retrieve the actions recorded for those events and their recorded outcomes.
3. Group similar actions and count how often each was followed by a good outcome.
4. Present actions with their wells, outcomes, sources and confidence.
5. Record user feedback on whether the recommendation was useful.

## 31. Reporting & Analytics

NWIS produces ten standard reports from the same verified knowledge, each exportable and each carrying its sources.

| Report | Content | Main audience | Export |
| --- | --- | --- | --- |
| Well summary | Metadata, formations, events, NPT, casing, mud, lessons | Engineers, managers | PDF |
| Offset well summary | Ranked offset wells with key events and similarity reasons | Engineers, planners | PDF, Excel |
| Drilling event report | All events of a type within a radius or field | Engineers | PDF, CSV |
| NPT analysis | NPT by cause, formation, section, well and period | Managers | PDF, Excel, dashboard snapshot |
| Formation risk report | Events and risks per formation across wells | Geologists, planners | PDF |
| Historical incident report | Detailed record of a single incident with sources | Engineers, safety teams | PDF |
| Risk trend report | Alerts and risk levels over time for a well or field | Managers | PDF, dashboard snapshot |
| Daily intelligence summary | Today's depth, upcoming risk intervals, alerts and acknowledgements | Wellsite and office teams | PDF |
| Cross-well comparison report | Aligned tracks of selected wells | Engineers, geologists | PDF, image |
| Lessons learned report | Curated lessons by formation or risk type | All, training | PDF |

All exports are logged and marked with the generating user, date and data version.

## 32. KPIs & Metrics

Success is measured in the pilot against a baseline taken before NWIS; all numeric targets below are **pilot / illustrative targets**, to be confirmed with OIL.

### 32.1 Operational KPIs

| KPI | How measured | Pilot / illustrative target |
| --- | --- | --- |
| Time to find relevant offset-well history | Timed tasks before and after NWIS | Hours reduced to minutes for standard questions |
| Repeated incidents | Incidents of a type previously recorded in offset wells in the same interval | Downward trend over successive wells |
| Avoidable NPT | NPT linked to risks that NWIS had flagged in advance | Downward trend; measured, not assumed |
| Early risk identification | Lead time between NWIS alert and incident onset (when incidents occur) | Alerts ahead of the interval in most tested cases |
| Decision-support turnaround | Time from question to evidence-backed answer | Under 1 minute for search; under 10 seconds for map and alerts |
| Adoption | Active users per week; alerts reviewed | Steady use by pilot team |

### 32.2 System KPIs

| KPI | Meaning | Pilot / illustrative target |
| --- | --- | --- |
| Search relevance | Share of top results judged relevant | Precision@5 ≥ 0.8 on a test question set |
| Document extraction accuracy | Correctly extracted fields vs expert labels | F1 ≥ 0.85 on key fields |
| OCR accuracy | Correct characters on scanned test pages | ≥ 95% on typed scans |
| Event extraction accuracy | Correct events (type, depth, formation) | F1 ≥ 0.8 |
| Alert precision | Share of alerts judged relevant by engineers | Improving each review cycle |
| False-alert rate | Alerts judged not relevant | Falling each review cycle |
| Model performance | Recall on historical incidents in back-tests | Reported per risk type |
| System response time | Time for map, search, alert updates | Map < 2 s; alert update < 10 s after new data |

## 33. Business & Operational Impact

NWIS is expected to make decisions faster and better informed, and to keep OIL's drilling experience available for the long term.

| Stakeholder | Benefit |
| --- | --- |
| Drilling Engineer | Minutes instead of hours to find offset history; early warning with evidence; fewer surprises |
| Wellsite Team | Simple, clear alerts and one relevant past case at the moment it matters |
| Geologist | Formation-level view of drilling events; better correlation between geology and drilling behaviour |
| Drilling Manager | Consolidated risk and NPT view; better-informed planning and resource decisions |
| Management | Visibility of recurring issues and the value of digital investments |
| Knowledge Management Team | A structured, growing repository instead of unsearchable archives |
| OIL Organisation | Preserved institutional knowledge; more consistent operations; support for NPT reduction |
| Future Engineers | Learn from every past well from day one |

**Impact themes**

- **Faster decisions:** relevant history in seconds rather than hours.
- **Better knowledge reuse:** lessons recorded once, used many times.
- **Less dependence on individual memory:** knowledge stays when people move on.
- **Improved safety awareness:** hazards surfaced earlier and shared widely.
- **Potential reduction in NPT:** by helping teams prepare for known problem intervals; actual reduction to be measured.
- **Better planning:** well plans informed by structured offset history.
- **Better training:** real cases for new engineers.
- **More consistent operations:** the same evidence available to every team.
- **Better collaboration:** field and office look at the same facts.

## 34. Safety & Human-in-the-Loop

NWIS informs people; it never acts on the well.

- NWIS is **not** an autonomous drilling control system.
- It does **not** connect to, or control, any drilling equipment.
- It does **not** replace engineers, supervisors or established procedures (including well-control procedures).
- It **provides** evidence, alerts, patterns and decision support.
- **Final decisions remain with authorised, qualified personnel.**

> **AI → Recommendation → Human Review → Operational Decision**

### 34.1 Guarding against blind reliance on AI

| Safeguard | How it helps |
| --- | --- |
| Evidence with every output | Users judge the facts, not a bare score |
| Confidence always shown | Low-confidence outputs are visibly weaker |
| Non-prescriptive wording | Recommendations describe past practice, not orders |
| Absence of alert is not "all clear" | Interface states that NWIS only knows what is recorded |
| Acknowledgement and notes | Users record their own reasoning |
| Training | Users are taught what NWIS can and cannot do |
| Read-only integration | No technical path exists for NWIS to change rig operations |

## 35. False Positives / False Negatives

Like any predictive system, NWIS will sometimes warn when nothing happens and sometimes miss something that does; the design limits both and makes them visible.

| Term | Plain meaning | Consequence |
| --- | --- | --- |
| False positive | An alert when no problem develops | Wasted attention; if frequent, users stop trusting alerts |
| False negative | No alert before a problem that does develop | Missed early warning; users may over-trust silence |

### 35.1 Why errors occur

- **Incomplete historical data:** events not recorded, or reports missing.
- **Changing geology:** the active well may meet conditions no offset well saw.
- **New drilling methods:** new mud systems, bits or techniques change behaviour.
- **Data quality issues:** OCR or extraction errors, wrong depths.
- **Model drift:** patterns learned from past wells gradually stop matching current practice.

### 35.2 Mitigation

| Strategy | Effect |
| --- | --- |
| Confidence thresholds | Weak signals are shown as information, not warnings |
| Multiple evidence sources | Higher severities need agreement between history and live data |
| Human review | Engineers judge relevance; nothing is automatic |
| Explainability | A wrong reason is easy to spot and dismiss |
| Feedback loops | "Not relevant" marks tune thresholds and ranking |
| Model monitoring | Precision, recall and alert rates tracked per field and risk |
| Continuous retraining | Models refreshed with new wells and verified labels |
| Coverage warnings | "Limited offset data for this formation" is shown explicitly |

## 36. AI Governance

Every AI output in NWIS can be traced to the model version, data and evidence that produced it.

| Element | Practice |
| --- | --- |
| Model versioning | Every model registered with version, training data snapshot, metrics and approver |
| Data lineage | Training and inference data traceable to sources |
| Explainability | Explanation required for every alert and recommendation |
| Human approval | New or retrained models approved by a named technical owner and a drilling domain expert before deployment |
| Auditability | Alerts store the model version, inputs and evidence used |
| Performance monitoring | Scheduled reports on accuracy, alert precision and drift |
| Bias / data coverage checks | Check that fields, formations and periods are represented; flag areas with thin data |
| Retraining strategy | Scheduled (e.g. quarterly) and triggered (drift or major new data) retraining, always re-evaluated before release |
| Incident logging | AI-related issues (a missed or misleading alert) logged, investigated and fed back |
| AI output traceability | Any answer or alert can be reconstructed later for review |

## 37. Feedback Loop

Every alert, answer and recommendation invites a one-tap judgement, and those judgements steadily improve NWIS.

### 37.1 What users can mark

| Rating | Used for |
| --- | --- |
| Useful / Not useful | Recommendations, answers |
| Correct / Incorrect | Extracted facts, event details |
| Relevant / Not relevant | Alerts, search results, similar wells |
| Optional comment | Context ("mud weight already adjusted", "wrong formation") |

### 37.2 What feedback improves

| Area | How |
| --- | --- |
| Search ranking | Results marked relevant rise; irrelevant ones fall |
| Alerts | Thresholds tuned where alerts are often marked not relevant |
| Recommendations | Actions judged useful are shown first |
| Models | Confirmed and rejected alerts become new training labels |
| Knowledge quality | Corrections go to the review queue and update records |

The cycle is: **use → feedback → review → improve → redeploy → use**. Feedback is reviewed by the data team before it changes models, so that one person's mistaken mark cannot distort results.

## 38. MVP / Hackathon Scope

The MVP demonstrates the whole value chain end to end on sample data: from a scanned report to an explained alert on a live (simulated) well.

### 38.1 MVP data

The prototype uses **representative sample data**, not OIL production data: publicly available or synthetic well reports modelled on typical WCR/DDR structure, a synthetic field of about 15–25 wells with coordinates, formation tops and events, and a simulated eRTMAC stream replaying a synthetic active well. All demo data is clearly labelled as such.

### 38.2 MVP Must-Have Features

| # | Feature | Demonstrates |
| --- | --- | --- |
| 1 | Document upload with OCR for scanned pages | Document intelligence |
| 2 | NLP extraction of depth, formation, event type, mud weight, action, outcome | AI extraction |
| 3 | Structured knowledge repository with source links | Institutional memory |
| 4 | Active well selection and overview | Context |
| 5 | Nearby wells map with radius and key filters | Geospatial intelligence |
| 6 | Offset well details with event list and depth chart | Offset well intelligence |
| 7 | Keyword + semantic search with source traceability | Knowledge search |
| 8 | Cross-well comparison by depth and by formation | Correlation |
| 9 | Similar-well ranking with "why similar" | Explainable similarity |
| 10 | Risk indication from historical occurrence + simple anomaly detection | Predictive analytics |
| 11 | Depth-, formation- and parameter-based alerts with "Why?" | Proactive, explainable alerts |
| 12 | Simulated eRTMAC real-time stream | Real-time integration concept |
| 13 | At-a-glance dashboard | Field usability |
| 14 | Natural-language assistant built on read-only MCP tool servers, with a visible tool trace | Grounded, auditable AI |
| 15 | Offline well pack for the active well, connection-status indicator and queued acknowledgements (PWA) | Low-connectivity field access |

### 38.3 MVP Nice-to-Have Features

- Voice questions through self-hosted Whisper, with transcript confirmation.
- Hindi and Assamese questions and answers through self-hosted IndicTrans2, with protected technical terms.
- Low-bandwidth mode (compressed data, cached map tiles, images on demand).
- Recorded mitigations shown alongside alerts.
- Feedback buttons on alerts and results.
- Human review queue for low-confidence extractions.
- Simple PDF export of well summary.

### 38.4 Future Production Features

- Real eRTMAC and OIL database integration.
- Trained ML models per risk type on OIL history.
- Enterprise SSO, full RBAC and audit.
- Knowledge graph at scale; enterprise vector search.
- Rig-site edge node, so the assistant and voice keep working when the link to the central data centre drops.
- Complete reporting suite and management analytics.
- Model governance tooling and automated retraining.

### 38.5 Suggested demo flow (about 9 minutes)

1. Upload a scanned DDR; show extracted events with confidence and source highlight.
2. Select the active well; show the map, radius and ranked similar wells with reasons.
3. Open an offset well; show its event depth chart and lessons.
4. Ask: "Which nearby wells had mud losses in F2?"; show the cited answer and the MCP tool trace beneath it.
5. Ask the same question by voice in Hindi; show the transcript, then the Hindi answer with the same sources.
6. Start the simulated stream; the well approaches F2; show the advisory, then the warning.
7. Tap "Why?"; open the evidence page; show recorded mitigations.
8. Switch the tablet offline: cached offset history is still readable, live alerts are clearly marked paused; acknowledge the alert with a note.
9. Reconnect: the acknowledgement syncs; close on the human decision.

## 39. Six-Member Team Responsibility Split

Each of the six members owns one solution module end to end, with Member 6 holding integration and the demo together.

| Member | Module | Responsibilities | Main deliverables | Depends on | Suggested skill profile |
| --- | --- | --- | --- | --- | --- |
| **1** | Frontend / UX | Screen design, at-a-glance panel, map, comparison tracks, alert views, tablet layout | Clickable UI covering MVP screens; UX style guide | APIs from Member 2; data shapes from Member 5 | UI design, web front-end, data visualisation |
| **2** | Backend / APIs | Service design, APIs for wells, map, search, alerts, assistant; access control | Documented API catalogue; working services | Repository (5), AI outputs (3, 4) | Web back-end, API design, security basics |
| **3** | AI / NLP / OCR | Document processing, OCR, entity and event extraction, semantic search, RAG assistant | Extraction pipeline with measured accuracy; search and assistant | Sample documents (5); labelled test set | NLP, OCR, language models, evaluation |
| **4** | ML / Risk prediction | Similarity ranking, occurrence rates, anomaly detection, risk scoring, explanations | Risk engine with back-test results; explanation templates | Structured events (3, 5); simulated stream (6) | Machine learning, statistics, time-series |
| **5** | Data engineering / Database | Data model, synthetic dataset, normalisation rules, repository, geospatial and vector setup, data quality checks | Populated repository; data dictionary; quality report | Domain input from team research | Databases, geospatial data, data modelling |
| **6** | Integration / DevOps / Testing / Product | Simulated eRTMAC stream, packaging, test plan, demo script, documentation, pitch | One-command demo environment; test results; pitch deck | All modules | Systems integration, testing, communication |

**Shared duties:** everyone learns basic drilling vocabulary; everyone reviews the demo flow; Member 6 runs weekly integration checks.

**Owners of the added capabilities**

| Capability | Owner | Support |
| --- | --- | --- |
| MCP tool servers and tool trace | Member 2 (Backend) | Member 3 (assistant behaviour), Member 4 (risk tools) |
| Voice and Indian-language questions and answers | Member 6 (Integration) | Member 3 (glossary, evaluation) |
| Offline PWA, well pack and sync | Member 1 (Frontend) | Member 5 (well-pack content), Member 2 (sync service) |

## 40. Development Roadmap

NWIS is built in eight phases, with two gates before real operational use; the SIH MVP delivers a thin slice of Phases 1–6 on sample data.

```text
Phase 1 --> Phase 2 --> Phase 3 --> Phase 4 --> Phase 5 --> Phase 6 --> <GATE A> --> Phase 7 --> <GATE B> --> Phase 8
Data        Document    Knowledge   Map and     Risk        Real-time               Pilot in                  Production
foundation  intelligence repository search      analytics   integration             one field                 rollout

GATE A: OIL data access in place and accuracy targets met on OIL history
GATE B: pilot KPIs reviewed with OIL
```

The pilot is the turning point: it only starts once OIL data access is in place and extraction and alert accuracy meet agreed targets on OIL history.

| Phase | Objective | Major features | Dependencies | Expected outcome |
| --- | --- | --- | --- | --- |
| 1. Data foundation | Agree data model and sources | Data dictionary, well master, sample/pilot dataset, storage | Data access (sample for MVP; OIL data for pilot) | Common data backbone |
| 2. Document intelligence | Turn reports into facts | OCR, extraction, confidence, review queue | Phase 1; labelled test documents | Measured extraction accuracy |
| 3. Knowledge repository | Link facts into knowledge | Relationship layer, lineage, semantic index | Phases 1–2 | Queryable institutional memory |
| 4. Map & search | Make knowledge accessible | Nearby wells map, filters, search, offset well view, comparison | Phase 3 | Engineers find history in minutes |
| 5. Risk analytics | Identify risks from history | Similarity, occurrence rates, anomaly detection, explanations | Phases 3–4; enough labelled events | Explained risk indications |
| 6. Real-time integration | Connect live context | eRTMAC read-only connection (simulated in MVP), alert engine | Interface agreed with OIL | Proactive alerts on live depth |
| 7. Pilot | Validate in one field | Deployment on OIL infrastructure, training, feedback, KPI baseline | Gate: data access and accuracy targets | Evidence of value and usability |
| 8. Production rollout | Scale across OIL | Multi-field support, SSO, HA, governance, retraining | Gate: pilot KPIs reviewed with OIL | Organisation-wide decision support |

## 41. Testing Strategy

NWIS is tested both as software and as an advisor: it must work correctly, and its extractions, rankings and alerts must be accurate enough to trust.

| Test type | What is tested | How |
| --- | --- | --- |
| Functional | Every FR in Section 7 | Test cases per requirement ID |
| Data validation | Units, ranges, duplicates, depth and date consistency | Automated rules on every load; sample audits |
| AI/NLP accuracy | OCR and extraction vs expert labels | Held-out labelled document set |
| Search | Relevance of results | Fixed question set with expected answers; Precision@K |
| ML model evaluation | Risk scores and anomaly detection | Back-tests on historical wells (Section 42) |
| Alert | Right alert, right time, right severity, no duplicates | Replay historical wells through the simulated stream |
| UI/UX | Speed of understanding, ease of use | Timed tasks with target users; five-second test on the at-a-glance panel |
| Performance | Response times, concurrent users, stream load | Load tests with many simulated wells |
| Security | Access control, API security, data leakage | Role tests, vulnerability scans, penetration test before pilot |
| User acceptance | Fitness for real work | Pilot users run real scenarios and sign off |

**Validation with historical data (back-testing).** Take a completed well, hide its own records, and replay its drilling as if it were live. Check whether NWIS, using only wells drilled before it, would have raised relevant alerts ahead of the incidents actually recorded, and how many alerts would have been irrelevant. This gives realistic evidence without any risk to live operations.

## 42. Model Evaluation

Each AI component is judged by a small set of standard metrics, reported in plain language.

| Area | Metric | Plain meaning |
| --- | --- | --- |
| Extraction | Precision | Of the facts extracted, how many are correct |
| Extraction | Recall | Of the facts in the document, how many were found |
| Extraction | F1 score | A single balance of precision and recall |
| Extraction | Extraction accuracy | Share of fields exactly right (value and unit) |
| Search | Relevance | Are results judged useful by engineers? |
| Search | Precision@K | Of the top K results, how many are relevant |
| Search | Recall@K | Of all relevant items, how many appear in the top K |
| Risk prediction | Precision | Of the alerts raised, how many preceded a real problem |
| Risk prediction | Recall | Of the real problems, how many had an alert beforehand |
| Risk prediction | F1 | Balance of the two |
| Risk prediction | ROC-AUC | How well the score separates risky from normal intervals overall (1.0 = perfect, 0.5 = chance); used where enough labelled data exists |
| Alerts | False-alert rate | Share of alerts judged not relevant |
| Alerts | Lead time | How far ahead of an incident the alert appeared |

For safety-relevant risks such as kicks, recall is prioritised (missing a warning is worse than an extra one), balanced by severity levels so extra alerts stay low-key.

**Added capabilities.** Voice is measured by word error rate (the share of words transcribed wrongly) on recorded drilling questions in English, Hindi and Assamese, including samples with rig noise. Translation is measured by an automatic score (chrF) and by expert rating on a drilling phrase set, plus a check that numbers, units and names come through unchanged. MCP tool use is measured as the share of test questions where the assistant called the right tools with the right inputs. The offline mode is tested by cutting the connection mid-task and checking that cached data stays readable and queued actions sync.

## 43. Risks & Mitigation

The largest risks are data quality and access, not technology; the plan addresses them first.

| Risk | Impact | Mitigation |
| --- | --- | --- |
| Poor historical data quality | Wrong or missing facts reduce trust | Confidence scores, review queue, completeness indicators, source links |
| OCR errors | Misread depths or values | OCR quality scoring, range checks, human review of low-confidence pages |
| Sparse events | Too few incidents to train reliable ML | Start with occurrence rates and rules; add ML per risk as data grows; show confidence |
| False alerts | Alert fatigue, loss of trust | Severity tiers, multi-signal requirement, de-duplication, feedback tuning |
| Data integration issues | Delays connecting to OIL systems | Batch/file fallbacks; clear data contracts; start with document archive |
| Limited real-time access | No live context | Manual depth entry and periodic imports still enable depth/formation alerts |
| Model drift | Accuracy declines over time | Monitoring, scheduled re-evaluation, retraining with approval |
| User adoption | Tool not used | Co-design with engineers, at-a-glance design, training, champions |
| Security concerns | Approval blocked; data exposure | On-premise deployment, local LLM, RBAC, audit, security review |
| Infrastructure limitations | No GPU or limited servers | Small local models; CPU-friendly design; optional GPU for assistant only |
| Over-reliance on AI | Users treat silence as "safe" | Human-in-the-loop design, training, explicit coverage warnings |
| Terminology mismatch | Missed search results | Synonym lists maintained with OIL experts; semantic search |

## 44. Assumptions & Constraints

**Assumptions**

1. OIL will provide access to relevant data sources during later deployment (subject to its policies).
2. eRTMAC integration interfaces, fields and frequencies will need to be defined jointly with OIL.
3. Historical data will contain inconsistencies, gaps and scanned documents of variable quality.
4. Initial model performance will depend on the volume of recorded historical incidents.
5. The prototype will use anonymised, simulated or sample datasets, not OIL production data.
6. Some advanced predictions (e.g. pattern-based pre-incident detection) require digitised historical parameter data, which may not exist for older wells.
7. Formation naming and tops are available, or can be compiled, for the pilot field.
8. Deployment will be on OIL-approved on-premise or private-cloud infrastructure.

**Constraints**

- NWIS must be read-only towards eRTMAC and operational systems.
- No sensitive data may be processed by external public AI services.
- The hackathon build must be achievable by six students within the SIH timeframe.
- The system must support, not replace, existing procedures and authority structures.

## 45. Future Enhancements

These capabilities are beyond the MVP and pilot and are listed to show the growth path.

| Enhancement | Value |
| --- | --- |
| Advanced geomechanics intelligence | Wellbore stability and mud-window insight from offset data |
| Digital twin integration | Link NWIS knowledge to a virtual model of the well |
| Sophisticated trajectory comparison | 3D path similarity, dogleg and tortuosity comparison |
| Deeper reservoir analytics | Connect drilling events with reservoir performance |
| Automated daily drilling intelligence summaries | Morning briefing per rig, generated from records |
| Voice-based assistant | Extend the MVP's push-to-talk voice to hands-free, wake-word use at the rig site |
| Multilingual support | Extend Hindi and Assamese question-and-answer to more Indian languages and a fully translated interface |
| Mobile / tablet field app | Dedicated field interface with offline cache |
| Advanced predictive models | Sequence models on high-frequency data, once available |
| Enterprise-wide knowledge graph | All fields and disciplines in one graph |
| Cross-field learning | Lessons from one basin applied carefully to similar basins |
| Automated lessons-learned generation | Draft lessons after each well, reviewed by experts |
| Additional enterprise integrations | Maintenance, logistics, HSE and planning systems |

## 46. Accessibility & Ease of Use

AI in NWIS should make drilling history simpler to use, not add another layer of complexity.

| Principle | How NWIS applies it |
| --- | --- |
| Simple language | Plain words on screen; technical terms explained on hover |
| Visual indicators | Colour plus icon plus word for every status |
| Clear terminology | OIL's own formation and well names, maintained with its experts |
| Minimal clicks | Any key fact within two clicks of the active well |
| Natural-language search | Ask questions instead of learning filters |
| Responsive design | Works on office monitors, laptops and tablets |
| Tablet compatibility | Large touch targets; usable with gloves |
| Readability | High contrast for outdoor light; adjustable text size |
| Role-based simplicity | Each role sees what it needs; advanced tools stay out of the field view |
| Learnability | Consistent layout; built-in examples; short training module |

The platform must work for experienced engineers who want depth, new engineers who need guidance, supervisors who need speed, managers who need summaries, and both office and field staff.

## 47. Offline / Field Environment Considerations

NWIS should stay useful when the rig-site connection is slow or drops, by keeping essential knowledge locally and being honest about what is out of date.

| Challenge | Approach |
| --- | --- |
| Limited connectivity | Lightweight pages; summaries before full documents |
| Intermittent connectivity | Local cache of the active well's offset knowledge, events and key source pages |
| High latency | Pre-loading of the look-ahead interval's history when connected |
| Connection lost | Graceful degradation: last-known data shown with a clear timestamp; live alerts paused and labelled |
| Connection restored | Automatic synchronisation of acknowledgements, notes and feedback |

**What works offline (proposed):** cached summaries, events, lessons, recorded mitigations and selected source pages for the active well and its ranked offset wells; the at-a-glance panel with last-known values; acknowledgements, notes and feedback, queued for sync. **What needs a connection:** live parameter alerts, full-repository search, the AI assistant and voice, unless the optional rig-site edge node (47.2) is installed. NWIS does not claim full offline operation.

### 47.1 Low-connectivity design

NWIS is built as a Progressive Web App (PWA): a web application that installs on a tablet or laptop like an app and can keep working from its own local store.

| Mechanism | What it does |
| --- | --- |
| Installable app shell | The interface itself is stored on the device, so it opens without a network |
| Well pack | While connected, the device downloads a compact package: active well, top ranked offset wells, events in the look-ahead interval, mitigations, lessons and compressed source-page images; refreshed whenever the well moves into a new interval |
| Local store | The well pack is kept in the browser's on-device database, encrypted and limited to the user's authorised wells |
| Status indicator | Online, degraded or offline, with last-sync time, shown on every screen |
| Offline actions queue | Acknowledgements, notes and feedback are saved locally and sent on reconnect; server data wins on conflict, user notes are always kept |
| Delta sync | Only changed records are exchanged, to save bandwidth |
| Low-bandwidth mode | Text-first pages, compressed data, cached map tiles for the field area, images and full documents only on request |
| Priority for alerts | Alerts are small messages sent ahead of other traffic when the link is weak |

### 47.2 Optional rig-site edge node (production)

A small server at the rig or field office can hold the field's knowledge subset, the MCP tool servers, a small language model, Whisper and IndicTrans2. Field devices then reach search, the assistant and voice over the local network even when the link to the central data centre drops, and the edge node synchronises when the link returns. This depends on OIL's infrastructure and security policies; in the MVP, one laptop running everything simulates it.

## 48. Commercial / Organisational Value

NWIS turns OIL's existing archive into a long-term asset that improves with every well.

| Value | Explanation |
| --- | --- |
| Knowledge retention | Expertise survives retirements and transfers |
| Standardisation | One way to record and retrieve drilling events across fields |
| Operational efficiency | Less time searching, more time analysing |
| Reduced dependence on individual experts | Experts' knowledge is shared, and their time is freed for harder problems |
| Better onboarding | New engineers learn from real, local cases |
| Faster learning from past wells | Lessons are reused on the next well, not the next decade |
| Improved data utilisation | Existing reports and databases produce new value without new data collection |
| Better decision quality | Decisions made with evidence at hand |
| Reusability across fields | The same platform and models extend to new areas |
| Enterprise digital transformation | A practical, visible step in OIL's digital journey, building on eRTMAC |

## 49. Why This Solution Is Different

Many tools cover one piece of this problem; NWIS is designed to connect the pieces around the active well.

| Existing type of tool | What it does well | What it lacks for this problem | What NWIS adds |
| --- | --- | --- | --- |
| Document management system | Stores and finds files | Does not extract events or link them to depth and formation | Structured, linked facts from inside documents |
| Generic search engine | Finds words | No drilling context, no depth, no geography | Search scoped by radius, formation and depth, with drilling vocabulary |
| Basic drilling dashboard | Shows live parameters | No historical offset context | Live data compared with offset history |
| Static GIS map | Shows well locations | No event history or relevance ranking | Map linked to history and similarity |
| Chatbot / generic AI assistant | Fluent answers | Can invent facts; no access to OIL records | Answers only from retrieved OIL records, with citations |
| Conventional analytics tool | Charts and reports | Needs structured data that does not yet exist | Creates that structured data from documents |

The distinctive feature of this proposed implementation is the combination, centred on the active well:

> **Geospatial intelligence + historical well knowledge + document intelligence + cross-well correlation + real-time drilling context + predictive risk identification + explainable decision support**

Three design choices make this implementation stand out further:

- **Open, auditable AI access through MCP.** NWIS's assistant reaches data only through read-only MCP tools, so every answer is a traceable sequence of tool calls. Any MCP-compatible AI assistant OIL adopts later can use NWIS knowledge without custom integration.
- **Indian-language voice access.** Field staff can question the drilling archive by voice in Hindi or Assamese, with all speech and translation models self-hosted.
- **Built for weak links.** An offline well pack and queued actions keep essential history available at the rig when connectivity drops.

## 50. Expected Final User Journey

The whole value of NWIS fits in one short journey, from opening the app to making an informed decision.

1. **Engineer opens NWIS.**
2. **Selects the active well.**
3. **Sees nearby wells** on the map.
4. **System identifies the relevant offset wells** and explains why.
5. **Engineer sees historical risks** along the depths ahead.
6. **Engineer asks natural-language questions.**
7. **System provides evidence-backed answers** with sources.
8. **Current drilling data is compared with historical patterns.**
9. **Potential risks are highlighted.**
10. **Engineer receives an early warning.**
11. **Engineer reviews the supporting evidence.**
12. **Engineer makes an informed operational decision.**

## 51. Final Requirements Traceability Matrix

Every requirement in OIL's problem statement maps to at least one NWIS feature, and all are demonstrated in some form in the MVP.

| # | Problem statement requirement | NWIS feature | AI / technology used | User benefit | MVP / Future |
| --- | --- | --- | --- | --- | --- |
| P1 | Display nearby wells on a geospatial map relative to the active well | Nearby Wells Map (FR-MAP-01–11) | PostGIS spatial queries, web map | See the neighbourhood instantly | MVP |
| P2 | Instant access to historical drilling experiences and events from offset wells | Offset Well Details, event list, timeline (FR-OW) | Extraction + repository | History in one click | MVP |
| P3 | Correlate drilling parameters, reservoir characteristics, losses, kicks, stuck pipe, casing, cementing, formation risks across wells | Cross-Well Correlation Engine, comparison view (Sec. 9) | Depth/formation alignment, statistics | Like-for-like comparison | MVP (core); Future (reservoir depth) |
| P4 | Proactive alerts when approaching depths/formations with similar past challenges | Depth, formation, parameter and combined alerts (Sec. 13) | Rules + anomaly detection + risk scoring | Early warning | MVP (simulated stream) |
| E1 | Use AI, NLP, OCR and analytics to extract and structure information from reports | Document Intelligence pipeline (FR-DI, Sec. 22) | OCR, NLP, entity/event extraction | Archive becomes usable data | MVP |
| E2 | Interactive map of nearby wells within a user-defined radius | Radius control and filters (FR-MAP-03, 05) | Geospatial indexing | Focus on relevant wells | MVP |
| E3 | Searchable knowledge repository of events, lessons and mitigations | Knowledge repository + search + assistant (Sec. 8, 17) | Hybrid search, embeddings, RAG with local LLM | Answers in minutes, with sources | MVP (search); assistant Nice-to-have |
| E4 | Correlate geological, drilling and reservoir data by depth and formation | Correlation engine, similar-well ranking (Sec. 9–10) | Similarity scoring, statistics | Relevant offsets, with reasons | MVP (geology, drilling); Future (reservoir) |
| E5 | Predictive analytics for mud losses, stuck pipe, overpressure, torque spikes, cementing issues and other risks | Risk prediction framework (Sec. 11) | Occurrence rates, anomaly detection, ML classifiers | Risk indication with confidence | MVP (rates + anomaly); Future (trained ML on OIL data) |
| E6 | Real-time alerts and recommendations | Alert engine + recommendations engine (Sec. 13, 30) | Rules, risk scores, retrieval of past mitigations | Proactive, evidence-based support | MVP |
| E7 | User-friendly dashboard for field and office personnel | 13 screens + at-a-glance panel, voice questions in Hindi/Assamese, offline well pack (Sec. 15–17, 47) | Responsive web UI | Understand in seconds | MVP |
| S1 | Standalone decision-support platform alongside eRTMAC | Read-only integration design (Sec. 12, 27) | Streaming / API connectors | No disruption to eRTMAC | MVP (simulated); Future (real) |
| S2 | Provide institutional memory | Repository, lineage, feedback loop (Sec. 5, 37) | Knowledge graph, review workflow | Knowledge retained permanently | MVP (core); Future (enterprise-wide) |
| D1 | Use OIL data sources (WCRs, DDRs, mud logging, trajectory, casing, cementing, events, NPT, eRTMAC) | Data architecture, 13 domains (Sec. 21) | Ingestion pipelines | All history in one place | MVP (sample data); Future (OIL data, subject to access) |

## 52. Final "Judges' View" Summary

**What is the problem?** OIL's drilling history sits in thousands of reports, databases and people's memories. Finding the right lesson takes hours, so it often arrives too late to change a decision.

**What is our solution?** NWIS, a standalone decision-support platform beside eRTMAC that turns that history into searchable, linked knowledge and brings it to the engineer at the right depth.

**How does it work?** It reads reports with OCR and NLP, stores the facts with links to their sources, maps nearby wells, finds the most similar ones, compares them by depth and formation, watches live drilling context, and raises explained early warnings.

**Where is AI used?** OCR and NLP read documents; semantic search finds meaning; machine learning spots unusual parameters and scores risk; a locally hosted language model answers questions only through audited, read-only MCP tool calls on NWIS data; self-hosted Whisper and IndicTrans2 let field staff ask by voice in Hindi or Assamese. Clear rules handle what rules do best.

**What makes it practical?** Open-source, on-premise technology; read-only integration; explainable outputs; confidence shown everywhere; a human decides every action; an offline well pack for weak rig-site connections; and an MVP that six students can build and demonstrate.

**Who benefits?** Drilling engineers, wellsite teams, geologists, managers, data teams, leadership, and every future engineer who joins OIL.

**What is the MVP?** A working end-to-end demo on sample data: scanned report → extracted events → map of nearby wells → search → comparison → simulated live well → explained alert → engineer's decision.

**What is the long-term impact?** A permanent, growing institutional memory that supports safer, faster and more consistent drilling decisions across OIL's fields, with benefits measured in a pilot rather than assumed.

### Why OIL would care about this solution

- It gets more value from reports and data OIL already has.
- It helps keep hard-won drilling experience inside the organisation as people move on.
- It supports the goal of reducing repeated incidents and avoidable NPT, with measurable pilot KPIs.
- It builds on eRTMAC rather than competing with it.
- It keeps sensitive data inside OIL and keeps qualified people in charge of every decision.
