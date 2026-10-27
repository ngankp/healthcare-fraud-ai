# Problem Discovery and UX Strategy — PAD (Provider Anomaly Detection)

*PM Artifact · Carelon (Elevance Health) · Details adjusted for confidentiality*

---

> **On the evidence below:** Investigator quotes are representative of recurring themes from working sessions and informal feedback. Specific time estimates and financial figures are illustrative, drawn from internal time-study patterns. The product decisions, design rationale, and outcome table reflect the actual approach used.

---

## The Problem in One Sentence

SIU investigators were spending more time finding fraud leads than working them — using three disconnected tools, no ML prioritization, and no ability to explain their case selection decisions.

---

## Discovery: How We Learned What the Problem Actually Was

### What We Were Told At The Start

The initial brief was tactical: "consolidate the three legacy dashboards into one." That's a UI migration, not a product problem.

Working sessions with the SIU team over three weeks changed the framing entirely.

**What we did:**
- Four structured sessions with SIU investigators (3–5 investigators per session)
- Observed two investigators running their actual morning start-of-day workflow end-to-end
- Reviewed the outputs they were saving to Excel for cross-referencing
- Asked: *"If the tool disappeared tomorrow, what would break first? What would you rebuild yourself?"*

**What we learned that changed the product direction:**

| Initial assumption | What investigators actually said | What changed |
|---|---|---|
| "They need one dashboard" | "I need to know which lead is worth my time, not just a list" | Problem isn't consolidation — it's prioritization |
| "ML will improve detection" | "I can't open a case if I can't explain why" | Explainability isn't optional UX — it's a professional requirement |
| "More leads = better" | "More leads with no ranking just means more Excel work" | Volume without signal hierarchy makes things worse |
| "Rules miss fraud patterns" | "I know there's fraud we're not catching but I can't prove it" | ML detection gap is real, but trust is the adoption lever |

---

## Jobs-to-Be-Done: What Investigators Were Actually Trying to Accomplish

### Functional Job
> **"Help me identify which of 200+ flagged providers is most likely committing fraud — and show me why — before I commit any investigation time."**

- Every day: decide which cases to open, prioritize limited time across high-value targets
- Every week: convert prioritization judgment into defensible case documentation
- Every investigation: connect behavioral signals across claim type, geography, and billing code

### Emotional Job
> **"Let me feel confident that I'm not missing real fraud — and that I can defend the cases I chose."**

- Investigators feel responsible for what they miss, not just what they catch
- Professional identity is tied to case quality, not throughput
- False positives are annoying; missing real fraud is a career risk

### Social Job
> **"Give me enough to explain my lead selection to my supervisor without it sounding like a guess."**

- Every case opened requires documented rationale
- Supervisors ask "why this one?" — pure ML scores with no explanation are not an answer
- Teams with poor prioritization discipline are visible as an operational risk

### The Insight That Reframed Everything

SIU investigators aren't just fraud detectors. They are evidence-builders whose professional reputation is on the line with every case they touch. A tool that improves detection but undermines their ability to explain their decisions is worse than no tool — it creates liability without support.

**This reframe drove every major product decision that followed.**

---

## Before State: The Fragmented Process

```
START OF INVESTIGATION DAY
│
├─ STEP 1: Dashboard A (behavioral analytics)
│   ↓ Log in, run manual filter, export results to Excel
│   ↓ "These are providers flagged for billing volume anomalies"
│   ↓ No priority ranking — every result looks equally urgent
│
├─ STEP 2: Dashboard B (claim-level rules)
│   ↓ Log in again (separate system), run own query
│   ↓ Cross-reference with Dashboard A results manually in Excel
│   ↓ Match providers across systems: "Is NPI 123456 in both?"
│
├─ STEP 3: Dashboard C (geographic/network view)
│   ↓ Third login, third query
│   ↓ Add geographic context — "Where are they operating?"
│
├─ STEP 4: Manual prioritization (most brittle step)
│   ↓ Investigator judgment: Which of ~200 flagged providers to work?
│   ↓ No ML scoring — experience + recent memory drives the call
│   ↓ Recency bias: whoever was opened last felt most important
│
└─ STEP 5: Open a case — with no paper trail for prioritization
    ↓ Pull claim history from a fourth system
    ↓ Document findings manually
    ↓ No record of WHY this case vs. the other 199
```

