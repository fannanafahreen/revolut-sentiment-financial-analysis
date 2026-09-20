# Revolut Public Data Analysis (Independent Project)

An independent, unofficial analysis of Revolut's publicly available data combining financial statement analysis, customer review sentiment analysis, and unsupervised machine learning to explore how the business has grown and where specific customer experience gaps exist.

**Disclaimer:** This project is not affiliated with, endorsed by, or produced in partnership with Revolut. All data used is publicly available audited annual reports filed with UK Companies House, and public Google Play Store reviews. No private, internal, or confidential information was used at any point.

![Python](https://img.shields.io/badge/Python-3776AB?style=flat&logo=python&logoColor=white)
![Transformers](https://img.shields.io/badge/HuggingFace-DistilBERT-FFD21E?style=flat)
![BERTopic](https://img.shields.io/badge/BERTopic-Unsupervised%20NLP-blue?style=flat)
![Power BI](https://img.shields.io/badge/Power%20BI-F2C811?style=flat&logo=powerbi&logoColor=black)

---

## Executive Summary

Most public-facing analyses of a fintech pick one lens — either the financials look strong, or the reviews are complained about — and stop there. This project set out to hold both at once and ask whether they actually agree with each other: **how has Revolut grown financially, and does that growth story line up with what real customers say they're experiencing?**

Two independent, publicly available data sources were combined to answer that: five years of audited annual reports (2021–2025), and roughly 290,000 Google Play Store reviews. Rather than presenting the financial story and the sentiment story as two separate write-ups, this project deliberately cross-references them — checking whether operational decisions (like staffing changes) actually show up in how customers describe their experience, rather than assuming they do.

---

## Headline Results

- Revenue grew from **£637.9M (2021)** to **£4.52bn (2025)**; net margin expanded from **0.6%** to **28.9%** over the same period
- Loan book scaled **123x** — from £18.2M (2021) to £2.24bn (2025) — while impairment rates held broadly steady
- Customer support's share of total headcount fell from **50% (2024)** to **43% (2025)**, even as the user base grew ~30%
- Benchmarked two sentiment methods on **~290,000 reviews**: a transformer model (DistilBERT) correctly identified **95.9%** of confirmed 1-star reviews as negative, versus **57.2%** for a lexicon-based method (VADER)
- Unsupervised topic modeling surfaced **6 distinct complaint categories** invisible to manual keyword tagging
- Identified a specific, independently verifiable technical compatibility issue (GrapheneOS) affecting a small but high-loyalty customer segment, with a company reply rate of just **2.8%** against a 24.5% overall average

---

## Tools Used

- **Python** — `google-play-scraper`, Companies House REST API, Tesseract OCR, Poppler (for extracting text from image-based financial filing PDFs)
- **pandas** — data cleaning and processing
- **VADER, HuggingFace Transformers (DistilBERT), BERTopic, scikit-learn** — sentiment analysis, unsupervised topic modeling, and predictive modeling
- **Power BI (DAX, Power Query)** — final dashboard and visualization layer

---

## Methodology

- **Data cleaning:** Identified and removed a CSV parsing corruption affecting ~5.9% of scraped review rows — unescaped commas inside reply text had caused column misalignment. Caught and excluded via validation against the `rating` field, rather than assumed away.
- **Sentiment analysis:** Started with VADER, a standard lexicon-based approach. Rather than take its output at face value, it was benchmarked against a transformer model using star rating as ground truth — and VADER underperformed significantly enough (57.2% vs 95.9% correct identification of confirmed 1-star reviews) that the transformer model was adopted as the primary method going forward. The benchmarking exercise itself is documented as a finding, not discarded once a "winner" was picked.
- **Theme analysis:** A manual, keyword-based tagging system covered the expected categories — account freezes, customer support, fees, verification, app stability, transfers.
- **Unsupervised topic modeling (BERTopic):** Used specifically to catch what a predefined keyword list couldn't anticipate. Default settings proved unreliable on this dataset — reproducibility testing across multiple runs showed topic composition shifting meaningfully run to run, a known risk of density-based clustering on a dataset this skewed (~88% positive reviews). Fixed by lowering the clustering threshold and sorting resulting clusters by actual average star rating, rather than trusting the auto-generated keyword labels, which were often too generic to be useful on their own.
- **Predictive modeling:** A Random Forest classifier was trained to predict 1-star reviews from sentiment score, review length, and theme flags — used to rank which specific complaint themes carry independent predictive weight, rather than relying on raw complaint volume alone.

---

## Insights

- **The financial story is a genuinely strong one, not just a growth-at-all-costs one.** Margin expansion came primarily from administrative expense ratio falling (72.3% → 40.0%), not from revenue alone — evidence of real operating discipline. Decelerating revenue growth (94.9% → 71.3% → 46.6% YoY) paired with accelerating profitability reads as a business transitioning from hypergrowth into maturity, not slowing down in a worrying way.

- **A staffing decision didn't produce the outcome you'd expect — and that's the more interesting result.** Customer support's relative share of headcount fell in 2025 even as the user base grew. The intuitive assumption is that this should show up as rising complaint volume in the reviews. It didn't — complaint volume trended downward over the same window. Rather than force a cause-and-effect conclusion either way, this is flagged as a genuinely open question: possible explanations include improved self-service tooling, automation, or a shift in *what* customers are dissatisfied about (verification, fraud handling) rather than support capacity itself.

- **Unsupervised modeling found something no keyword list would have looked for.** BERTopic surfaced a cluster tied to GrapheneOS, a privacy-focused Android OS, that manual theme-tagging had no way to anticipate. Reading the actual review text inside that cluster — not just its auto-generated keyword summary — revealed several reviewers self-identifying as long-tenured, Premium ("Metal" tier) customers. That detail was invisible in the model's own labeling; it only surfaced by going back and reading the underlying text by hand.

- **The reply-rate gap on that cluster is the sharpest single number in the whole project.** 52 distinct customers (zero repeat reviewers), 91% of them 1-star, and a 2.8% company reply rate — against a 24.5% average across every other complaint category. The complaint timeline also lines up precisely with GrapheneOS's own public statements describing the block as a deliberate policy decision, not a bug, adding independent corroboration outside the review data itself.

---

## Recommendations

1. **Prioritize a fix for GrapheneOS compatibility.** This affects a small but high-value, technically sophisticated, and publicly vocal segment — several of whom are long-tenured Premium subscribers. The underlying fix (supporting GrapheneOS's published attestation compatibility guide) is a comparatively low-cost technical change relative to the reputational and regulatory exposure of continued neglect, particularly with GrapheneOS's own channels reporting ongoing contact with the EU Commission on this issue.

2. **Investigate the reply-rate inconsistency across complaint categories, not just the GrapheneOS case.** Fraud/unauthorized transactions (45.5%) and unexplained account blocks (38.4%) also sit well below the 24.5% average, despite carrying some of the lowest average star ratings (1.10 and 1.07) in the dataset — these are the highest-severity complaints receiving comparatively little visible response.

3. **Investigate the support-headcount-to-complaint-volume disconnect directly, rather than treating the 2025 staffing reduction as validated by falling complaint volume.** The data doesn't support a simple "it worked" conclusion — it supports a more specific question worth answering internally: did resolution quality change, or did the *nature* of complaints shift toward categories this review data doesn't fully capture?

4. **Extend this analysis to iOS review data.** Every finding here — including the GrapheneOS case, which is Android-specific by nature — reflects Android users only. A genuinely complete picture of customer sentiment needs the other half of the user base included.

---

## Screenshots

<img width="1326" height="743" alt="Financial performance overview" src="https://github.com/user-attachments/assets/b21032ad-54a6-484e-9000-ba85c8b66ca0" />

<img width="1322" height="736" alt="Revenue mix and lending" src="https://github.com/user-attachments/assets/68853259-3477-4e91-ab58-0bf1ec3c6d24" />

<img width="1323" height="743" alt="People and operations" src="https://github.com/user-attachments/assets/4e7e28ac-314a-4524-8dbe-2142deb093b2" />

<img width="1321" height="742" alt="Complaint categories" src="https://github.com/user-attachments/assets/29714f6f-3f0b-4191-85e9-5a6a42c07f6d" />

<img width="1315" height="737" alt="Topic modeling detail" src="https://github.com/user-attachments/assets/606fce60-2890-4f48-ab4d-3b577c12f493" />

<img width="1320" height="740" alt="GrapheneOS spotlight" src="https://github.com/user-attachments/assets/20c8be14-5c19-4694-9eb3-986c34a1c23f" />

---

## Limitations

- **Review data is Android-only.** All reviews were collected from the Google Play Store; no App Store (iOS) reviews are included. Every finding on customer sentiment — including the GrapheneOS case study, which is itself Android-specific — should be read as representative of Android users only.
- **The two datasets don't share a common timeline.** Review data covers a variable historical window; financial data covers audited years 2021–2025 only. Where they don't overlap, they're presented as separate analytical views rather than forced into one misleading combined timeline.
- **BERTopic's reproducibility issue is disclosed, not hidden.** Multiple runs produced varying topic composition — this is documented transparently in the accompanying notebooks rather than presenting only the most favorable single run.
- **This project has no access to Revolut's internal data** — no support tickets, resolution outcomes, or retention metrics. Every conclusion is built entirely from public review text and public financial filings, and should be read with that ceiling in mind.

---

## Key Learnings

- **Benchmarking a method against ground truth is worth doing even when the "obvious" method seems fine.** VADER looked like a reasonable default until it was checked against actual star ratings — at which point it missed nearly half of confirmed negative reviews. Picking a method without checking it against something objective would have quietly weakened every downstream finding.
- **An automated label is a starting point, not the finding itself.** BERTopic's keyword summaries didn't surface the Premium-tier detail in the GrapheneOS cluster at all — that only came from reading the actual review text the model had grouped together. The model found the *shape* of the issue; understanding it required going back to the raw data.
- **A model's default settings deserve the same scrutiny as its output.** BERTopic's reproducibility problem wasn't visible from a single run — it only showed up by deliberately re-running the same analysis and checking whether the answer changed. Reporting one favorable run without that check would have been a real, avoidable blind spot.

---

## Repository Structure
