# HDB Resale Prices: A Market Split in Two?

An analysis of Singapore HDB resale transactions (Jan 2017 – Sep 2026) that looks at why record prices keep being set while the overall market softens.

**In short:** both headlines are true. Like-for-like prices dipped about 1% in 2026, while most million-dollar sales come from the whole market rising past a fixed S$1M line. Since mid-2025 the market has started to split by **lease and flat size, not location**. See [Findings](#5-analysis--findings) and [Recommendations](#6-recommendations).

---

## 1. Context

### Background

About 80% of Singapore residents live in HDB (Housing & Development Board) flats. These are public housing flats sold on **99-year leases**. After a Minimum Occupation Period (MOP), usually 5 years, owners can sell them on the open **resale market**.

In 2026, the news about this market seems to contradict itself:

- **The market looks like it is cooling.** HDB resale prices fell for a **second straight quarter** in Q2 2026 (−0.3%, flash estimate). Analysts pointed to a weaker hiring outlook and a "sluggish" market (*The Straits Times*, *CNA*, 1 Jul 2026).
- **Records keep being broken.** In Sep 2026, a 5-room flat at **Pinnacle@Duxton** (Block 1C, floors 19–21) sold for a record **S$1.72M**. It beat a same-sized unit on floors 28–30 in the neighbouring block, which had sold for a record S$1.701M just two weeks earlier. That unit had in turn beaten the S$1.63M record set in May 2026 by a unit on floors 43–45 (*Stacked Homes*, 9 and 25 Sep 2026). The same month, a 5-room flat at **Block 279B Sengkang East Ave** set a Sengkang record of **S$1.18M**, beating a higher-floor unit by more than S$51k (*Stacked Homes*, Sep 2026).
- **Million-dollar flats are no longer rare.** They were 46 sales in 2017 and **1,439 in Jan–Sep 2026**, which is 7.5% of all resales.

This split may reflect a wider **K-shaped economy**, where higher-income households keep getting ahead while others stall. These datasets can't test that on their own, because they contain no income data.

### Datasets

Both datasets come from [data.gov.sg](https://data.gov.sg) and are published by HDB.

| File | Grain | Rows | Description |
|---|---|---|---|
| `data/ResaleflatpricesbasedonregistrationdatefromJan2017onwards.csv` | One row per resale transaction | 241,148 | Month, town, flat type, block, street, storey range, floor area, flat model, lease commencement year, remaining lease, resale price |
| `data/HDBPropertyInformation.csv` | One row per HDB block | 13,357 | Max floor, year completed, what the block contains (residential, commercial, hawker, car park), number of units sold and rented by flat type |

**How they relate:** each block has many transactions. Join on `block` + `street_name` (resale) = `blk_no` + `street` (property). About 99.99% of transactions match a block.

### Things to know before you start

- `month` is the **registration** month, not the date the price was agreed. Recent months may be incomplete.
- The **median price in this data is not HDB's Resale Price Index**. The index adjusts for the mix of flats sold, and the median does not, so results won't exactly match news reports.
- S$1M is a **nominal** figure and is not adjusted for inflation.
- The data does **not** include precise location (e.g. distance to MRT stations or schools), cash-over-valuation (COV) amounts, or buyer profiles.
- Use the **cleaned files** for analysis (see [Data Cleaning](#4-data-cleaning)).

---

## 2. Objectives & Problem Statement

### Problem statement

> **Why are HDB resale records being broken while overall prices are falling?**
>
> In 2026, headline HDB resale prices dipped for two consecutive quarters, yet million-dollar transactions reached a record 538 in Q3 2026. Flats such as Pinnacle@Duxton (S$1.72M) and Sengkang's Block 279B (S$1.18M) set new highs, both on lower floors than previous record holders.
>
> Using 2017–2026 resale transactions joined with block-level attributes, we test whether the market has split into a **premium segment** (young lease, prime or well-connected estates, large units) that keeps rising and a **mainstream segment** that is softening. We also measure how much each factor (lease, location, size, storey) contributes to price today compared with before the 2021 price surge.

### Objectives

1. **Describe the market.** Track resale prices, price per sqm, and the number and share of S$1M+ transactions from 2017 to 2026, by town and flat type.
2. **Find the split.** Check whether prime towns and young-lease flats are moving differently from the rest of the market, and identify when that started.
3. **Explain what drives price.** Measure how much remaining lease, town, floor area, flat type, storey and block features (height, shops or hawker centre downstairs, unit mix) each contribute to resale price.
4. **Test the headline stories.**
   - *Is floor height losing importance?* Compare the price premium per storey band in 2021 vs 2026.
   - *How much is lease worth?* Compare flats with 85+ years left to flats with 60–70 years left, in the same town and flat type, over time.
   - *Where are million-dollar flats spreading?* Track S$1M+ sales outside the prime towns (e.g. Ang Mo Kio, Clementi, Tampines, Sengkang).

---

## 3. Key Stakeholders

### Homebuyers & sellers
*First-time buyers, upgraders, and owners of older flats*

A flat is usually a household's biggest asset, and in a split market the "HDB is booming" story may not apply to *their* flat. This analysis helps them **benchmark a fair price** against similar flats that have actually sold, **weigh the premium for a younger lease**, and **see whether their segment is rising or softening** before deciding to buy or sell.

### Policymakers
*HDB and the Ministry of National Development (MND)*

Keeping public housing affordable is a core policy goal, and broad cooling measures work poorly when segments move in opposite directions. This analysis helps them **track whether the premium segment is pulling away**, **pinpoint the towns and flat types where it's happening**, and **find where lease decay starts to erode value**. That evidence can inform whether targeted measures, such as the Prime Location Public Housing (PLH) model, are needed.

---

## 4. Data Cleaning

Both datasets were profiled and cleaned in [`code/data-cleaning.ipynb`](code/data-cleaning.ipynb). The cleaned data is saved as new files, and the original files are unchanged:

- `data/ResaleflatpricesbasedonregistrationdatefromJan2017onwards_clean.csv` (241,148 rows)
- `data/HDBPropertyInformation_clean.csv` (13,357 rows)

Overall the data is in good shape: there are no missing values, no gaps in months, and 99.99% of transactions match a block. Only a handful of values needed fixing.

### Changes made

| Change | Rationale |
| --- | --- |
| `remaining_lease` text (e.g. "61 years 04 months") → `remaining_lease_months` (number) | Text in 3 formats can't be sorted or averaged, and lease is central to the analysis |
| Block 37 Teban Gardens Rd (Jan 2025): `lease_commence_date` 1981 → **1966** | The block's other 11 sales, and the row's own remaining lease, both point to 1966 |
| 2 sales at **S$150K** set to missing: 522A Tampines Central 7 (Aug 2023) and 182A Woodlands St 13 (Dec 2023) | Each is less than half the price of the same flats in the same block, so they're likely typing errors or non-market transfers. The true price is unknown, so it isn't guessed; the rows are kept for volume counts |
| New `is_terrace` flag (123 sales) | Terrace flats are real but are landed-style units of up to 367 sqm labelled 3 ROOM / 4 ROOM. Exclude them from 3 ROOM / 4 ROOM comparisons |
| Resale data sorted by month | The raw file goes out of date order from 2021 onwards |
| Property Y/N columns → True/False | So they can be counted and filtered directly |

---

## 5. Analysis & Findings

The full analysis is in [`code/analysis.ipynb`](code/analysis.ipynb). Each question is answered with a simple grouped comparison, then checked with a regression.

### How it was done

- **Mix-adjusted price index:** the raw median moves whenever a different mix of flats is sold (e.g. more large flats). To compare like with like, sales are grouped by **town × flat type × lease band**, each group's price per sqm is compared with its own 2017–2019 level, and the groups are combined using fixed 2017–2019 weights. This works like HDB's Resale Price Index, in a simplified form.
- **Prime towns, defined from the data:** the towns above the largest price gap in the 2017–2019 ranking. These are Central Area, Queenstown, Bukit Timah, Bukit Merah, Marine Parade and Bishan.
- **Regression by period:** price per sqm is modelled on town, flat type, lease, storey and block features for **2019** (before the surge), **2021** (start of the surge) and **2025–26** (now), to see how each factor's importance changed.

### Findings

**1. The headline dip is real, but small.** Like-for-like prices peaked in 2025 Q4 and fell in each quarter of 2026, about 1% in total. The raw median didn't show this: it held at about S$630K because more large flats were being sold.

**2. Most of the million-dollar surge is the whole market crossing a fixed line.** Prices are about 55% higher than in 2017–2019, which pushes many ordinary 5 ROOM and EXECUTIVE flats past S$1M. Measured against a line that rises with the market, the share of top-end sales is about 7% today, *lower* than the 10–12% of 2017–2021.

| | 2019 | 2026 (Jan–Sep) |
|---|---|---|
| Sales at S$1M or more | 0.3% | 7.5% |
| Sales above a market-adjusted line | 10.8% | 7.2% |

**3. Since mid-2025 the market has split by lease and size, not location.** From 2025 Q2 to 2026 Q3, like-for-like:

| Rising | Falling |
|---|---|
| Flats with 90+ years left: **+2.0%** | Flats with under 60 years left: **−3.5%** |
| 5 ROOM flats: **+1.3%** | 3 ROOM flats: **−1.9%** |

The lease split holds in both prime and other towns. The record sales all come from the rising segment.

**4. Location matters less; lease matters more.** For the same flat type and lease, prime towns' premium fell from about **50% (2019) to 32–37% (2024–26)**, and outer towns' discounts narrowed. In the regression, town now explains 21 points of price variation (down from 35 in 2019), while lease left explains 18 (up from 12).

**5. Floor height adds less than people think.** Within the same block, floors 25+ sell for only about **5% more** per sqm than floors 4–9, and that hasn't changed since 2019. The Pinnacle@Duxton records were set by successively *lower* floors (49–51 → 43–45 → 28–30 → 19–21), which points to differences between individual units (renovation, facing, view) outweighing height.

> This corrects part of the original problem statement: the premium segment is defined by **young lease and larger flats**, not by prime estates.

---

## 6. Recommendations

| For | Recommendation | Confidence |
|---|---|---|
| **Homebuyers** | **Judge a flat by its lease, not just its town.** A young-lease flat in a non-prime town can hold value better than an old flat in a prime town. | High |
| **Homebuyers** | **Treat a cheap short-lease flat as a risk, not a bargain.** Flats with 60–70 years left are about 20–25% cheaper per sqm than young flats in the same town, and they are a falling segment. | High for the gap; medium for the trend |
| **Homebuyers** | **Don't overpay for height.** Within the same block, high floors are worth only about 5% more than mid-low floors. | High |
| **Sellers of older or smaller flats** | **Don't benchmark against record headlines.** Records come from young-lease 5 ROOM flats. Use recent sales of similar flats in the same town and lease band. | High |
| **Policymakers** | **Focus on lease decay rather than prime locations.** The location premium is shrinking; the gap between young and old leases is where values are diverging. | Medium: the split is recent |
| **Policymakers** | **Read S$1M counts carefully.** Most of the rise is general price growth past a fixed nominal line. Track a market-adjusted measure alongside it. | High |

### Limitations

- **No buyer data** (income, age, household type), so the analysis can't show who is priced out or test the K-shaped economy idea.
- **No location detail** below block, such as distance to MRT stations or schools.
- **The split is recent** (5 quarters) and could reverse, so it should be re-checked as new data comes in.
- **Recent sales are still being registered:** the S$1.72M Pinnacle@Duxton record (Sep 2026) isn't in the data yet.
- **The mix-adjusted index approximates** HDB's Resale Price Index. It matches the direction of the published figures, not the exact values.

---

## Repository Structure

```
├── README.md
├── code/
│   ├── data-cleaning.ipynb   # profile the raw data, apply fixes, save the cleaned files
│   └── analysis.ipynb        # answer the problem statement and make recommendations
└── data/
    ├── ResaleflatpricesbasedonregistrationdatefromJan2017onwards.csv          # raw
    ├── ResaleflatpricesbasedonregistrationdatefromJan2017onwards_clean.csv    # cleaned
    ├── HDBPropertyInformation.csv                                             # raw
    └── HDBPropertyInformation_clean.csv                                       # cleaned
```

---

*Data source: Housing & Development Board, via data.gov.sg. News context: The Straits Times, CNA, Stacked Homes, EdgeProp, Mothership (2026).*
