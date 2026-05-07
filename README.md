# MLEE — Mouse Longevity Evidence Engine
### Sample Results: 1,000 Papers

An AI system that reads scientific papers on mouse aging and extracts every lifespan result automatically — interventions, strains, survival data, p-values, and full citations.

---

## What's in this repository

| File | Description |
|------|-------------|
| `vision_sample1000_experiments.xlsx` | 102 experiments extracted from 1,000 papers. 6 sheets: All / Passed / Flagged / Failed / Extends lifespan / Shortens lifespan |
| `mlee_gamma_qr.png` | QR code linking to the project overview |

---

## Dataset overview

- **1,000 papers** processed from PubMed / PMC Open Access
- **102 experiments** extracted and verified
- **91 passed** automated quality checks (3-layer verification)
- **7 experiments** extracted exclusively from Kaplan-Meier figure images via Gemini Vision

### Field coverage

| Field | Coverage |
|-------|----------|
| Intervention class | 100% |
| Sex | 100% |
| value_treatment (median survival, days) | 36% |
| p-value | 27% |
| n_treatment | 36% |
| funding_source | 50% |

---

## Extraction pipeline

Data is extracted in two passes:

**Pass 1 — Text (Gemini Flash)**
Reads PMC XML full text → structured JSON with ~40 fields: intervention, strain, sex, median survival, p-value, sample size, funding, ITP flag.

**Pass 2 — Vision (Gemini Flash Vision)**
For papers where survival data is only shown in a Kaplan-Meier figure: downloads image from PMC Open Access S3, asks Gemini Vision to read the 50% survival crossing point, patches null fields only.

**Verification (3 layers)**
1. Plausibility — numeric range checks
2. Consistency — percent_change recompute, HR/CI/p cross-check
3. Crossref — ChEMBL (drug IDs), HGNC (gene symbols)

---

## Example finding

> **FGF21 extends lifespan by +40% (females) and +37% (males)**
> p = 1.8×10⁻⁸ · C57BL/6J · PMID 24175087

---

## Project overview

[![Scan to open](mlee_gamma_qr.png)](https://gamma.app/docs/j5sob9qdwabesdg)

🔗 [Open full project overview](https://gamma.app/docs/j5sob9qdwabesdg)

---

## About

**Margarita Soloshenko**
MSc Applied Genomics — Universidad de Vigo (CINBIO)
Built during the Google Cloud × Devpost Hackathon, May 2026

[![LinkedIn](https://img.shields.io/badge/LinkedIn-margarita--soloshenko-blue)](https://www.linkedin.com/in/margarita-soloshenko/)
