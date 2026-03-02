---
title: "Methodology"
permalink: /methodology.html
---

This page explains how we collect, process, and analyze artifact evaluation data, including detailed calculation formulas for all metrics displayed on this site.

## Table of Contents
- [Conferences Covered](#conferences-covered)
- [Data Collection](#data-collection)
- [Pipeline](#pipeline)
- [Author Metrics](#author-metrics) — how AR%, RR%, and other author statistics are calculated
- [Institution Metrics](#institution-metrics) — how institutional data is aggregated
- [Badge Definitions](#badge-definitions) — what each badge type means
- [Weighted Scoring](#weighted-scoring-combined-rankings) — how the combined score is computed
- [Repository Statistics](#repository-statistics) — GitHub/Zenodo metrics
- [Data Format](#data-format)
- [Contributing](#contributing)

## Conferences Covered

Data is collected from conferences tracked by [sysartifacts](https://sysartifacts.github.io) and [secartifacts](https://secartifacts.github.io):

- **Systems**: {{ site.data.summary.systems_conferences | join: ", " }}
- **Security**: {{ site.data.summary.security_conferences | join: ", " }}

## Data Collection

We scrape artifact evaluation results from sysartifacts/secartifacts websites, extract paper titles, authors, badges (Available, Functional, Reproducible, Reusable) and repository URLs. For USENIX conferences (ATC, FAST) we also scrape badge data from technical session pages. AE committee data is gathered from sysartifacts/secartifacts plus direct scraping (USENIX, CHES, PETS websites).

Repository statistics (GitHub stars/forks, Zenodo/Figshare downloads) are collected via their public APIs. Author names are matched to [DBLP](https://dblp.org) for disambiguation and total-publication counts. Author affiliations are enriched using DBLP person pages and [CSRankings](http://csrankings.org) faculty data.

All scripts are in the [artifact_analysis](https://github.com/researchartifacts/artifact_analysis) repository.

## Pipeline

The pipeline ([run_pipeline.sh](https://github.com/researchartifacts/artifact_analysis/blob/main/run_pipeline.sh)) runs monthly via GitHub Actions:

1. **Scrape artifact results** from sysartifacts/secartifacts
2. **Match papers to DBLP authors** and extract author affiliations
3. **Filter papers by AE-active years** — only count papers from years when venues had artifact evaluation
4. **Collect repository statistics** (GitHub/GitLab stars/forks, Zenodo downloads)
5. **Compute combined rankings** with weighted scoring
6. **Aggregate institution statistics** by summing across affiliated authors
7. **Generate area-specific rankings** (systems, security, overall)
8. **Export data** (JSON/YAML) and charts to this website

The complete pipeline takes ~30 minutes to run and processes the DBLP XML database (~3GB compressed) to match ~2,500+ artifact papers to author records and compute total paper counts.

## Author Metrics

Individual author statistics are computed by matching artifact papers to DBLP records. Each metric is calculated as follows:

### Artifacts
The total number of evaluated artifacts (papers with at least one badge) authored by this person across all tracked conferences.

### Total Papers
The total number of papers this author published at tracked conferences, **counting only years when that conference was conducting artifact evaluation**. For example:
- If ACSAC started AE in 2017, only papers from 2017–present are counted
- If an author published at ACSAC in 2010–2024, only 2017–2024 papers contribute to the denominator
- This prevents artificial deflation of Artifact Rate by excluding pre-AE papers

The paper count is determined by matching author names to DBLP records and filtering by conference and year.

### Artifact Rate (AR%)
The percentage of an author's papers (at AE-active conferences) that have artifact badges:

```
AR% = (Artifacts / Total Papers) × 100
```

**Key point:** The denominator includes only papers from years when the venue had artifact evaluation. This ensures the rate reflects artifact adoption within the relevant time window, avoiding both over-inflation (counting only artifact papers) and under-inflation (counting all historical publications).

**Cross-area handling:** For authors active in both systems and security, contributions are **summed**. If an author has 10 systems papers and 5 security papers (all in AE-active years), the denominator is 15. This additive approach is correct because systems and security conferences are disjoint publication venues.

### Reproducibility Rate (RR%)
Among papers with artifacts, the percentage achieving the highest-tier badge (Reproduced or Reusable):

```
RR% = (Reproduced badges / Total artifacts) × 100
```

This measures the depth of reproducibility beyond mere artifact availability.

### AE Memberships
The number of times this author served on an artifact evaluation committee across all tracked conferences.

### Chair Count
The number of times this author served as an AE chair or co-chair.

### Combined Score
A composite metric combining artifact production and AE service (see [Weighted Scoring](#weighted-scoring-combined-rankings) below).

---

## Institution Metrics

Institution-level statistics aggregate contributions from all authors affiliated with that institution. Affiliations are determined from DBLP person pages and CSRankings faculty data.

### How Institution Data is Aggregated

All metrics are **summed across affiliated authors**:

- **Artifacts**: Total artifacts from all affiliated authors
- **Total Papers**: Total papers from all affiliated authors (AE-active years only)
- **AE Memberships**: Total committee memberships from all affiliated authors
- **Combined Score**: Sum of all affiliated authors' combined scores

**Artifact Rate and Reproducibility Rate** are then computed from these aggregated totals:

```
Institution AR% = (Total artifacts / Total papers) × 100
Institution RR% = (Total reproduced badges / Total artifacts) × 100
```

### Cross-Area Aggregation

For institution rankings broken down by area (systems vs. security):

- **Systems rankings**: Include only artifacts, papers, and AE service from systems conferences
- **Security rankings**: Include only artifacts, papers, and AE service from security conferences
- **Overall rankings**: Sum of systems + security contributions

When an author appears in both areas, their contributions are **summed** in the overall rankings. For example, if an author has 5 systems artifacts and 3 security artifacts, the institution's overall count includes all 8.

This ensures:
- Overall institution scores ≥ systems-only scores
- Overall institution scores ≥ security-only scores
- No double-counting (each artifact/paper counted exactly once)

---

## Badge Definitions

Artifact evaluation committees award badges to recognize different levels of reproducibility:

### Available
The artifact is publicly accessible via a persistent repository (e.g., GitHub, Zenodo, Figshare). Basic documentation is provided.

### Functional
The artifact is not only available but also documented, complete, and exercisable. Evaluators can successfully build/install and run the artifact's main functionality.

### Reproduced / Reusable
**Reproduced** (security conferences): Evaluators successfully reproduced the main results/claims from the paper using the artifact.

**Reusable** (systems conferences): The artifact is well-documented, modular, and designed for reuse beyond reproducing paper results. May include extensibility features, clean APIs, or generalizability to other use cases.

Both badges represent the highest tier, indicating the artifact went beyond basic functionality to either validate the paper's claims or enable broader community reuse.

---

## Weighted Scoring (Combined Rankings)

The combined score recognizes both artifact production and evaluation service:

```
Combined Score = Artifact Score + AE Service Score
```

### Artifact Score Calculation

For each artifact, badges are scored **additively** (not hierarchically):

| Badge Type | Points |
|---|---|
| Available | +1 |
| Functional | +1 |
| Reproduced/Reusable | +1 |

**Maximum per artifact: 3 points** (e.g., an artifact with all three badges earns 3 points)

**Example:** An author with 2 artifacts:
- Artifact 1: Available + Functional → 2 points
- Artifact 2: Available + Functional + Reproduced → 3 points
- **Total artifact score: 5**

### AE Service Score Calculation

| Role | Points |
|---|---|
| Committee member | 3 points per membership |
| Committee chair | 5 points (3 base + 2 bonus) |

**Example:** An author who served as:
- AE member 2 times → 2 × 3 = 6 points
- AE chair 1 time → 1 × 5 = 5 points
- **Total AE service score: 11**

### Minimum Score Threshold

Only individuals and institutions with a **combined score ≥ 3** are included in rankings. This threshold ensures meaningful participation (at least one reproduced artifact, OR one AE membership, OR equivalent contributions).

### Why These Weights?

- **Additive badge scoring** reflects that each badge level requires distinct effort (making code available, ensuring it builds/runs, reproducing results)
- **AE membership = 3 points** recognizes the substantial time investment (~50 hours per evaluation cycle)
- **Chair bonus = +2** acknowledges additional coordination and leadership responsibilities
- The formula balances recognition between artifact producers and evaluators, countering the traditional invisibility of evaluation labor in academic metrics

---

## Repository Statistics

For artifacts with GitHub/GitLab repositories or Zenodo/Figshare archives, we collect engagement metrics as supplementary signals of community uptake:

### GitHub/GitLab Metrics
- **Stars**: Number of users who starred the repository
- **Forks**: Number of times the repository was forked
- **Watchers**: Number of users watching the repository for updates

### Zenodo/Figshare Metrics
- **Downloads**: Total download count from the archive platform
- **Views**: Number of views/visits to the artifact page

**Important notes:**
- Repository statistics are **displayed separately** and do not contribute to the combined score
- These metrics reflect external reuse signals but are subject to biases:
  - Age effects (older artifacts accumulate more stars)
  - Repository type differences (libraries vs. experiment code)
  - Discovery algorithm effects (GitHub trending, recommendation systems)
- We report these as observational data, not as quality judgments

---

## Data Format

All data is available in JSON format under `/assets/data/`. YAML summaries live in `_data/` for Jekyll templating.

## Contributing

Report data errors via [GitHub issues](https://github.com/researchartifacts/artifact_analysis/issues). Contributions welcome — see the [Contribute]({{ '/about.html' | relative_url }}) page.

---

*Last updated: {{ site.data.summary.last_updated | default: "Pending initial run" }}*
