# Predicting 30-Day Hospital Readmission with Social Determinants of Health (MIMIC-IV)

Research project from DePaul University's CDM Lab, under Professor Roselyne Tchoua. The question: **do social determinants of health (SDOH), like homelessness, unemployment, or substance use, help predict whether a patient will be readmitted to the hospital within 30 days?**

## Data

[MIMIC-IV](https://physionet.org/content/mimiciv/) and [MIMIC-IV-Note](https://physionet.org/content/mimic-iv-note/), from Beth Israel Deaconess Medical Center. The cohort covers **546,028 hospital visits** from **364,627 patients**, plus **653,586 discharge notes**.

> MIMIC-IV is a credentialed-access dataset. Under the PhysioNet data use agreement, **no data, derived data, or notebook outputs are included in this repository**, only code. To run it, you need your own PhysioNet credentialing.

## Approach

**1. Readmission labels.** For each visit, I computed days until the patient's next admission. Visits followed by another admission within 30 days were labeled readmitted, which was 20.3% of visits. I also built 60- and 90-day labels.

**2. Feature matrices.** I built a tiered set of matrices to isolate what each data source adds:

| Matrix | Features | Visits |
|---|---|---|
| **M0** (baseline) | Demographics: insurance, race, language, marital status, age | 546,028 |
| **M1** (+ EHR) | M0 + the most common diagnosis groups (8 or 15 ICD groups, chosen with an elbow plot) | 546,028 |
| **M2** (+ SDOH) | M0 + SDOH Z-codes + 11 NLP-extracted SDOH flags | 546,028 |
| **M2_complete_full** | Only patients with ≥1 documented SDOH Z-code (all 74 codes + NLP) | 24,330 |
| **M3_full** | Only patients with ≥1 of the top 5 Z-codes + NLP | 20,061 |

**3. SDOH extraction, two ways:**
- **Structured:** ICD-10 Z-codes Z55–Z65, which cover problems like housing, employment, and social environment
- **Unstructured:** keyword-based NLP over all 653K discharge notes, covering 11 categories like homelessness, alcohol use, drug use, living alone, food insecurity, and domestic violence

**4. Modeling.** I trained Random Forest classifiers (balanced class weights, 80/20 stratified split) on every matrix. I also compared 30/60/90-day windows, ran K-means clustering (k=5) with t-SNE to look for patient subgroups, and repeated the best model over 10 random splits to check stability.

## Key Findings

- **SDOH is badly under-documented.** Only 3,749 of 364,627 patients (about 1%) have any SDOH Z-code, and no single code reaches 1% of the full population. Most coded patients have just one code, so a "complete SDOH" matrix (90–95% completeness across 5–10 codes) wasn't possible. Zero patients qualified.
- **On the full population, adding SDOH barely helped.** AUC was about 0.60 for every matrix (M0: 0.604, M1: 0.601, M2: 0.608), because SDOH features are almost all zeros for 99% of patients.
- **The SDOH-coded cohort was more predictable.** M2_complete_full reached **AUC 0.659**, with a 10-run mean of 0.655 ± 0.008, and **0.674 at the 60-day window**. This cohort also has a much higher readmission rate (38.6% vs. 20.3%), so these AUCs aren't directly comparable to the full-population models.
- **Within-cluster comparisons show a small, consistent SDOH lift.** Inside each K-means cluster, adding SDOH raised AUC in all 5 clusters (for example, 0.608 → 0.619 in Cluster 1).

**Bottom line:** SDOH carries real signal, but only where it's actually documented. The bigger problem is that hospitals rarely record it.

## Limitations

- **Visit-level split:** the train/test split is by visit, so the same patient can appear in both sets, which may inflate scores. A patient-level split would be stricter.
- **Timing of SDOH codes:** Z-codes are aggregated per patient across all visits, so a code recorded at a later visit can inform an earlier one.
- **Simple NLP:** keyword matching doesn't handle negation, so "denies alcohol use" still gets flagged as alcohol use.

## Tools

Python · pandas · scikit-learn (Random Forest, K-means, t-SNE) · matplotlib · Google Colab

## Files

- `readmission_sdoh_analysis.ipynb`: the full analysis notebook, with all outputs cleared
