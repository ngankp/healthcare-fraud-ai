# Demo Notebook — Healthcare Fraud Detection EDA

## What This Is

A hands-on exploration of healthcare claims data patterns that fraud detection AI learns from — using **publicly available CMS Medicare data** (no proprietary Carelon data).

The goal isn't to replicate the production PAD model. It's to demonstrate:
1. What fraud signals look like in real claims data
2. How feature engineering works for this problem
3. Why the three-engine approach (clustering, structured prediction, code analysis) makes sense given the data shape

---

## Dataset

**Source:** [CMS Medicare Provider Utilization and Payment Data](https://data.cms.gov/provider-summary-by-type-of-service)

Public dataset containing:
- Provider NPI, specialty, location
- Procedure codes (HCPCS/CPT)
- Number of services, beneficiaries
- Average submitted/allowed/payment amounts
- Geographic information

This is the type of data the Provider Behavior Analytics Engine works with (at much larger scale and with actual adjudication outcomes).

---

## Notebook Contents

### `fraud-detection-eda.ipynb`

**Section 1: Understanding Claims Data Structure**
- What a healthcare claim looks like
- Key fields: NPI, CPT/HCPCS codes, allowed amount, number of services
- Provider-level vs. claim-level granularity

**Section 2: Provider Behavior Patterns**
- Distribution of services per provider by specialty
- Geographic cost variation for the same CPT code
- Outlier detection: which providers are statistically anomalous?
- Visualization: geographic clustering using DBSCAN concepts

**Section 3: Medical Code Analysis**
- CPT code co-occurrence patterns
- Which code combinations appear together most frequently?
- What "suspicious" unbundling looks like in aggregate data
- Word embedding intuition: why Code2Vec works for medical codes

**Section 4: Building an Anomaly Score (Simple Baseline)**
- Feature engineering: cost-per-service vs. specialty average, code complexity score, geographic deviation
- Simple rule-based baseline (replicating what legacy dashboards did)
- Where ML adds value vs. where rules are sufficient
- PM decision: when to use rules vs. ML

**Section 5: PM Perspective — What These Patterns Mean for Product**
- Which patterns are high-confidence enough to surface to investigators?
- What additional data would improve detection?
- How model outputs translate to investigator workflow

---

## Setup

```bash
# Clone this repo (replace with actual repo URL when published)
git clone https://github.com/ngankp/healthcare-fraud-ai.git
cd healthcare-fraud-ai/notebooks

# Install dependencies
pip install -r requirements.txt

# Launch Jupyter
jupyter notebook fraud-detection-eda.ipynb
```

**Requirements:**
```
pandas
numpy
matplotlib
seaborn
scikit-learn
jupyter
requests
```

---

## A Note on Scope

This notebook demonstrates pattern recognition and feature analysis — the concepts underlying the Provider Behavior Analytics and MediCode Insight engines.

The production PAD system ran on proprietary claims data at 10x+ this scale, with a full MLOps pipeline on GCP. This demo illustrates the *type* of analysis, not the production implementation.

If you're interested in the architecture design decisions, see [architecture/system-architecture.md](../architecture/system-architecture.md).  
If you're interested in the PM evaluation framework used to select and deploy models, see [pm-artifacts/ml-evaluation-framework.md](../pm-artifacts/ml-evaluation-framework.md).
