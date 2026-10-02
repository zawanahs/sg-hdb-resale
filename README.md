# HDB Resale Prices: A Market Split in Two?

An analysis of Singapore HDB resale transactions (Jan 2017 – Sep 2026) that looks at why record prices keep being set while the overall market softens.

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

### Kept as-is (keep in mind during analysis)

- **318 exact duplicate rows** (mostly 2017): they may be real sales of identical neighbouring flats, so they weren't removed.
- **67 prices include cents**, mostly "lucky" `.88` endings (e.g. S$1,388,888.88). These are real, so prices are kept as decimals.
- **3 sales at Block 82 Macpherson Lane** have no matching block, so use a left join to keep them.
- **2026 is a partial year (Jan–Sep)**, and **Apr–May 2020** volumes are very low (COVID-19 circuit breaker). Compare quarters or matching periods.
- **Small groups**, such as 1 ROOM (88 sales), MULTI-GENERATION (90) and Bukit Timah (588), give unreliable medians.

---

*Data source: Housing & Development Board, via data.gov.sg. News context: The Straits Times, CNA, Stacked Homes, EdgeProp, Mothership (2026).*
