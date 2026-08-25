# Capstone Report — Content Archetype Clustering & Priority Queue

**Author:** Yumna Kashif  
**Lane:** Structured Content Archetype Clustering  
**Repository:** [FlyRank-ML-Internship](https://github.com/yumna-09/FlyRank-ML-Internship)  
**Date:** August 25, 2026

---

## Abstract

When a website contains far more pages than a content team can manually review, determining which pages deserve attention first becomes a practical prioritization problem.

Using FlyRank’s anonymized 30,000-page starter dataset, this project develops a content-review prioritization framework based on behavioral archetype discovery. A transparent rule-based baseline was first created using observable search-performance signals. KMeans clustering was then applied to group pages according to five standardized metrics:

- Impressions
- Click-through Rate (CTR)
- Average Ranking Position
- Engagement Rate
- Content Age

The initial clustering evaluation produced a silhouette score of **0.3763**, but this result contained evaluation leakage because centroids were fitted and scored on the same data. Validation was redesigned using a **client-held-out approach**, where entire clients were excluded during model fitting.

### Evaluation Results

| Method | Silhouette Score |
|----------|----------|
| Rule-Based Baseline | 0.0013 |
| KMeans (All Data Fit + Score) | 0.3763 |
| KMeans (Client Held-Out) | **0.3046** |

The resulting behavioral clusters were translated into a practical review queue consisting of four actions:

- **Refresh**
- **Boost**
- **Prune**
- **Monitor**

All recommendations are intended as **decision support for human reviewers**, not automated instructions or causal claims regarding traffic outcomes.

---

## 1. Problem Framing

### Decision Supported

Which pages should a content strategist review first when the backlog is too large to inspect manually?

### Unit of Analysis

One content page (`content_id`) represented by observed metrics aggregated across a trailing 90-day window.

### Output

Each page receives:

- Behavioral cluster assignment
- Recommended action
- Supporting reason codes
- Human-review flags

### Human Workflow

1. Review highest-priority pages first.
2. Inspect live content and performance signals.
3. Decide whether the recommended action is appropriate.

### Why Clustering?

Rule-based systems can identify isolated issues such as low CTR or weak rankings.

However, content pages often exhibit combinations of behaviors:

- High impressions with weak CTR
- Low impressions with strong engagement
- Aging content with stable rankings

Rather than evaluating one threshold at a time, clustering groups pages according to multiple observed signals simultaneously.

> This project does not attempt to predict Google's ranking algorithm. It is a behavioral grouping and prioritization system.

---

## 2. Data & Safety

### Source

FlyRank ML Internship warehouse release:

```text
flyrank_pseudonymized_warehouse_release_v20260703
```

### Full Production Catalog

| Metric | Value |
|----------|----------|
| Daily Performance Rows | 78.8M |
| Distinct Content Items | 519,606 |
| Clients | 104 |

### Starter Dataset Used

| Metric | Value |
|----------|----------|
| Rows | 30,000 |
| Aggregation Window | 90 Days |

### Features Used for Clustering

- `impressions_90d`
- CTR
- Average Ranking Position
- Engagement Rate
- Content Age

All features were standardized before clustering.

### Deliberately Excluded

- `trend_direction`
- `trend_pct`
- `content_id`
- `client_id`
- FlyRank health scores
- FlyRank priority scores
- Product action flags

No client names, domains, URLs, page titles, or search queries were used.

---

## 3. Rule-Based Baseline

Before fitting KMeans, a transparent rule-based baseline was created.

### Signal Check 1 — Staleness vs. Decline

**Verdict:** Mixed

| Age Bucket | Decline Rate |
|------------|-------------|
| 0–30 Days | 51.1% |
| 91–180 Days | 61.1% |
| 181+ Days | 47.1% |

Because the relationship was inconsistent, staleness was excluded from the baseline score.

### Signal Check 2 — CTR vs. Ranking Position

**Verdict:** Confirmed

| Position Tier | Median CTR |
|---------------|-----------|
| Top 3 | 0.20 |
| Page 1 | 0.24 |
| Striking Distance | 0.17 |
| Page 3–5 | 0.09 |
| Deep | 0.00 |

### Baseline Formula

```text
eligible * visibility_score * ctr_gap_norm
```

### Eligibility

```text
impressions_90d >= 500
AND avg_position > 0
AND position_tier != 'no_data'
```

### Queue Statistics

| Action | Pages |
|----------|----------|
| Monitor | 13,274 |
| Review CTR | 12,814 |
| Prioritize CTR Review | 3,912 |

---

## 4. Model & Analysis

### Method

**KMeans Clustering**

### Features

- Impressions
- CTR
- Average Ranking Position
- Engagement Rate
- Content Age

### Choosing K

Search performed across:

```text
k = 2 through k = 7
```

Selected value:

```text
k = 7
```

### Why KMeans?

KMeans produces cluster memberships that can be translated into understandable behavioral archetypes for non-technical reviewers.

Clusters are based solely on observed performance signals and not page content.

---

## 5. Evaluation & Validation

### Baseline vs KMeans

| Method | Silhouette Score | Interpretation |
|----------|----------|----------|
| Baseline | 0.0013 | No meaningful separation |
| KMeans (same data fit & score) | 0.3763 | Inflated by leakage |
| KMeans (client-held-out) | **0.3046** | Trustworthy evaluation |

### Client-Held-Out Validation

Validation was redesigned to hold out entire clients:

- ~80% clients used for fitting
- ~20% clients held out
- Centroids fitted only on training clients

Held-out evaluation:

| Metric | Value |
|----------|----------|
| Held-Out Clients | 7 |
| Held-Out Pages | 7,611 |

Silhouette score:

```text
0.3763 → 0.3046
```

### Leakage Checks

1. K-selection leakage audit
2. Feature leakage audit
3. Synthetic leakage trap validation

A separate exercise demonstrated how leakage can inflate performance:

```text
0.591 → 1.000
```

Such features were excluded.

---

## 6. Cluster Interpretation

Seven behavioral archetypes emerged.

Examples include:

- High-traffic top performers (~112K impressions)
- Stable older content
- Low-CTR poorly ranked pages
- High-engagement content
- Near-zero-impression but high-CTR pages

Approximately **2.6%** of pages were weakly matched to their assigned cluster.

### Decline Rate by Cluster

Observed decline rates ranged from:

```text
14.6% – 61.9%
```

This analysis is descriptive and based on seven held-out clients.

---

## 7. Recommendation System

### Priority Queue

| Priority | Action | Trigger | Pages |
|----------|----------|----------|----------|
| 1 | REFRESH | Strong rankings, declining visibility, weak CTR | 1,447 |
| 2 | BOOST | Positions 11–20 with CTR opportunity | 4,348 |
| 3 | PRUNE | Deep rankings, declining visibility | 405 |
| 4 | MONITOR | No strong intervention signal | 23,800 |

### Human Review Requirement

Mandatory review applies to:

- All PRUNE candidates
- Top 10% visibility pages
- Uncertain-score pages
- Weak-cluster-fit pages

Total flagged pages:

**7,012 (23.4%)**

### Never Automate

- URL deletion
- URL redirects
- Brand-page modifications
- Legal-page modifications
- Navigation changes
- URL-structure changes

---

## 8. Limitations

- Cross-sectional, not causal
- Does not predict Google's ranking algorithm
- Not semantic clustering
- Small held-out sample for decline analysis
- Uses starter sample (30,000 pages)
- Baseline not re-tested under held-out validation
- Single snapshot rather than longitudinal data
- Playbook thresholds remain heuristic

---

## 9. Reproducibility

### Random Seed

```python
42
```

### Relevant Notebooks

```text
work/notebooks/capstone.ipynb
work/notebooks/w05_model.ipynb
work/notebooks/w06_validation_audit.ipynb
work/notebooks/w07_action_playbook.ipynb
```

### Environment

```bash
pip install -r requirements.txt
```

Run notebooks inside:

```text
work/notebooks/
```

### Ranked Action Queue

Release asset:

```text
ranked_action_queue.csv
```

### Repository

https://github.com/yumna-09/FlyRank-ML-Internship

---


## 10. Acknowledgments & Data Credit

Built on the **FlyRank ML Internship dataset** — real, pseudonymized production search and analytics data provided for the internship track.

**FlyRank:** https://flyrank.ai

---

> **Claims checklist before submitting:** observed / measured / directional / decision-support language throughout · no causal claims without an experiment or causal design · no claim of predicting Google's ranking algorithm · no client-identifying details · client-held-out validation used for clustering evaluation · decline rate was measured only after clustering and was not used as a clustering feature · recommendations are intended to support human review and prioritization, not automate content decisions or site changes.

---
