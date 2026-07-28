# Revolut Public Data Analysis (Independent Project)

An independent, unofficial analysis of Revolut's publicly available data — combining
financial statement analysis, customer review sentiment analysis, and unsupervised machine
learning to explore how the business has grown and where specific customer experience gaps
exist.

**Disclaimer:** This project is not affiliated with, endorsed by, or produced in partnership
with Revolut. All data used is publicly available — audited annual reports filed with UK
Companies House, and public Google Play Store reviews. No private, internal, or confidential
information was used at any point.

---

## Overview

This project set out to answer two connected questions:

1. **How has Revolut grown financially, and how efficiently?**
2. **What are real customers actually experiencing, and does it align with the growth story?**

Rather than treat these as separate exercises, the project deliberately cross-references
financial/operational data (staffing, revenue, lending) against customer sentiment data, to
see whether operational decisions show up in how customers describe their experience.

---

## Data Sources

- **Financial data:** Revolut Group Holdings Ltd (Companies House company no. 12743269) —
  audited annual report PDFs, 2021–2025, downloaded via the Companies House public API and
  extracted via OCR (the filings are image-based, not text-searchable PDFs).
- **Customer review data:** ~290,000 Google Play Store reviews for the Revolut Android app,
  collected via public review APIs.

---

## Methodology

- **Data cleaning:** Identified and removed a CSV parsing corruption issue affecting ~5.9%
  of scraped review rows (unescaped commas in reply text caused column misalignment);
  documented and excluded via validation on the `rating` field.
- **Sentiment analysis:** Initially used VADER (lexicon-based sentiment scoring). Benchmarked
  against a transformer model (DistilBERT) using star rating as ground truth — VADER
  correctly identified only 57.2% of confirmed 1-star reviews as negative, versus 95.9% for
  the transformer model. The transformer model was adopted as the more reliable method going
  forward, and this benchmarking exercise is documented as a methodological finding in its
  own right.
- **Theme analysis:** A manual keyword-based tagging system (account freezes, customer
  support, fees, verification, app stability, transfers) was used for structured theme
  counting.
- **Unsupervised topic modeling (BERTopic):** Used to discover complaint patterns not
  captured by the predefined keyword list. Default settings proved unreliable —
  reproducibility testing across multiple runs showed topic composition varied significantly
  depending on random sampling, a known limitation of density-based clustering on datasets
  with a strong positive skew (~88% positive reviews). Lowering the clustering threshold and
  sorting clusters by actual average star rating (rather than trusting auto-generated
  keyword labels, which were often generic) was necessary to reliably surface genuine
  negative clusters.
- **Predictive modeling:** A Random Forest classifier was trained to predict 1-star reviews
  from sentiment score, review length, and theme flags, to rank which specific complaint
  themes carry the most independent predictive weight.

---

## Key Findings

<img width="1326" height="743" alt="image" src="https://github.com/user-attachments/assets/b21032ad-54a6-484e-9000-ba85c8b66ca0" />

### Financial Performance — Efficient, Maturing Growth
- Revenue grew from £637.9M (2021) to £4.52bn (2025); net margin expanded from 0.6% (2022)
  to 28.9% (2025), driven primarily by administrative expense ratio falling from 72.3% to
  40.0% — evidence of genuine operating efficiency, not just top-line growth.
- 2022 recorded a real pre-tax operating loss (-£25.4M), coinciding with a corporate
  restructuring event; a one-off tax credit pushed net profit narrowly positive that year.
- Revenue growth is decelerating (94.9% → 71.3% → 46.6% year-on-year) even as profitability
  accelerates — consistent with a business transitioning from hypergrowth to maturity.

<img width="1322" height="736" alt="image" src="https://github.com/user-attachments/assets/68853259-3477-4e91-ab58-0bf1ec3c6d24" />

### Revenue Mix & Lending
- Cards & interchange overtook FX/Wealth as the largest fee income source between 2021 and
  2022; subscriptions held a stable ~20% share of fee income across all five years.
- The loan book grew 123x from 2021 (£18.2M) to 2025 (£2.24bn). Mortgages, a near-zero
  product in 2024, grew to £87.6M in 2025. Loan impairment rates held broadly steady
  (~4% in 2024–2025) despite this rapid scaling, though an earlier spike (6.5% in 2023)
  suggests risk processes matured alongside the loan book rather than being static.

<img width="1323" height="743" alt="image" src="https://github.com/user-attachments/assets/4e7e28ac-314a-4524-8dbe-2142deb093b2" />

### People & Operations
- Total headcount grew from 2,365 (2021) to 10,909 (2025). Customer support's share of the
  overall workforce peaked at 50% in 2024, then fell to 43% in 2025 even as the user base
  grew roughly 30% that year — a real reduction in relative support investment.
- **Notably, this reduction in relative support staffing did not correspond to a rise in
  customer support complaint volume in the review data — complaint volume trends downward
  over the same period.** This is a genuinely interesting finding worth further
  investigation rather than a simple cause-and-effect story: possible explanations include
  improved self-service tooling, automation, or changes in the underlying causes of
  dissatisfaction shifting toward other themes (e.g. verification, fraud handling) rather
  than general support capacity. The data supports a more nuanced read than "less staff
  means more complaints" — it did not straightforwardly play out that way here.
  
<img width="1321" height="742" alt="image" src="https://github.com/user-attachments/assets/29714f6f-3f0b-4191-85e9-5a6a42c07f6d" />
<img width="1315" height="737" alt="image" src="https://github.com/user-attachments/assets/606fce60-2890-4f48-ab4d-3b577c12f493" />

