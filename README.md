# Tariff Wall Similarity Analysis

## Overview

This section analyzes the structure of U.S. import tariffs to understand
how different industries are protected under trade policy. Using tariff
data from the **2025 U.S. Tariff Database**, the project explores
patterns in tariff rates across product chapters and builds a **network
model of tariff similarity** between industries.

The analysis investigates how tariff structures cluster together and
whether U.S. trade policy follows a pattern of **conditional
openness**---where some sectors remain open to global trade while others
receive stronger protection.

------------------------------------------------------------------------

## Dataset

The dataset used in this project is:

-   **tariff_database_2025.xlsx**

It contains tariff schedule information including:

-   `hts8` -- 8-digit Harmonized Tariff Schedule code\
-   `mfn_ad_val_rate` -- Most Favored Nation tariff rate\
-   `col2_rate` -- Column 2 tariff rate (penalty tariffs for non-normal
    trade relations)\
-   `mfn_rate_type_code` -- To identify duty-free items\
-   Additional product classification data

For analysis, the HTS code is reduced to its **first two digits**, which
represent one of the **97 tariff chapters (industry sectors)**.
We skip chapters 98 and 99 as they are legislatures and not industrial sectors.

------------------------------------------------------------------------

## Goals

The section addresses three primary questions:

1.  **Which industries face the highest tariff protection?**
2.  **Which industries operate under nearly free trade conditions?**
3.  **Do industries cluster together based on their tariff structures?**

Additionally, this section tests a broader hypothesis about trade policy
behavior.

------------------------------------------------------------------------

## Data Processing

Several preprocessing steps were performed:

1.  **Chapter Extraction**
    -   The first two digits of the HTS8 code are used to determine the
        tariff chapter.
2.  **Tariff Cleaning**
    -   Extremely high tariff placeholder values (e.g., `9999.99`) were
        capped or removed to prevent distortion.
3.  **Feature Engineering** Three key metrics were calculated for each
    chapter:
    -   Average MFN tariff rate
    -   Average Column 2 tariff rate
    -   Percentage of duty-free items
4.  **Duty-Free Indicator**
    -   A binary feature (`is_free`) identifies items with zero tariff.

------------------------------------------------------------------------

## Exploratory Analysis

We first performed statistical and correlation analysis across
chapters.

### Key Observations

-   A **moderate to strong correlation** exists between MFN tariffs and
    Column 2 tariffs.
-   Industries protected from regular trade partners tend to have **even
    higher punitive tariffs** for countries without normal trade
    relations.

The analysis also identifies:

-   **Most Protected Chapters** (highest average MFN tariff)
-   **Most Open Chapters** (highest percentage of duty-free goods)
-   **Highest Penalty Tariff Chapters** (highest Column 2 tariff rates)

------------------------------------------------------------------------

## Tariff Similarity Network

To understand structural similarities between industries, we construct a **network graph**.

### Steps

1.  Standardize chapter-level tariff features.
2.  Compute **cosine similarity** between chapters.
3.  Build a graph where:
    -   Each node represents a **tariff chapter**
    -   Edges connect **most similar chapters**

Network analysis is performed using:

-   `NetworkX` for graph construction
-   `Matplotlib` and `Plotly` for visualization

### Community Detection

The project uses:

`greedy_modularity_communities`

to detect clusters of chapters that share similar tariff
characteristics.

These clusters reveal groups of industries with similar trade protection
strategies.

------------------------------------------------------------------------

## Interactive Visualization

An interactive network visualization allows users to explore:

-   Tariff chapter relationships
-   Community clusters
-   Similar industries in tariff structure

The network layout is generated using a **spring layout algorithm** for
clear visualization of clusters.

------------------------------------------------------------------------

## Hypothesis: The Preference Exclusion Effect

### Hypothesis

The **Preference Exclusion Hypothesis** suggests that U.S. trade policy
follows a model of **conditional generosity**.

Under this model:

-   Some products are duty-free for **all countries**
-   Others receive **special trade preferences** (such as GSP or AGOA)
-   But products that are already duty-free rarely receive additional
    preferential programs

### Key Findings

-   Only about **12% of duty-free items** appear in preference programs.
-   This suggests that trade preference programs primarily target
    **products that would otherwise face tariffs**.
-   There was a **~91%** acceptance rate for the GSP and AGOA programs for moederately tariffed sectors.
-   As a sector was deemed to be too sensitive, the acceptance rate for these programs dropped to **55.96% and 60.72%** for GSP and AGOA respectively.

This supports the idea that preference systems are designed to **reduce tariffs selectively rather than expand already free sectors**.

------------------------------------------------------------------------

## Technologies Used

