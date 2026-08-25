# Capstone Report — Content Archetype Clustering & Priority Queue

* **Author:** Yumna Kashif
* **Lane:** Structured Content Archetype Clustering
* **Repo:** https://github.com/yumna-09/FlyRank-ML-Internship
* **Date:** 2026-08-25

## 0. Abstract

Which content pages should a reviewer check first when a site has far more pages than a team can review by hand?

Using FlyRank's anonymized 30,000-page starter sample, I built a transparent rule-based baseline and used KMeans clustering to group pages by observed search behavior across five signals: impressions, click-through rate, average ranking position, engagement rate, and content age.

The first clustering evaluation was leaky because centroids were fit and scored on the same data, producing a silhouette score of 0.3763. To test whether the archetypes generalized beyond the clients used for fitting, I held out entire clients rather than random rows. The honest client-held-out silhouette score was 0.3046, compared with 0.0013 for the rule-based baseline's buckets.

The resulting behavioral archetypes were translated into a four-action review queue: `refresh`, `boost`, `prune`, and `monitor`. Every recommendation is decision support for human review, not an automated instruction or causal claim about traffic outcomes.

---

## 1. Problem framing

**Decision supported:** which pages a content strategist or reviewer should open first when the backlog is too large to inspect page-by-page.

**Unit of analysis:** one content page (`content_id`), represented by observed metrics aggregated over a trailing 90-day window.

**Output:** behavioral cluster membership and a priority action: `refresh`, `boost`, `prune`, or `monitor`, with supporting reason codes and human-review flags.

**Action a human takes:** review the highest-priority pages first, inspect the live page and its observed signals, then decide whether the recommended action is appropriate.

**Cost of a wrong call:** a page incorrectly treated as healthy may continue losing visibility unnoticed, while a page incorrectly prioritized consumes editorial review time.

Why clustering helps here: hand-written rules can identify individual signals, such as pages with unusually low CTR for their ranking position. However, real pages can have combinations of behavior — for example, high impressions with weak CTR, low impressions with strong engagement, or aging content with stable rankings.

Instead of evaluating one threshold at a time, this project asks whether **grouping pages by several observed signals simultaneously** can provide a clearer starting point for human review.

This is not a system for predicting Google's ranking algorithm or automatically changing pages. It is a behavioral grouping and prioritization system.

---

## 2. Data & safety

**Source:** FlyRank ML Internship warehouse release, build id `flyrank_pseudonymized_warehouse_release_v20260703`, exported 2026-07-03 from FlyRank's pseudonymized production search-and-analytics warehouse.

The broader production catalog contains:

* **78.8M** rows in the daily performance fact table
* **519,606** distinct content items
* **104** clients

The main clustering and playbook pipeline uses the bundled anonymized starter sample:

* **30,000 rows**
* one row per content item
* metrics pre-aggregated over a trailing 90-day window

A separate data-contract exercise used a March 2026 warehouse slice containing:

* **9,841,378 rows**
* **331,437 content items**
* **55 clients**

That exercise was used to verify data grain and test a feature-leakage trap. It was not the main modeling pipeline.

### Features used for clustering

The KMeans model uses five observed metrics:

* `impressions_90d`
* click-through rate
* average ranking position
* engagement rate
* content age

All features were standardized before clustering so that larger numerical scales did not dominate the distance calculations.

### Deliberately excluded

* GA4 columns where availability was not confirmed
* FlyRank product decision flags such as health score, priority score, or action type
* `trend_direction`
* `trend_pct`
* `content_id`
* `client_id`

`content_id` and `client_id` are used only as identifiers and grouping keys, never as clustering features.

`trend_direction` and `trend_pct` are excluded from the clustering feature set because they relate to the decline label and are used only afterward for descriptive analysis and action logic.

No client names, domains, URLs, page titles, or raw search queries appear in the analysis.

---

## 3. Baseline

Before fitting KMeans, I built a transparent rule-based baseline with no learned weights.

Two candidate relationships were checked against the data before being included in the baseline logic.

### Signal check 1 — Staleness vs. decline

**Verdict: MIXED**

The decline rate rose from approximately **51.1%** in the youngest freshness bucket to **61.1%** across later freshness buckets, but reversed in the oldest bucket:

