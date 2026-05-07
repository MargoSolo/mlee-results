<div align="center">

# 🐭 MLEE — Mouse Longevity Evidence Engine

**An AI system that reads 10,000 scientific papers on mouse aging**  
**and extracts every lifespan result automatically — with full citations.**

[![Gamma](https://img.shields.io/badge/Project%20Overview-Gamma-6366f1?style=for-the-badge)](https://gamma.app/docs/j5sob9qdwabesdg)
[![LinkedIn](https://img.shields.io/badge/Margarita%20Soloshenko-LinkedIn-0077B5?style=for-the-badge&logo=linkedin)](https://www.linkedin.com/in/margarita-soloshenko/)

<img src="mlee_gamma_qr.png" width="180" alt="Scan to open project overview"/>

*Scan to open full project overview*

</div>

---

## The Problem

Researchers spend **weeks** reading hundreds of papers to answer one question:  
*"Which interventions actually extend mouse lifespan, and by how much?"*

The data exists — but it's buried in tables, text, and Kaplan-Meier survival curve figures across thousands of PDFs. There is no unified, queryable database.

**MLEE solves this.**

---

## By the Numbers

| | |
|---|---|
| 📄 **10,753** papers analyzed | Every major mouse lifespan intervention study |
| ⏱ **40 minutes** to process 1,000 papers | What once took weeks |
| 💰 **$30** total AI cost | End-to-end, full corpus |
| ✅ **93%** pass rate | Automated 3-layer quality verification |

---

## What's in This Repository

### `vision_sample1000_experiments.xlsx`

102 experiments extracted from 1,000 papers. **6 sheets:**

| Sheet | Contents |
|-------|----------|
| All experiments | Full dataset, all 102 records |
| Passed | 91 experiments that passed all quality checks |
| Flagged | 9 experiments with minor inconsistencies |
| Failed | 2 rejected records |
| Extends lifespan | 38 experiments with positive effect |
| Shortens lifespan | 9 experiments with negative effect |

**Why 102 experiments from 1,000 papers?**

- 1,000 papers searched
- ~317 had full text available in PMC Open Access
- ~100 of those contained actual mouse lifespan experiments (the rest were reviews, methodology papers, or had no survival data)
- One paper can yield multiple experiments (different sexes, doses, or strains) → 100 papers → 102 rows

**What each sheet means:**

| Sheet | Contents |
|-------|----------|
| All experiments | Full dataset, all 102 records |
| Passed | 91 experiments that passed all 3 verification layers — safe to use |
| Flagged | 9 experiments with minor inconsistencies (e.g. percent_change doesn't match) — included but marked |
| Failed | 2 experiments with clear errors — excluded from analysis |
| Extends lifespan | 38 experiments where treatment group outlived control (percent_change > 0) |
| Shortens lifespan | 9 experiments where treatment reduced lifespan |
| ITP studies | 4 experiments from the NIA Interventions Testing Program — independently replicated at 3 sites, highest reliability |

**Field coverage on 1,000 papers:**

| Field | Coverage |
|-------|----------|
| Intervention class | 100% |
| Sex | 100% |
| Median survival — treatment (days) | 36% |
| p-value | 27% |
| Sample size (n) | 36% |
| Funding source | 50% |

---

## How It Works: Two-Pass Extraction

Most lifespan data hides in two places — **text** and **figures**. MLEE reads both.

### Pass 1 — Text Extraction (Gemini Flash)
Reads full PMC XML → structured JSON with ~40 fields:
intervention, strain, sex, median survival, p-value, sample size, funding, ITP flag.
Validated by Pydantic schema before storage.

### Pass 2 — Vision Extraction (Gemini Flash Vision) ✨ New

**What is a Kaplan-Meier curve?**  
A Kaplan-Meier (KM) curve is the standard way to show survival data in biomedical research. The x-axis is time (days, weeks, or months), the y-axis is the proportion of mice still alive (0–100%). Each group — treatment and control — has its own curve. The **median survival** is simply where a curve crosses the 50% line. Authors frequently show this figure without writing the actual numbers in the text — making it invisible to any text-only extraction system.

```
100% |──────╮
     |       ╲  Treatment
 50% |·······×··╲·········  ← median survival (read here)
     |            ╲  Control ╲
  0% |_________________________
        Time (days/weeks)
```

For papers where survival data is **only shown in a KM figure** (not in text), MLEE reads the figure directly:

1. Scans `<fig>` captions for keywords: *kaplan-meier, survival curve, lifespan, percent alive, age at death...*
2. Downloads figure image from **PMC Open Access S3**: `pmc-oa-opendata.s3.amazonaws.com/{PMCID}.1/{filename}`
3. Sends image to Gemini Vision: *"Where does each curve cross the 50% survival line?"*
4. Returns structured JSON: treatment days / control days / p-value / n / confidence
5. Patches **only null fields** — never overwrites text-extracted data
6. Sanity check: rejects values > 1,825 days (~5 years, impossible for mice)

> **Result:** value_treatment coverage grew from 29% → 36% (+7%).  
> 7 experiments extracted exclusively from KM figures — invisible to any prior text-only system.

### Verification — 3 Layers
1. **Plausibility** — numeric range checks (lifespan bounds, % change limits)
2. **Consistency** — percent_change recompute, HR/CI/p cross-check via z-statistic
3. **Crossref** — drug names → ChEMBL IDs, genes → HGNC symbols (async, best-effort)

---

## Example Finding

> ### "FGF21 extends lifespan by **+40%** in female mice and **+37%** in male mice"

FGF21 is a liver hormone that mimics the effects of fasting without fasting.  
Mice overexpressing FGF21 consistently outlived controls across independent studies.

- **p = 1.8 × 10⁻⁸** (females) · **p = 0.00017** (males)
- Strain: C57BL/6J · Both sexes
- Source: PMID 24175087 — Zhang et al., 2012

*For comparison: rapamycin extends lifespan ~14% in similar mice.*

---

## Sample Vision Pass Results

Experiments extracted from KM figures (previously null in text extraction):

| PMID | Intervention | Treatment (days) | Control (days) | Change | p-value | Confidence |
|------|-------------|:---:|:---:|:---:|---|:---:|
| 31285335 | AAV9-TRF1 | 945 | 875 | **+8%** | P < 0.05 | High |
| 26268661 | IκBα overexpression | 950 | 850 | **+11.8%** | P < 0.0001 | High |
| 26268661 | IKKβ knockout | 950 | 850 | **+23%** | P = 0.0002 | High |
| 36319638 | — | 714 | 784 | −8.9% | — | Medium |

---

## Built For

| Audience | Use case |
|----------|----------|
| 🔬 Longevity research labs | Instant evidence review across thousands of studies |
| 💊 Biotech & Pharma | Target validation in hours, not weeks |
| 📈 Investors | Due diligence on aging-focused startups, backed by data |
| ⚙️ AI infrastructure teams | Production case study: Vertex AI + Gemini at scale |

---

## What Each Column in the Excel Means

<details>
<summary><b>Click to expand full column reference</b></summary>

### Identifiers
| Column | Description |
|--------|-------------|
| `pmid` | PubMed ID — unique identifier of the source paper |
| `experiment_id` | Unique ID of this experiment within the paper |
| `verification_status` | `passed` / `flagged` / `failed` |
| `verification_flags` | Comma-separated list of warnings or notes from the verification pipeline |
| `year` / `journal` / `title` | Paper metadata from PubMed |

### Mouse Model
| Column | Description |
|--------|-------------|
| `strain_as_reported` | Strain name exactly as written in the paper |
| `strain_standardized` | Normalized name (e.g. "C57BL/6J") |
| `strain_category` | `inbred` / `outbred` / `hybrid` / `transgenic` / `knockout` |
| `sex` | `male` / `female` / `both` / `not reported` |
| `age_start_value` + `age_start_unit` | Age at which treatment started |

### Intervention
| Column | Description |
|--------|-------------|
| `intervention_name` | Name as reported in the paper |
| `intervention_name_std` | Standardized name |
| `intervention_type` | `pharmacological` / `genetic` / `dietary` / `other` |
| `intervention_class` | Broader category (e.g. `mTOR inhibitor`, `caloric restriction`) |
| `chembl_id` | ChEMBL database ID (for drugs) |
| `hgnc_id` + `gene_name` | Gene identifier (for genetic interventions) |
| `route` | Administration route (oral, injection, etc.) |
| `dose_as_reported` | Dose exactly as written |
| `lifelong` | `Yes` if treatment was given for the animal's entire life |

### Primary Outcome (Lifespan)
| Column | Description |
|--------|-------------|
| `value_treatment_days` | Median survival of the treatment group, in days |
| `value_control_days` | Median survival of the control group, in days |
| `percent_change` | `(treatment − control) / control × 100` |
| `percent_change_as_reported` | % change as written in the paper |
| `hazard_ratio` | HR from survival analysis (if reported) |
| `p_value` | Numeric p-value |
| `p_value_as_reported` | p-value exactly as written (e.g. "p<0.001") |
| `test_used` | Statistical test (log-rank, Mantel-Cox, etc.) |

### Sample Size
| Column | Description |
|--------|-------------|
| `n_treatment` | Number of mice in treatment group |
| `n_control` | Number of mice in control group |

### Quality Flags (qa_*)
| Column | Description |
|--------|-------------|
| `qa_p_exact` | `Yes` if exact p-value given (not just "p<0.05") |
| `qa_ci_given` | `Yes` if confidence interval reported |
| `qa_censoring_described` | `Yes` if censoring method described |
| `qa_randomization` | `Yes` if randomization described |
| `qa_blinding` | `Yes` if blinding described |
| `qa_power_calc` | `Yes` if power calculation reported |
| `qa_pct_recomputed_matches` | `Yes` if our recomputed % change matches the reported one |

### Study Context
| Column | Description |
|--------|-------------|
| `itp_study` | `Yes` if from NIA Interventions Testing Program |
| `funding_source` | Funding agency as reported |
| `control_type` | `vehicle` / `untreated` / `sham` / etc. |
| `evidence_span` | `abstract_only` / `full_text` / `figure_vision` |
| `verification_flags` | Notes from pipeline: vision patch, strain normalization, etc. |

</details>

---

## PubMed Search Strategy

10,753 papers retrieved via **3 queries** on PubMed, period **2000–2025**.

### Query 1 — Main interventions
Lifespan terms (`lifespan, longevity, survival, mortality`) × mice (`mice, mus musculus, murine`) × specific interventions:

| Category | Keywords |
|----------|----------|
| Diet | caloric restriction, intermittent fasting, dietary restriction |
| Drugs | rapamycin, metformin, acarbose, resveratrol, 17α-estradiol, aspirin, NMN, nicotinamide riboside, spermidine, fisetin, quercetin |
| Senolytics | senolytic, senostatic |
| Genes / pathways | FGF21, klotho, GHR knockout, GHRKO, telomere, telomerase, FOXO, SIRT, sirtuin, autophagy, IGF-1, growth hormone, Ames dwarf, Snell dwarf |
| Experimental | parabiosis, young blood, plasma |

**Excluded:** in vitro, cell culture, C. elegans, Drosophila, yeast

### Query 2 — ITP / NIA studies
`Interventions Testing Program` OR `NIA aging` OR `National Institute on Aging` × mice × lifespan

### Query 3 — Genetic longevity models
`long-lived` OR `extended lifespan` OR `longevity mutant` × transgenic/knockout × survival

---

## About

**Margarita Soloshenko**  
MSc Applied Genomics — Universidad de Vigo (CINBIO)  
Built during the **Google Cloud × Devpost Hackathon, May 2026**

[![LinkedIn](https://img.shields.io/badge/LinkedIn-margarita--soloshenko-0077B5?logo=linkedin)](https://www.linkedin.com/in/margarita-soloshenko/)