-   **Python**
-   **Pandas** -- data processing
-   **NumPy** -- numerical operations
-   **Scikit-Learn** -- feature scaling and similarity
-   **NetworkX** -- graph modeling
-   **Matplotlib / Seaborn** -- static visualizations
-   **Plotly** -- interactive network graphs

------------------------------------------------------------------------

## How to Run

### 1. Install dependencies

``` bash
pip install pandas numpy matplotlib seaborn scikit-learn networkx plotly openpyxl
```

### 2. Place the dataset

Add the dataset file:

    tariff_database_2025.xlsx

into the project directory.

### 3. Run the notebook

Open and run:

    Project_Q1.ipynb

in Jupyter Notebook or VS Code.

------------------------------------------------------------------------
# Supply Chain Tariff Inversion Analysis

## Overview

This section identifies **tariff inversions** in the 2025 U.S. Harmonized Tariff Schedule —
cases where raw materials are taxed at a **higher rate** than the finished goods made from them.

When raw materials are taxed more than finished imports, domestic manufacturers are
structurally penalized. It becomes cheaper to import the finished product than to
import the ingredients and make it here.

---

## Dataset

The datasets used in this project are:

- **hts_cleaned.csv** — Cleaned 2025 Harmonized Tariff Schedule
- **IOUse_Before_Redefinitions_PRO_Detail.xlsx** — BEA 2017 Input-Output Use Table

The HTS dataset contains:

- `hts8` — 8-digit Harmonized Tariff Schedule code
- `brief_description` — Plain English product name
- `rate_clean` — MFN tariff rate as a decimal
- `has_adval` — Whether the item has a percentage-based rate
- `chapter` — First two digits of HTS code, groups related products

Chapters 98 and 99 are excluded as they are administrative, not product categories.

---

## Goals

This section addresses three primary questions:

1. **Which industries have the most severe tariff inversions?**
2. **Which raw material → finished good pairs are most penalized?**
3. **Do inversions exist consistently across unrelated industries?**

---

## Data Processing

Several preprocessing steps were performed:

1. **Rate Filtering**
   - Only ad valorem (percentage-based) rates are kept.
   - Specific rates such as $/kg cannot be compared to percentages and are excluded.
2. **Chapter Extraction**
   - First two digits of the HTS8 code determine the tariff chapter.
3. **Supply Chain Mapping**
   - 26 keyword-based rules connect raw material chapters to finished good chapters
     across 6 industries.
4. **BEA Validation**
   - Every raw → finished pair is verified against the BEA Input-Output table.
   - Pairs must have at least $100M in documented industry flows to be confirmed.

---

## Methodology

We define supply chain relationships using keyword matching on HTS product descriptions.
Each rule maps a raw material chapter to a finished good chapter using keywords that
must appear in the product description.

For each rule, every matching raw item is compared against every matching finished item.
Pairs where `raw_rate > finished_rate` are flagged as inversions.

**Deduplication:**
- Keep only the worst gap per unique raw → finished pair
- Keep only one row per raw material (its single worst inversion)

This produces a conservative lower bound — the true number of inversions
within our defined supply chains is significantly larger.

---

## Key Findings

- **682 inverted pairs** identified across 6 industries
- All 682 pairs confirmed by BEA Input-Output flows
- Food Processing has the largest average gap at **10.1 percentage points**
- Worst single pair: raw peanuts (163.8%) → peanut butter (131.8%) = **32 pt gap**
- Apparel & Textiles has the most inversions by volume — **349 of 682 pairs**
- Inversions exist in every industry examined

| Industry | Pairs | Avg Gap | Max Gap |
|---|---|---|---|
| Food Processing | 34 | 10.1 pts | 32.0 pts |
| Apparel & Textiles | 349 | 9.0 pts | 24.2 pts |
| Electronics | 69 | 3.3 pts | 5.5 pts |
| Automotive | 87 | 2.5 pts | 12.6 pts |
| Leather & Footwear | 92 | 1.4 pts | 3.2 pts |
| Metal Products | 51 | 0.9 pts | 1.0 pts |

---

## Technologies Used

- **Python**
- **Pandas** — data processing
- **Matplotlib** — visualizations
- **openpyxl** — BEA Excel file loading

---

## How to Run

### 1. Install dependencies
```bash
pip install pandas matplotlib openpyxl
```

### 2. Place the datasets

Add the following files into the project directory:

    hts_cleaned.csv
    IOUse_Before_Redefinitions_PRO_Detail.xlsx

### 3. Run the notebook

Open and run:

    tariff_inversion_analysis.ipynb

in Jupyter Notebook or VS Code.

---

## Authors

- Ari Munkhtur




## Authors

-   Jayesh Bane
