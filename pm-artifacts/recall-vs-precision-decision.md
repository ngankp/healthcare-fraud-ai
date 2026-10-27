# Recall vs. Precision — The Cost-Asymmetry Decision

*PM artifact: how the optimization target for PAD was chosen before model training began*
*Details adjusted for confidentiality*

---

## The Decision

For PAD, we optimized for **recall** (catching more fraud) over **precision** (fewer false positives).

This was documented before any training data was touched. Once you see model results, anchoring bias makes the threshold negotiation harder. Setting the optimization target first is a forcing function — it separates the technical decision from the political one.

---

## The Framework: Cost Asymmetry

Every ML product has two types of wrong prediction. The PM's job is to quantify the cost of each — not for the data science team, but for the business decision.

| Error Type | What It Means | Healthcare Fraud Example |
|---|---|---|
| **False Negative** | Model misses a real case | Fraud goes undetected; improper payment not recovered |
| **False Positive** | Model flags a clean case | Investigator reviews a legitimate provider; no fraud found |

The question: **which mistake is more expensive?**

For PAD:

| Error | Estimated Cost | Basis |
|---|---|---|
| False negative (missed fraud) | $50K–$500K per case, unrecovered | Payment history patterns from prior investigations |
| False positive (clean claim) | ~2–4 hours investigator time | Internal time study; investigator hourly rate |

**The ratio is not close.** A single missed fraud case costs more than hundreds of false positive reviews. The optimization target was clear: **recall first, accept more false positives, let investigators filter**.

---

## The Acceptance Criteria (Set Before Training)

| Metric | Minimum | Target | Rationale |
|---|---|---|---|
| Recall | ≥ 70% | ≥ 80% | 30% missed-fraud floor; cost asymmetry drives this number |
| Precision@100 | ≥ 40% | ≥ 60% | Investigator time constraint; too many false positives lose adoption |
| AUC-ROC | ≥ 0.80 | ≥ 0.88 | General class separation quality |
| Inference time | < 2s/batch | < 0.5s | UI responsiveness |

**Note on Precision@100:** This is a more useful operational metric than global precision. Of the top 100 leads the model surfaces, how many convert to real cases? This directly maps to investigator capacity — teams typically have bandwidth for ~100 high-priority reviews per week.

---

## Why Not Optimize for Both?

This is the most common PM pushback on recall-first decisions.

The short answer: precision and recall pull in opposite directions at any given decision threshold. A model tuned for higher precision will miss more fraud. You can shift the threshold, but you can't avoid the tradeoff.

The right question is not "can we get both?" but "which error is more expensive, and by how much?" For PAD, the cost ratio was documented, reviewed by the business, and signed off before training began. The threshold choice was no longer a debate — it was an implementation of a business decision.

---

## Applied to Model Selection

The 5-model comparison evaluated against these pre-set thresholds:

| Model | Recall | Precision@100 | Recall threshold met? | Selected? |
|---|---|---|---|---|
| FCNN | 78% | 48% | ✅ | **Yes — production** |
| Gradient Boost | 74% | 51% | ✅ | No — lower recall vs FCNN |
| Random Forest | 68% | 44% | ❌ | No — below 70% minimum |
| Decision Tree | 61% | 41% | ❌ | Retained for explainability only |
| Logistic Regression | 55% | 38% | ❌ | No |

Gradient Boost had higher precision (51% vs 48%) but lower recall (74% vs 78%). Given the cost asymmetry framework, the 4-point precision gain did not offset the recall loss. FCNN was selected.

**The cost-asymmetry framework made this a straightforward call** — without it, a 51% vs 48% precision difference might have been treated as meaningful signal.

---

## The Counterpart Decision: Where to Stop

Recall-first creates a downstream obligation: **you need investigator capacity to work the leads**.

If precision drops too low (< 40%), investigators spend most of their day on false positives, lose trust in the queue, and revert to legacy tools. This is not a model problem — it's an adoption problem caused by the model. The Precision@100 ≥ 40% minimum threshold was set specifically to guard against this failure mode.

The two thresholds work together: recall floor protects business value; precision floor protects adoption.

---

## When to Revisit

The cost-asymmetry framework is not static. Two conditions warrant re-evaluation:

1. **Investigator capacity changes** — If team size shrinks, the false positive tolerance drops. Precision floor should rise.
2. **Fraud cost distribution shifts** — If schemes become lower-value (e.g., smaller individual claims, higher volume), the cost of missing individual cases decreases. The recall floor could move.

In the PAD monitoring framework, case conversion rate and override rate are the leading indicators of whether the current threshold is still calibrated correctly.
