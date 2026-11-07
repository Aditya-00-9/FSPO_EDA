# FSPO Banking Decisions – Exploratory Data Analysis

Ireland's Financial Services and Pensions Ombudsman (FSPO) Banking sector analysis for all the
legally binding decisions made between 2018 and March 2026 (1,686 decisions).

The notebook scrapes the database of public decisions made by the FSPO, ensures the extract is
complete, cleans and enriches the data, and explores variations of outcomes on complaints based
on product, type of complaint and year.

---

## Key findings

| Measure | Result |
|---|---|
| Decisions analysed | 1,686 (100% of site total – 20 of 20 filter cross-checks matched) |
| Rejected | 74.7% |
| Upheld to any degree | 25.3% (Upheld 6.4%, Substantially 5.3%, Partially 13.6%) |
| Class imbalance | 14.15 : 1 (four classes), 2.95 : 1 (binary) |
| Tracker mortgages | 48.2% of all decisions (90.8% rejected) |
| All other banking | 59.7% rejected |

- **Tracker mortgages drive the headline rejection rate.** 115 decisions refer to the Central
  Bank's "tracker-mortgage examination", under which most of those affected had already
  received compensation.
- **Product matters.** Consumer credit, credit cards and personal loans are upheld in about half
  of the decisions, while tracker mortgages are upheld in only 9.2%.
- **Conduct matters.** Customer service (50.8%), maladministration (45.7%) and arrears handling
  (44.0%) complaints fare significantly better than tracker rate complaints (8.1%).
- **Rejection rates have risen**, from 47.8% in 2018 to about 88% in 2023–2026. This increase is
  observed even when tracker cases are not included (approx. 46% to approx. 67–77%).
- **Class imbalance.** If a model predicted "Rejected" every time, it would still be 74.7%
  accurate but quite useless. Any predictive work will require class weighting and/or
  resampling, along with measures like F1 or balanced accuracy.

---

## Method

### 1. Scraping
`requests` + `BeautifulSoup` page through the FSPO decisions listing (Banking sector,
25 decisions per page, 68 pages), extracting the reference, title, sector, product, primary
conduct, all conducts, outcome, year and PDF link. Requests are rate-limited (1.5 s), retried
with backoff, and cached to disk.

### 2. Verification
- The number of scraped rows vs. the number of decisions the site indicates
- Duplicate check
- A probe of the page after the last one
- A cross-check of all outcome, product and year filters against the scraped counts

### 3. Cleaning
- Normalising whitespace and labels
- An ordered outcome category and a binary outcome
- 43 titles mapped to 10 product groups
- A tracker-mortgage flag
- Parsed conduct lists
- Checks on reference format, year consistency and PDF-link matching
- Flags on text truncated by the source site

### 4. Analysis
- Outcome distribution
- Class-imbalance metrics: ratio, normalised entropy and majority baseline
- Outcomes by product group, by year and by complaint conduct (tracker vs non-tracker)

### 5. Visualisation
Four publication-ready charts (300 dpi) with source notes.

---

## Repository contents

| File | Description |
|---|---|
| `FSPO_DGE.ipynb` | The whole pipeline: scraping, verification, cleaning, analysis, charting |

When the notebook is run, it creates:

```
data/raw/fspo_banking_decisions.csv        # raw scraped decisions
data/raw/scrape_verification.json          # completeness checks
data/raw/html_cache/                       # cached listing pages
data/processed/fspo_banking_cleaned.csv    # cleaned, enriched dataset
data/processed/data_quality_report.json    # data-quality summary
eda_statistics.json                        # every figure used in the analysis
charts/chart1_outcome_distribution.png
charts/chart2_class_imbalance.png
charts/chart3_outcomes_by_product.png
charts/chart4_rejected_share_by_year.png
```

---

## How to run

Install packages:

```bash
pip install pandas numpy matplotlib requests beautifulsoup4
```

Open the notebook:

```bash
jupyter notebook FSPO_DGE.ipynb
```

Settings at the top of the notebook:

| Setting | Default | Purpose |
|---|---|---|
| `USE_CACHE` | `True` | Reuse cached HTML instead of downloading it again |
| `CROSS_CHECK` | `True` | Check the counts against the site's own filters |
| `DELAY_SECONDS` | `1.5` | Delay (s) between requests |
| `SECTOR` | `"1"` | FSPO sector code (1 = Banking) |

---

## Limitations

- **Published decisions only.** In most cases, FSPO complaints are resolved informally via
  resolution or mediation. Outcome shares are not the likelihood that a complaint will be
  successful.
- **Metadata only.** The reasoning, the amount of redress and the names of providers were out of
  scope because the decision PDFs were not read.
- **Source truncation.** 28 titles and 25 conduct lists are cut off on the FSPO site. They are
  flagged, not filled in.
- **2026 is a partial year** (decisions until March 2026).
- **Product groups are an analytical choice.** See `PRODUCT_GROUP_MAP` in the notebook.
- **The results are descriptive associations**, rather than causal effects.

---

## Data source

FSPO Legally Binding Decisions database:
<https://www.fspo.ie/complaint-outcomes/investigation-services/legally-binding-decisions/>

Data was gathered for academic research purposes (NCI student project) with rate-limited
requests and User-Agent identification. Only metadata was gathered.
