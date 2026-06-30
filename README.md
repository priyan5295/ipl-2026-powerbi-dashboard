# 🏏 IPL 2026 — Power BI Dashboard

An end-to-end Power BI project built on a real, incomplete IPL 2026 dataset — cleaned, modeled, and validated against raw ball-by-ball data to catch and fix inaccurate summary statistics.

---

## 📌 Why this project

Most beginner dashboards use a single clean CSV. This project intentionally used a messy, real-world dataset from Kaggle — 5 separate files with inconsistent quality — to practice the kind of data validation work that comes up on the job.

**Key finding:** the bowling summary file (`bowling_stats.csv`) reported Kagiso Rabada with 13 wickets. Recalculating from the raw ball-by-ball data (`deliveries.csv`) showed the actual figure was **29**. That discovery shaped the rest of the build — every player-level stat in this dashboard is calculated from raw deliveries data, not from the (incomplete) summary tables, and cross-checked against Python/Pandas before being trusted.

---

## 🧰 Tools

Power BI Desktop · Power Query (M) · DAX · Python (Pandas, for validation)

---

## 📁 Dataset

Source: [Kaggle — IPL 2026 Complete Dataset](https://www.kaggle.com/datasets/krishd123/ipl-2026-complete-dataset)

| File | Rows | Role |
|------|------|------|
| `deliveries.csv` | 17,477 | Ball-by-ball data — single source of truth |
| `matches.csv` | 39 | Match results, toss, venue (partial dataset) |
| `batting_stats.csv` | 15 | Top 15 batsmen — reference only |
| `bowling_stats.csv` | 15 | Top 15 bowlers — reference only |
| `fielding_stats.csv` | 15 | Top 15 fielders — used for Total Catches |

---

## 🔧 Data cleaning (Power Query)

- Renamed `match_no` → `match_id` in `deliveries` to enable a clean relationship with `matches`
- Replaced nulls based on logic, not blanket defaults — e.g. `wb_runs` and `wb_wickets` are mutually exclusive by cricket logic (a match is won by either runs or wickets, never both), so those nulls were **kept**, not zeroed
- Fixed malformed values in `best_bowling_figure` (`3--22` → `3/22`)
- Standardized data types across all 5 tables

## 🔗 Data model

```
matches [match_id] ──(1)────(*)── deliveries [match_id]
batting_stats / bowling_stats / fielding_stats → standalone
```

Added a calculated column on `matches`:
```dax
Toss Match Winner = IF(matches[toss_winner] = matches[match_winner], "Won", "Lost")
```

## 📊 DAX approach

All player-level measures (runs, sixes, fours, strike rate, wickets, dot balls) are built directly from `deliveries.csv` using `CALCULATE`, `FILTER`, and `SUMMARIZE` — not from the summary tables, which were found to be incomplete during validation. Strike rate calculations exclude wide deliveries to match the official scoring convention.

---

## 📑 Dashboard pages

**Home** — tournament KPIs (runs, sixes, fours, wickets, fifties, catches), top 5 batsmen and bowlers, win-type breakdown, toss-win-by-team chart with official team colors, date and team slicers.

**Player's Spotlight** — Orange Cap (Vaibhav Sooryavanshi — 776 runs, SR 237.31, 72 sixes) and Purple Cap (Kagiso Rabada — 29 wickets, economy 9.48) profile cards.

**Match Analysis** — centuries and fifties counts, most fours/sixes/dot-balls leaderboards, venue-wise match distribution, individual spotlight cards for Sai Sudharsan and Mohammed Siraj, stage slicer (League / Playoffs / Final).

---

## ✅ Key learnings

- Raw transactional data beats summary tables for accuracy — always validate before trusting an aggregated source
- Null handling needs context: some nulls are missing data, others are valid by logic and shouldn't be filled
- Calculated columns (row-level) and measures (aggregations) solve different problems — using the wrong one breaks visual-level filtering
- Top N visual filters scale better than hardcoding player names into DAX
- Cross-filter direction in relationships determines whether a slicer on one table affects visuals built on another

---

## 📷 Preview

*(screenshots in `/screenshots` folder)*

---

**Author:** K. Sathiya Priyan — [LinkedIn](https://linkedin.com/in/sathiya-priyan-da/)