### Pain in the investigators' own words

> *"I spend more time finding leads than working them."* — On lead discovery overhead

> *"I know fraud is happening but I can't prove it from the data I have."* — On detection gaps in rules-based tools

> *"If I get it wrong, I can't explain my decision process to my supervisor."* — On prioritization accountability

> *"We're probably missing whole categories of fraud because the rules don't cover it."* — On systematic blind spots

### Quantified cost of the before state

| Problem | Estimated Impact |
|---|---|
| Lead discovery overhead | 2–3 hours per investigator per day before active investigation begins |
| Cases reviewable per week | ~8–12 per investigator (constrained by discovery time) |
| Undetectable FWA patterns | Multi-provider coordination, systematic upcoding — invisible to rule-based tools |
| Prioritization consistency | Subjective per investigator; not defensible under audit |
| False positive tracking | None — no signal fed back to improve lead quality |

---

## Problem Statement (Refined After Discovery)

SIU investigators lose 2–3 hours per day to multi-tool lead discovery overhead — time that competes directly with actual investigation capacity. The tools they use can't detect sophisticated FWA schemes (multi-provider coordination, systematic code manipulation), and neither the tools nor the investigators have a defensible, consistent method for prioritizing which of 200+ flagged providers to investigate first.

The result: high-value fraud cases are missed or deprioritized not because investigators lack skill, but because the tooling forces every workday to start with low-value reconciliation work.

**The opportunity: one prioritized interface with ML-powered lead ranking and explainability baked into every output — so investigators spend 30 minutes on lead discovery, not 3 hours, and can defend every case they open.**

---

## Solution Framing: Three Engines for Three Blind Spots

Before designing anything, we mapped which fraud patterns investigators were confident catching vs. consistently missing. This drove the three-engine architecture.

```
DETECTION GAP ANALYSIS — what rule-based tools miss
───────────────────────────────────────────────────────────────────────────────

Fraud Type          │ Rule-Based Tools     │ Why Rules Fail
────────────────────┼──────────────────────┼──────────────────────────────────
Billing volume      │ Partial coverage     │ Thresholds don't adjust by
anomalies           │ (top outliers only)  │ specialty or geography
────────────────────┼──────────────────────┼──────────────────────────────────
Individual claim    │ Not covered          │ Rules match known patterns;
FWA risk            │                      │ FCNN finds statistical anomalies
                    │                      │ no rule has been written for
────────────────────┼──────────────────────┼──────────────────────────────────
Medical code        │ Known unbundling     │ Rules catch explicit code pairs;
manipulation        │ rules only           │ Code2Vec catches semantic
                    │                      │ co-occurrence patterns at scale
────────────────────┼──────────────────────┼──────────────────────────────────
Multi-provider      │ Not covered          │ Rules are single-provider;
coordination        │                      │ DBSCAN finds network anomalies
────────────────────┴──────────────────────┴──────────────────────────────────

Each engine was designed to close a specific gap. All three outputs feed
a single composite priority score — not three separate lists.
```

**Why a composite score vs. three separate views:**

In early design discussions, the proposal was three tabs — one per engine. Investigators in prototype testing gave unanimous feedback:

> *"Now I have three lists. I still don't know which one to start with."*

The composite score solves this. If a provider appears in all three engines, they rank higher — automatically. Investigators think in cases, not engine types. The UI follows their mental model.

---

## UX Design Decisions — Evidence, Options Considered, Outcomes

### Decision 1: Decision Tree Path Visualization (Non-Negotiable)

**The problem it solved:**
After the first PAD demo, investigators saw fraud risk scores (0.87, HIGH) with no explanation. Their consistent response:

> *"How do I know it's right? If I open a case and it's wrong, that's on me."*

This was a legitimate professional concern. SIU investigators document every case decision — to supervisors, to legal. A probability score they can't explain creates professional risk, not investigative help.

**Options considered:**

| Option | What it provides | Why rejected/deferred |
|---|---|---|
| SHAP values (feature importance bars) | Quantitative feature contribution | Too technical; "mean SHAP = 0.34" is not defensible to a supervisor |
| FCNN score only | High accuracy ranking | Black box — investigators rejected it in testing |
| Reason code categories (e.g., "HIGH VOLUME") | Plain English label | Not enough detail to differentiate; still felt arbitrary |
| **Decision Tree path visualization** | Step-by-step logic: *avg_charge > $2,400 → services_per_patient > 12 → specialty_deviation > 340% → HIGH* | ✅ Selected — mirrors how investigators already think through cases manually |

