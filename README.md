# Content Archetype Clustering — FlyRank ML Capstone

- **Author:** Yumna Kashif
- **Program:** FlyRank ML Internship
- **Track:** Applied Search Intelligence
- **Capstone Lane:** Structured Content Archetype Clustering

> **30,000 content pages → 7 behavioral archetypes → ranked human-review actions**

## 🔗 Project Links

* 🌐 **[Live Research Paper](https://yumna-09.github.io/FlyRank-ML-Internship/)**
* 💻 **[GitHub Repository](https://github.com/yumna-09/FlyRank-ML-Internship)**
* 📓 **[Capstone Notebook](https://github.com/yumna-09/FlyRank-ML-Internship/blob/main/work/notebooks/capstone.ipynb)**
* 📄 **[Full Capstone Report](https://github.com/yumna-09/FlyRank-ML-Internship/blob/main/work/CAPSTONE_REPORT.md)**
* 📥 **[Download Ranked Action Queue](https://github.com/yumna-09/FlyRank-ML-Internship/releases)**

---

## Overview

When a website contains thousands of pages, manually deciding which pages deserve attention first becomes a prioritization problem.

This capstone analyzes an anonymized dataset of **30,000 content pages** and groups them into behavioral archetypes using **KMeans clustering**. Instead of evaluating individual signals in isolation, the model considers combinations of search-performance and engagement signals to identify pages with similar behavior.

The resulting clusters are translated into a practical, human-review action queue with four possible recommendations:

* **Refresh**
* **Boost**
* **Prune**
* **Monitor**

The project is designed as a **decision-support workflow** for content strategists and reviewers. It does **not** attempt to predict Google's ranking algorithm or automatically make content decisions.

---

## The Problem

The question guiding this project was:

> **Which content pages should a strategist review first when the backlog is too large to inspect manually?**

A rule-based approach can identify isolated signals such as low CTR or weak rankings. However, content performance is often influenced by combinations of signals.

For example, a page may have:

* High impressions but weak CTR
* Low impressions but strong engagement
* Aging content with stable rankings

Clustering makes it possible to group pages according to these combined behavioral patterns and turn the results into a more structured review process.

---

## Who Is This For?

This project is intended for:

* Content strategists
* SEO and search-intelligence teams
* Website owners managing large content inventories
* Analysts who need to prioritize manual content review

The output is intended to **support human decision-making rather than replace it**.

---

# Results at a Glance

| Result                            |      Value |
| --------------------------------- | ---------: |
| Content pages analyzed            | **30,000** |
| Features used for clustering      |      **5** |
| Selected number of clusters       |      **7** |
| Rule-based baseline silhouette    | **0.0013** |
| Initial KMeans silhouette         | **0.3763** |
| Client-held-out KMeans silhouette | **0.3046** |
| Final action categories           |      **4** |

The initial KMeans evaluation produced a silhouette score of **0.3763**.

During validation, an evaluation-leakage issue was identified because the cluster centroids were fitted and evaluated on the same data.

To address this, the evaluation was redesigned using a **client-held-out validation approach**, where entire clients were excluded during model fitting and used for evaluation.

The resulting held-out silhouette score was **0.3046**.

Although this score is lower than the original result, it provides a more honest estimate of how well the clustering structure generalizes to unseen clients.

---

# Data

The project uses an anonymized FlyRank internship dataset containing **30,000 content pages**.

The analysis uses a trailing **90-day aggregation window**.

## Features Used

The five features used for clustering are:

* Impressions
* Click-through rate (CTR)
* Average ranking position
* Engagement rate
* Content age

All features were standardized before applying KMeans clustering.

## Deliberately Excluded

The following fields were excluded to reduce leakage and avoid using internal priority information:

* `trend_direction`
* `trend_pct`
* `content_id`
* `client_id`
* FlyRank health scores
* FlyRank priority scores
* Product action flags

No client names, domains, URLs, page titles, or search queries were used in the analysis.

---

# Project Workflow

```text
Anonymized Content Data
          │
          ▼
Data Cleaning & Feature Selection
          │
          ▼
Feature Standardization
          │
          ▼
Rule-Based Baseline
          │
          ▼
KMeans Clustering
          │
          ▼
Cluster Evaluation & Validation
          │
          ▼
Behavioral Archetype Interpretation
          │
          ▼
Human-Review Action Mapping
          │
          ▼
Ranked Action Queue
```

---

# Architecture and Design

The workflow follows a simple decision-support pipeline:

1. **Prepare the data** by selecting relevant observable performance signals.
2. **Create a transparent baseline** to establish a simple comparison point.
3. **Standardize the selected features** so that no individual metric dominates clustering because of scale.
4. **Test multiple values of K** from 2 through 7.
5. **Select K = 7** for the final clustering approach.
6. **Validate the clustering structure** using a client-held-out evaluation approach.
7. **Interpret the clusters** as behavioral archetypes.
8. **Translate archetypes into recommended actions** for human review.
9. **Produce a ranked action queue** to help reviewers decide where to start.

---

# Why KMeans?

KMeans was selected because it provides a relatively understandable way to group pages with similar observed behavior.

The model uses multiple signals simultaneously rather than applying one fixed rule at a time.

The goal is not to discover a universal or permanent taxonomy of content. Instead, the clusters provide a structured way to explore behavioral patterns within the available dataset.

---

# Setup

## Option 1 — Run the Capstone Notebook

The main capstone analysis is located at:

```text
work/notebooks/capstone.ipynb
```

Open the notebook directly:

📓 **[Open Capstone Notebook](https://github.com/yumna-09/FlyRank-ML-Internship/blob/main/work/notebooks/capstone.ipynb)**

The notebook contains the analysis workflow, clustering approach, validation, and action-playbook generation.

You can run it using Google Colab or a local Jupyter environment.

## Option 2 — Run Locally

Clone the repository:

```bash
git clone https://github.com/yumna-09/FlyRank-ML-Internship.git
```

Move into the project directory:

```bash
cd FlyRank-ML-Internship
```

Install the required dependencies:

```bash
pip install -r requirements.txt
```

---

# Usage

The intended workflow is:

1. Open the capstone notebook.
2. Load the approved anonymized dataset.
3. Select the five clustering features.
4. Standardize the features.
5. Run KMeans across the candidate values of K.
6. Evaluate the resulting clusters.
7. Validate the selected approach using the client-held-out methodology.
8. Review the resulting behavioral archetypes.
9. Generate the recommended human-review actions.
10. Inspect the ranked action queue.

The output should be interpreted as a **prioritization aid**.

A human reviewer should inspect the page and its current context before taking actions such as refreshing, boosting, pruning, or monitoring content.

---

# Evaluation and Validation

## Rule-Based Baseline

Before fitting KMeans, a transparent rule-based baseline was created using observable search-performance signals.

**Silhouette score: `0.0013`**

## Initial KMeans Evaluation

The first KMeans evaluation produced:

**Silhouette score: `0.3763`**

However, this result was not used as the final validation result because the clustering centroids were fitted and evaluated on the same data.

This created an evaluation-leakage issue.

## V2 — Client-Held-Out Validation

The validation design was revised so that entire clients were held out during model fitting.

The model was fitted without those clients and then evaluated on the held-out data.

The resulting silhouette score was:

**`0.3046`**

| Method                                  | Silhouette Score |
| --------------------------------------- | ---------------: |
| Rule-Based Baseline                     |           0.0013 |
| KMeans — Fit and Evaluated on All Data  |           0.3763 |
| **KMeans — Client-Held-Out Validation** |       **0.3046** |

## What Changed in V2?

The most important change was **not increasing the score**. It was improving the validity of the evaluation.

The original evaluation looked stronger, but it measured clustering structure on data that had already influenced the fitted centroids.

The V2 approach produced a lower score, but it provides a more credible estimate of how the approach behaves on unseen clients.

> **An honest evaluation was prioritized over a better-looking metric.**

---

# Action Playbook

The behavioral archetypes are translated into four broad action categories.

## 🔄 Refresh

Pages that may benefit from content review or improvement.

## 🚀 Boost

Pages that show useful potential and may benefit from additional optimization or visibility efforts.

## ✂️ Prune

Pages that should be flagged for careful human review to determine whether they continue to provide sufficient value.

## 👀 Monitor

Pages that do not currently require immediate action but should remain under observation.

These actions are **recommendations for human review, not automated instructions**.

📥 **[Download the Ranked Action Queue](https://github.com/yumna-09/FlyRank-ML-Internship/releases)**

---

# Project Structure

```text
FlyRank-ML-Internship/
│
├── work/
│   ├── notebooks/
│   │   └── capstone.ipynb
│   │
│   └── CAPSTONE_REPORT.md
│
├── docs/
│   └── index.html
│
├── outputs/
│   └── project outputs and generated artifacts
│
├── data/
│   └── approved anonymized data resources
│
├── requirements.txt
│
└── README.md
```

---

# Live Project

The capstone is presented as a deployed research paper through GitHub Pages.

🌐 **[View the Live Research Paper](https://yumna-09.github.io/FlyRank-ML-Internship/)**

The live report presents:

* The research question
* The clustering approach
* Evaluation results
* Behavioral archetypes
* Action recommendations
* Project limitations

For the complete technical analysis:

📄 **[Read the Full Capstone Report](https://github.com/yumna-09/FlyRank-ML-Internship/blob/main/work/CAPSTONE_REPORT.md)**

📓 **[Explore the Capstone Notebook](https://github.com/yumna-09/FlyRank-ML-Internship/blob/main/work/notebooks/capstone.ipynb)**

---

# Limitations

This project has several important limitations.

## 1. Results Depend on the Available Data

The discovered archetypes depend on the dataset, selected features, and observation window used in this analysis.

Different datasets or additional features could produce different cluster structures.

## 2. Clusters Are Not Ground-Truth Labels

The seven clusters are analytical groupings based on observed behavior.

They should not be treated as universal or permanent categories of content.

## 3. Recommendations Require Human Review

An action such as **Refresh** or **Prune** should not be automatically executed.

A human reviewer should inspect the actual page, business context, content quality, and other relevant information before making a decision.

## 4. Silhouette Score Does Not Measure Business Impact

The evaluation measures clustering structure.

It does not prove that following the recommended actions will cause traffic, rankings, or business outcomes to improve.

## 5. This Is Not a Model of Google's Algorithm

The project does not predict Google's ranking algorithm or make causal claims about why a page performs in a particular way.

All findings should be interpreted as:

> **Observed · Measured · Directional · Decision-Support**

---

# Key Design Decision

A major design decision was to convert clustering results into a **ranked human-review action queue** rather than stopping at cluster labels.

A clustering result alone can be difficult for a non-technical stakeholder to act on.

Mapping behavioral archetypes into recommended review actions makes the analysis more operational while keeping a human in the decision loop.

---

# AI Transparency

AI tools were used during this project to assist with:

* Brainstorming
* Coding support
* Debugging
* Documentation
* Iteration

I reviewed the generated suggestions, checked the analysis workflow and outputs, identified and corrected the evaluation-leakage issue, and made the final decisions about the project's methodology, validation approach, interpretation, and presentation.

AI assisted the development process, but the final project required **human review and validation of the results**.

---

# Repository

This repository contains the work completed throughout the **FlyRank ML Internship**, including weekly assignment notebooks, supporting documentation, and the final capstone.

The capstone represents the final application of that work:

```text
Problem Framing
        ↓
Data Understanding
        ↓
Leakage Checking
        ↓
Signal Auditing
        ↓
Baseline Design
        ↓
Modeling
        ↓
Validation
        ↓
Action Playbook
        ↓
Capstone
```

---

# Acknowledgments

This project was completed as the capstone for the **FlyRank ML Internship — Applied Search Intelligence track**.

The project builds on the internship's technical foundation and uses the approved anonymized data and learning framework provided for the program.

Thank you to **FlyRank** and the internship team for the learning environment, technical resources, and guidance that supported this project.

The analysis, evaluation, interpretation, and final capstone presentation represent my work completed during the internship.