* 0–30 days: 0.511, `n=20,480`
* 91–180 days: 0.611, `n=9,171`
* 181+ days: 0.471, `n=174`

The oldest bucket fell below the dataset average decline rate of approximately 0.542.

Because the relationship did not remain consistent across the observed range, staleness was excluded from the baseline score.

### Signal check 2 — CTR vs. ranking position

**Verdict: CONFIRMED**

Median CTR generally declines as ranking position becomes weaker:

* `top_3`: 0.20
* `page_1`: 0.24
* `striking`: 0.17
* `page_3_5`: 0.09
* `deep`: 0.00

This check was limited to visible pages with:

```text
impressions_90d >= 500
```

The observed relationship supported using a position-adjusted CTR gap as the core signal in the baseline.

### Baseline rule

**Reason code:**

```text
ctr_below_position_expected
```

**Score formula:**

```text
eligible * visibility_score * ctr_gap_norm
```

**Eligibility:**

```text
impressions_90d >= 500
AND avg_position > 0
AND position_tier != 'no_data'
```

### Baseline action thresholds

| Action                  | Score condition |
| ----------------------- | --------------: |
| `prioritize_ctr_review` |          >= 0.6 |
| `review_ctr`            |   > 0 and < 0.6 |
| `monitor`               |            == 0 |

### Baseline queue statistics

| Action                  |  Pages |
| ----------------------- | -----: |
| `monitor`               | 13,274 |
| `review_ctr`            | 12,814 |
| `prioritize_ctr_review` |  3,912 |

Additional baseline statistics:

* **Rows:** 30,000
* **Median score:** 0.2075
* **Maximum score:** 0.9909
* **Top-10 declining rate:** 0.70

The baseline score uses only:

```text
impressions_90d
avg_position
position_tier
ctr
```

The decline-label source is not used as a score input, and FlyRank product flags are not present as model features.

---

## 4. Model / analysis

**Method:** KMeans clustering.

This project uses an unsupervised approach because the primary goal is not to predict a target variable. The goal is to identify groups of pages that exhibit similar observed search behavior.

### Feature set

Five standardized observed features:

* impressions
* click-through rate
* average ranking position
* engagement rate
* content age

### Choosing the number of clusters

KMeans was evaluated across:

```text
k = 2 through k = 7
```

The silhouette search selected:

```text
k = 7
```

The same cluster count also won when the search was repeated using only training-client data.

### Why KMeans

KMeans was chosen because its output — cluster membership — can be translated into understandable behavioral archetypes for a non-technical reviewer.

The clusters are not semantic categories and do not use page text. They represent similarity across the five observed performance and age signals.

---

## 5. Evaluation & validation

### Baseline vs. KMeans

The rule-based baseline and KMeans were compared in the same feature space using silhouette score as a measure of how naturally separated the resulting groups are.

| Method                                 | Silhouette score | Reading                                    |
| -------------------------------------- | ---------------: | ------------------------------------------ |
| Baseline: rule-based buckets           |       **0.0013** | Essentially no natural separation          |
| KMeans (`k=7`), all-data fit + score   |           0.3763 | Strong, but inflated by evaluation leakage |
| KMeans (`k=7`), honest client-held-out |       **0.3046** | Trustworthy evaluation                     |

### Client-held-out validation

The first clustering evaluation fit the centroids and scored the same 30,000 rows. That does not test whether the discovered structure generalizes to unseen clients.

The validation was redesigned by grouping on `client_id`:

* approximately 80% of clients used for fitting
* approximately 20% of clients held out
* centroids fit only on training-client rows
* held-out clients scored without being used to fit the centroids

The held-out evaluation used:

* **7 held-out clients**
* **7,611 pages**

The silhouette score dropped from **0.3763** to **0.3046**.

The drop is expected when clusters are evaluated on clients the model did not see during fitting. The held-out score is therefore treated as the trustworthy evaluation rather than the higher all-data score.

### Leakage checks

Three checks were performed:

1. **k-selection leakage:** the search for the best value of `k` was repeated using only training-client data. `k=7` remained the selected value.

2. **Feature leakage:** identifiers and decline-label source columns were excluded from the clustering feature list.

3. **Separate leakage trap:** a data-contract exercise showed that constructing a feature from the same time window as a toy label could inflate accuracy from **0.591** to **1.000**. That pattern was excluded from this project's clustering features.

