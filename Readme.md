# Super Puna - Labor Market Analysis 2025

A data analysis project examining the data scraped and gathered about the labor market demand in Kosovo using job postings scraped from [SuperPuna.com](https://superpuna.com) — the country's largest online job portal. Job titles are standardized to the European **ESCO v1.2** and **ISCO-08** classification frameworks, enabling comparison with EU-level labor market research.
Super Puna is an employment platform created by the Government of Kosovo, part of the Guaranteed Employment scheme designed to help young people access the labor market and assist businesses in hiring young workers.

---
# Data

## Raw Dataset

The SuperPuna job postings dataset is not included in this repository as it contains scraped third-party data from SuperPuna.com .

To replicate this analysis you would need a dataset of Kosovo job postings containing the following columns:

| Column | Description |
|---|---|
| `Titulli_i_vendit_te_punes` | Job title (Albanian) |
| `Numri_i_vendeve_te_lira` | Number of vacancies |
| `Kompania` | Company name |
| `Vendi_i_punes` | City or work location |
| `Orari_i_punes` | Work schedule (time range format HH:MM:SS - HH:MM:SS) |
| `Paga` | Monthly salary (EUR) |
| `Data_e_shpalljes` | Posting date |
| `Kerkest` | Job requirements (free text, Albanian) |
| `Pergjegjesite` | Job responsibilities (free text, Albanian) |
| `Kualifikimet` | Qualifications required (free text, Albanian) |
| `Benefitet` | Benefits offered (free text, Albanian) |

## Included Files

### translation_map.csv
A dictionary of 6,172 Albanian job titles translated to English, generated during this analysis using the Google Translate free tier via the `deep-translator` library.

This file allows you to skip the 45-minute translation step when running `01_data_cleaning.ipynb`. To use it:

1. Download `translation_map.csv` from this repository
2. Upload it to your Google Drive at `My Drive/kosovo_lm/translation_map.csv`
3. Run Notebook 1 — it will detect the file automatically and skip translation

To re-translate from scratch, simply delete the file from Google Drive before running Notebook 1.

## Overview

| | |
|---|---|
| **Data source** | SuperPuna.com |
| **Period covered** | January 1 – December 31, 2025 |
| **Total job postings** | 15,422 |
| **Total vacancies** | 20,902 |
| **Unique companies** | 9,114 |
| **Cities/regions** | 38 |
| **Occupation standard** | ESCO v1.2 (EU Commission) / ISCO-08 (ILO) |
| **Match rate** | 90.5% of postings standardized |
| **Language** | Albanian → English (machine translated) |

**Occupation standard** 

|**ESCO (European Skills, Competences, Qualifications and Occupations)** It is the European multilingual classification system that works like a dictionary, describing and classifying professional occupations and skills relevant to the EU labour market and education |

|**ISCO - International Standard Classification of Occupation**  It is a framework developed by the International Labour Organization (ILO) that organizes all jobs and occupations into a hierarchical structure for statistical and research purposes.|
---

## Research Questions

1. What is the total number of job postings and total number of vacancies?
2. How many unique companies are actively hiring?
3. How do postings vary by day, week, and month?
4. Which companies post the most jobs and advertise the most positions?
5. How are vacancies distributed across cities and regions?
6. What is the average wage level by occupational sector?
7. How are vacancies distributed across ISCO-08 occupational sectors?

---

## Key Findings

- **Service and Sales Workers** is the most demanded sector (25.7% of postings), driven by waiters, cashiers, and sales assistants
- **Pristina accounts for 32% of all vacancies** — reflecting extreme economic concentration in the capital
- **The median wage across all sectors is 350 EUR** — Kosovo's statutory minimum wage — indicating that more than half of advertised positions offer minimum wage regardless of sector
- **Professionals are the second largest group** (21.2%), suggesting growing white-collar demand
- **Agriculture is nearly invisible** in online job postings (0.2%), consistent with the sector relying on informal hiring channels
- **July is the busiest hiring month** (1,660 postings), driven by seasonal demand in hospitality and retail

---

## Methodology

### 1. Data Collection
Job postings were scraped from SuperPuna.com covering the full calendar year 2025. The raw dataset contains 15 fields including job title, company, city, salary, work schedule, qualifications, and posting date.

### 2. Job Title Normalization
Albanian job titles use gendered suffixes (e.g. `Shitës/e`, `Kamarier\e`) that create artificial duplicates. A normalization function was applied to collapse these variants:
- Backslash and forward slash gender markers removed (`/e`, `\e`, `/ja`)
- Albanian diacritic characters standardized (`ë → e`, `ç → c`)
- Result: 6,649 unique raw titles reduced to 6,172 normalized titles

### 3. Translation
Normalized Albanian titles were translated to English using the **Google Translate free tier** via the `deep-translator` library (no API key required). Only unique titles were translated (not all 15,422 rows) to minimize API calls.
A saved translation map (`translation_map.csv`) allows re-use without re-translating.

### 4. ESCO Matching 
Translated English titles were matched against the **ESCO v1.2 occupation dataset** (3,039 occupations, ~20,000 searchable labels including alternative labels) using a two-stage pipeline:

| Stage | Method | Library | Threshold | Titles matched |
|---|---|---|---|---|
| Stage 1 | Fuzzy string matching | RapidFuzz | Score ≥ 85 | 1,011 |
| Stage 2 | Semantic similarity | Sentence-BERT (`all-MiniLM-L6-v2`) | Cosine ≥ 0.50 | 3,088 |
| Manual | Domain expert overrides | — | — | 678 rows |

Each matched title receives:
- ESCO preferred occupation label
- 4-digit ISCO-08 code
- ISCO major group (1-digit)
- Match confidence score
- Match method (fuzzy / semantic / manual)

### 5. Analysis
Standardized data was used to produce descriptive statistics across all seven research questions, including time-series analysis, geographic distribution, wage analysis by sector, and occupational demand ranking.

---

## Repository Structure

```
kosovo-labor-market-analysis/
│
├── README.md
├── requirements.txt
│
├── notebooks/
│   ├── 01_data_cleaning.ipynb         # Load, normalize, translate
│   ├── 02_esco_standardization.ipynb  # ESCO matching pipeline
│   └── 03_demand_analysis.ipynb       # All 7 research questions
│
├── data/
│   ├── README.md                      # Data sources and access instructions
│   └── translation_map.csv            # Albanian -> English title translations
│
└── outputs/
    ├── Kosovo_Labor_Market_Analysis_2025.xlsx
    └── ISCO08_Reference_Sheet.xlsx
```

---

## How to Run

### Requirements
- Google Colab (free) or a local Python 3.10+ environment
- The SuperPuna dataset (CSV or Excel format)
- `occupations_en.csv` from the ESCO v1.2 download (see below)

### Step 1 — Get the ESCO dataset
1. Go to [esco.ec.europa.eu/en/use-esco/download](https://esco.ec.europa.eu/en/use-esco/download)
2. Select: Version = v1.2.0 | Language = English | Format = CSV
3. Download and extract the ZIP
4. Keep `occupations_en.csv` — you will upload this to Colab when prompted

### Step 2 — Run the notebooks in order

Open each notebook in Google Colab and run all cells top to bottom:

```
01_data_cleaning.ipynb          → produces: df with title_normalized, title_english
02_esco_standardization.ipynb   → produces: df with esco_occupation, isco_group
03_demand_analysis.ipynb        → produces: charts + Excel report
```

### Step 3 — Outputs
The final Excel report (`Kosovo_Labor_Market_Analysis_2025.xlsx`) contains 8 sheets covering all research questions, formatted to EU Commission reporting standards.

---

## Libraries Used

```
pandas
openpyxl
matplotlib
rapidfuzz
sentence-transformers
scikit-learn
deep-translator
tqdm

```

Install all with:
```bash
pip install pandas openpyxl matplotlib rapidfuzz sentence-transformers scikit-learn deep-translator tqdm
```

---

## Data

The raw SuperPuna dataset is not included in this repository as it contains scraped third-party data. The `translation_map.csv` file (Albanian to English job title translations) is included and can be reused to skip the translation step.

To replicate this analysis with a new dataset, ensure your data contains the following columns (Albanian column names from SuperPuna):

| Column | Description |
|---|---|
| `Titulli_i_vendit_te_punes` | Job title |
| `Numri_i_vendeve_te_lira` | Number of vacancies |
| `Kompania` | Company name |
| `Vendi_i_punes` | City / work location |
| `Paga` | Salary (EUR) |
| `Data_e_shpalljes` | Posting date |

---

## References

- European Commission (2024). *ESCO — European Skills, Competences, Qualifications and Occupations*, Version 1.2. Publications Office of the European Union. [esco.ec.europa.eu](https://esco.ec.europa.eu)
- International Labour Organization (2012). *International Standard Classification of Occupations: ISCO-08*. ILO Geneva. [ilo.org](https://www.ilo.org/public/english/bureau/stat/isco/isco08/)
- Cedefop (2021). *Online job vacancies and skills in 2020: a statistical analysis of online job vacancy data*. Luxembourg: Publications Office. Cedefop reference series; No 114.
- Reimers, N. & Gurevych, I. (2019). Sentence-BERT: Sentence Embeddings using Siamese BERT-Networks. *EMNLP 2019*. [arxiv.org/abs/1908.10084](https://arxiv.org/abs/1908.10084)

---

## Author

Developed as part of a labor market demand analysis project for Kosovo.
Built entirely in Google Colab using open-source tools and free-tier APIs.

---

## License

This project is for academic and research purposes. Job posting data belongs to SuperPuna.com.

ESCO data is published under the [EUPL v1.2 licence](https://esco.ec.europa.eu/en/use-esco/esco-data-legal-notice).