### Six Distinct Complaint Categories (via ML Topic Modeling)
Beyond the general keyword-based themes, unsupervised topic modeling surfaced six specific,
independently distinguishable complaint categories, each pointing to a different
operational issue rather than one generic "poor support" problem:

| Category | Reviews | Avg. Rating | Company Reply Rate |
|---|---|---|---|
| Fraud / unauthorized transactions | 167 | 1.10 | 45.5% |
| Login & authentication failures | 153 | 1.23 | 62.7% |
| Identity verification failures | 131 | 1.26 | 54.2% |
| Scam allegations & failed recovery | 151 | 1.07 | 57.0% |
| Unexplained account blocks | 86 | 1.07 | 38.4% |
| **GrapheneOS compatibility** | 52 | 1.13 | **2.8%** |

*(Overall dataset average reply rate: 24.5%)*

---
<img width="1320" height="740" alt="image" src="https://github.com/user-attachments/assets/20c8be14-5c19-4694-9eb3-986c34a1c23f" />

## Spotlight: The GrapheneOS Compatibility Issue

This finding stood out as the most specific, rigorously verified, and independently
corroborated result in the entire project, so it received a dedicated dashboard page.

**What was found:** Since around December 2024, Revolut has enforced Google's Play Integrity
API, which blocks login for users running GrapheneOS — a privacy and security-focused custom
Android operating system. This was discovered through unsupervised topic modeling, not
through the original manually-defined keyword themes, since "GrapheneOS" was not a term
anticipated in advance. Printing and inspecting a sample of the actual review text contained
within this cluster revealed a notable pattern: several reviewers explicitly self-identified
as long-tenured, **Premium ("Metal" tier)** customers — for example, referencing having used
the app "for more than 5 years" or being "a metal customer" for years — indicating this issue
disproportionately affects Revolut's higher-value, higher-loyalty subscriber base rather than
casual free-tier users. This customer-tier detail was not something the model itself
detected, labeled, or quantified — the cluster's automated keyword summary gave no hint of
it. It only became apparent by examining the underlying review text the model had grouped
together, underscoring that cluster labels and keyword summaries alone were insufficient to
extract the full business-relevant detail here.

**Verification steps taken:**
- Confirmed 52 distinct customers, with zero repeat reviewers, explicitly referenced
  GrapheneOS or Play Integrity API-related terms in their reviews — ruling out a small
  handful of vocal repeat complainers as the source of the pattern.
- 91% of these reviews were 1-star.
- The complaint timeline (spiking sharply in December 2024, continuing through September
  2025) aligns precisely with independently published, public statements from GrapheneOS's
  official channels, which describe the block as a deliberate policy decision — not a
  compatibility bug — and report ongoing contact with the EU Commission regarding it as of
  September 2025.
- **Company reply rate to these complaints was just 2.8%** — roughly 1/9th of the 24.5%
  overall average reply rate across all complaint types — despite several affected customers
  identifying themselves as long-tenured, Premium ("Metal" tier) subscribers.

**Why this matters:** This is a small-volume but high-value, technically sophisticated, and
publicly vocal customer segment. The underlying fix (supporting GrapheneOS's published
attestation compatibility guide) is a comparatively low-cost technical change relative to the
reputational and regulatory exposure risk of continued neglect, particularly given the
existing EU Commission attention.

This case study demonstrates the practical value of unsupervised machine learning as a
complement to hypothesis-driven analysis — it surfaced a specific, real, and independently
verifiable business issue that a predefined keyword list would never have anticipated.

---

## Overall Business Assessment

Taken together, the data suggests a business executing a genuinely strong growth and
profitability story: margin expansion, disciplined lending growth, and revenue
diversification all point to a maturing, well-managed fintech. Customer sentiment overall
remains strongly positive (the large majority of reviews are 5 star). Rather than a broad
customer service crisis, the data points to **specific, addressable service gaps** —
particularly around fraud recovery support, identity verification flexibility, and
niche technical compatibility issues like GrapheneOS — where response and resolution appear
inconsistent relative to the severity and, in some cases, the value of the customers
affected. These are the kind of targeted, fixable issues that a growing fintech can address
without requiring a wholesale change in strategy.

---

## Tech Stack

- **Data extraction:** Python, `google-play-scraper`, Companies House REST API, Tesseract
  OCR, Poppler
- **Data processing:** pandas
- **NLP / ML:** VADER, HuggingFace Transformers (DistilBERT), BERTopic, scikit-learn
  (Random Forest)
- **Visualization:** Power BI (DAX measures, Power Query)

---

## Repository Structure

```
notebooks/       Jupyter notebooks (cleaning, sentiment analysis, ML modeling)
                  Note: raw data fetching / OCR extraction notebooks are excluded
                  from this repository.
Dashboard/        Power BI file
```

---

## Notes & Limitations

- **Review data is Android-only.** All review data was collected from the Google Play Store;
  no App Store (iOS) reviews are included. Findings on customer sentiment and specific
  complaint patterns — including the GrapheneOS case study, which is itself an Android-only
  issue by nature — should be read as representative of Android users' experience only, not
  the full customer base across both platforms.
- Review data covers a variable historical window; financial data covers audited years
  2021–2025 only. Where the two datasets don't share an overlapping time period, they are
  presented as separate analytical views rather than forced into a single misleading
  timeline.
- BERTopic results required a documented reproducibility investigation — this is disclosed
  transparently in the accompanying notebooks rather than presenting only the most favorable
  single run.
- This analysis relies entirely on public review text and public financial filings; it does
  not have access to Revolut's internal support tickets, resolution data, or customer
  acquisition/retention metrics, and conclusions should be read with that limitation in mind.