**Outcome:** Decision Tree path was retained in production despite failing performance thresholds (61% recall vs. 70% minimum) — because it solved a user trust problem that no higher-performing model could solve.

---

### Decision 2: Single Ranked Queue (Not Three Engine Tabs)

**The problem it solved:**
Early design mirrored the old multi-tool experience: three tabs, one per engine. Investigators still had to decide which tab to start with — same cognitive burden, different interface.

**Options considered:**

| Option | Investigator Experience | Decision |
|---|---|---|
| Three engine tabs (default view) | "Now I have three lists. Still don't know where to start" | ❌ Rejected — shifts cognitive burden rather than removing it |
| Composite score, single queue, engine tabs as filters | One list, ranked by cross-engine composite score; tab = "show me only E1 leads" | ✅ Selected |
| Separate tools maintained, lightweight summary view | Preserves existing mental model; reduces training burden | ❌ Rejected — doesn't solve discovery overhead |

**Design principle applied:** Design for the investigator's mental model (cases to work), not the architecture's structure (engines that detected them). The engines are implementation detail. The lead queue is the product.

---

### Decision 3: Estimated Case Recovery Value (Added After MVP)

**The problem it solved:**
The MVP lead queue had priority tiers (HIGH / MEDIUM / LOW) but no financial context. Investigators used personal judgment to route Medium-priority leads — creating inconsistency across the team.

**What investigators did vs. what we wanted:**
A HIGH lead worth $12K would get worked before a MEDIUM lead worth $85K — purely because of the tier label.

**Options considered:**

| Option | Risk | Decision |
|---|---|---|
| Priority tier only (HIGH/MED/LOW) | Subjective routing within tier; inconsistent across investigators | MVP state — insufficient |
| Case value score alongside priority | Anchors prioritization to financial impact; creates defensible triage | ✅ Added post-MVP |
| Suppress value to avoid gaming | Concern: investigators cherry-pick high-value leads, leave routine cases | Rejected — fraud value concentration is real; high-value leads ARE higher priority |

**Outcome:** Once estimated recovery value was visible, prioritization became consistent across the team without coordination overhead. A formal policy wasn't needed — the information changed behavior.

---

### Decision 4: Morning Notification — Information Architecture

**Design principle:** Investigators needed to open PAD *before* reverting to legacy dashboards. The notification had to create urgency and confidence in under 10 seconds of reading.

```
┌──────────────────────────────────────────────────────────────────────┐
│  FROM:    PAD Lead Queue <pad-alerts@[internal]>                    │
│  SUBJECT: Your PAD leads for Monday — 3 HIGH, 8 MEDIUM             │
│                                                                      │
│  Good morning. Your priority leads are ready.                       │
│                                                                      │
│  ●●● HIGH-priority leads: 3                                         │
│      Estimated combined recovery: $149,500                          │
│                                                                      │
│  ●●  MEDIUM-priority leads: 8                                       │
│      Estimated combined recovery: $203,200                          │
│                                                                      │
│  2 leads flagged by all 3 engines (highest confidence)             │
│                                                                      │
│  [ OPEN MY LEAD QUEUE ]                                             │
│                                                                      │
│  Last week: 12 leads reviewed · 3 converted to active cases        │
└──────────────────────────────────────────────────────────────────────┘
```

**Three signals communicated before a single click:**
1. **Urgency** — HIGH lead count
2. **Financial significance** — combined estimated recovery, not individual amounts
3. **Confidence** — multi-engine flagged leads surfaced explicitly as "highest confidence"

**Last week's activity** was added specifically to close the feedback loop: investigators who saw their conversion rate improved adoption. Seeing that 3 of 12 reviewed leads became real cases made the queue feel productive, not performative.

---

## After State: PAD Workflow