---

## 6. Cluster interpretation

The seven clusters were profiled descriptively across all 30,000 pages.

The resulting archetypes include:

* a small cluster of very high-traffic top performers, averaging approximately **112k impressions per 90 days**
* two larger groups of steady, older content that differ primarily by age
* a low-CTR, poorly ranked cluster with an average ranking position of approximately **46**
* a near-zero-impression, high-CTR group that may reflect low-volume or highly targeted queries
* an unusually high-engagement group
* a borderline group with weaker cluster fit

Approximately **2.6%** of points were poorly matched to their assigned cluster. These observations were concentrated in a borderline-engagement group positioned between clearer behavioral archetypes.

### Decline rate by cluster

Decline rate was measured descriptively after clustering and was **not used as a clustering feature**.

Across the held-out clients, observed decline rates ranged from:

```text
14.6% to 61.9%
```

Because this range was measured on only seven held-out clients, it is treated as descriptive evidence rather than proof that the same pattern generalizes to the full client population.

---

## 7. Recommendation

The clustering and observed performance signals are translated into a four-action priority queue.

Every page receives one of the following actions:

| Priority | Action    | Trigger                                                           |  Pages |
| -------- | --------- | ----------------------------------------------------------------- | -----: |
| 1        | `REFRESH` | ranks `top_3` / `page_1`, declining >20% MoM, CTR well below tier |  1,447 |
| 2        | `BOOST`   | position 11–20 ("striking distance"), CTR below expectation       |  4,348 |
| 3        | `PRUNE`   | deep/unranked, declining, bottom-quartile visibility              |    405 |
| 4        | `MONITOR` | no strong signal either way                                       | 23,800 |

### Human review requirement

**23.4%** of the queue is flagged for mandatory human sign-off.

This includes:

* every `PRUNE` candidate
* top-10%-visibility pages
* pages in an uncertain score band
* pages with weak cluster fit

### What should never be automated

The system must not automatically perform:

* URL deletion
* URL redirection without human verification
* changes to core brand or legal pages
* site-navigation changes
* URL-structure changes based solely on the model or playbook output

The action queue is a prioritization tool. A recommendation means that the page is worth human inspection; it does not mean that the action is automatically correct.

---

## 8. Limitations & honest framing

**Cross-sectional, not causal.** No page was actually refreshed and re-measured. The results do not show that following a recommendation will improve traffic.

**Not "predicting Google."** The clusters describe similarity across five observed metrics. They do not model Google's ranking algorithm.

**Not semantic clustering.** No page text or semantic content representation was used.

**Small held-out sample for decline-rate analysis.** The observed 14.6%–61.9% range comes from seven held-out clients and should not be generalized without further validation.

**Starter sample, not the full catalog.** The main clustering pipeline uses 30,000 pages out of the broader 519,606-content-item catalog.

**Baseline was not re-tested under the same held-out evaluation.** The client-held-out validation specifically applies to KMeans.

**Single snapshot rather than a long time series.** The analysis uses observed windows rather than repeated intervention and outcome measurement.

**Playbook thresholds are simple, un-tuned cutoffs.** They provide an initial queue design and have not yet been validated against real editor decisions or downstream business outcomes.

---

## 9. Reproducibility

Everything in the project can be re-run from the repository.

**Random seed:**

```text
42
```

### Relevant notebooks

* `work/notebooks/capstone.ipynb` — full capstone pipeline
* `work/notebooks/w05_model.ipynb` — KMeans model and cluster interpretation
* `work/notebooks/w06_validation_audit.ipynb` — client-grouped validation and leakage audit
* `work/notebooks/w07_action_playbook.ipynb` — ranked action playbook

### Environment

```bash
pip install -r requirements.txt
```

Then open the notebooks inside:

```text
work/notebooks/
```

and run them in sequence.

Repository:

https://github.com/yumna-09/FlyRank-ML-Internship

---

## 10. Acknowledgments & data credit

Built on the **FlyRank ML Internship dataset** — real, pseudonymized production search and analytics data provided for the internship track.

https://flyrank.ai

---

> **Claims checklist before submitting:** observed / measured / directional / decision-support language throughout · no causal claims without an experiment or causal design · no claim of predicting Google's algorithm · no client-identifying details · KMeans results and action counts match the executed project notebooks.