```
START OF INVESTIGATION DAY
│
└─ PAD LEAD QUEUE (single interface)
    │
    ├─ RANKED LEAD VIEW
    │   ┌──────────────────────────────────────────────────────────────┐
    │   │  PRIORITY  │  PROVIDER             │  FLAGS  │  EST. VALUE  │
    │   ├────────────┼───────────────────────┼─────────┼──────────────┤
    │   │  HIGH      │  Dr. J. Smith, TX     │  3/3    │  $82,000     │
    │   │  HIGH      │  Sunrise Clinic, FL   │  2/3    │  $67,000     │
    │   │  MED       │  ABC Medical, NY      │  2/3    │  $31,000     │
    │   │  MED       │  West Coast Labs, CA  │  1/3    │  $28,500     │
    │   └──────────────────────────────────────────────────────────────┘
    │
    ├─ OPEN LEAD → Investigator sees WHY this was flagged
    │   ┌──────────────────────────────────────────────────────────────┐
    │   │  PROVIDER: Dr. J. Smith, NPI 1234567890                     │
    │   │                                                              │
    │   │  [BEHAVIORAL]  Billing volume 8.3x peer median (TX cardiol)│
    │   │  [STRUCTURED]  FCNN score: 0.87 — high FWA likelihood      │
    │   │  [MEDICODE]    Code pair 93306+93308: co-occurrence anomaly │
    │   │                                                              │
    │   │  DECISION TREE PATH:                                        │
    │   │  avg_charge > $2,400 → services_per_patient > 12           │
    │   │  → specialty_deviation > 340% → SCORE: HIGH                │
    │   │                                                              │
    │   │  [OPEN CASE]  [DEFER]  [NOT FWA — CLOSE]                   │
    │   └──────────────────────────────────────────────────────────────┘
    │
    └─ OUTCOME: Investigator knows why the case was surfaced.
        They can defend their prioritization to supervisors.
        They can start investigation immediately — no stitching.
```

---

## The Override → Improvement Loop

The most important post-launch product decision: investigator overrides as a quality signal, not a failure signal.

```
OVERRIDE LEARNING LOOP
─────────────────────────────────────────────────────────────────────

Investigator opens lead → reviews → decides
  [OPEN CASE] ─────────────────────────────→ Case proceeds
  [NOT FWA — CLOSE] ──────────────────────→ Override captured
                                    ↓
  Override data stored: engine, specialty, score, optional comment
                                    ↓
  PM + DS: weekly pattern analysis
  "E1 override rate for oncology: 73% vs 8% for GPs"
  "Sustained 3 weeks — not noise"
                                    ↓
  Root cause: DBSCAN comparing specialty providers to GP baseline
  Oncology volume looks anomalous vs GPs — but it isn't
                                    ↓
  Fix: stratified peer groups by specialty_type
                                    ↓
  Result: E1 override 41% → 18% over 2 months
          Oncology override 73% → 28%
                                    ↓
  Feedback to investigators:
  "Your overrides in October improved 34 leads in November"
                                    ↓
  Trust increases → investigators invest in override quality
  Better overrides → faster model improvement → deeper adoption

─────────────────────────────────────────────────────────────────────
```

**Why this matters:** Investigators who see their feedback improve future lead quality become invested in the tool's quality — not just its output. This is the structural difference between a tool users tolerate and one they advocate for.

---

## Before / After: What Changed

| Dimension | Before PAD | After PAD |
|---|---|---|
| Lead discovery time | 2–3 hrs/day | < 30 min/day |
| Cases reviewable/week | ~8–12 per investigator | 20–30 per investigator |
| Lead explainability | Investigator judgment only | Decision tree path + reason codes |
| Detection coverage | Rules-only patterns | + ML patterns (behavioral, structured, code) |
| Cross-engine correlation | Manual Excel matching | Automated composite scoring |
| Prioritization consistency | Per-investigator subjectivity | Financial value + composite score |
| False positive feedback | No tracking | Override rate tracked; model-quality signal |

---

## PM Takeaway

The product decisions that drove adoption weren't about making the model better. They were about making investigators' professional lives better.

**The three decisions that mattered most:**
1. **Decision tree path** — gave investigators something they could say out loud to a supervisor
2. **Single composite queue** — removed a daily cognitive burden rather than shifting it to a new interface
3. **Estimated recovery value** — turned a subjective prioritization problem into a defensible one

None of these were technically hard. All of them required understanding what investigators were actually trying to accomplish — and that only came from spending time with them before a line of model code was written.
