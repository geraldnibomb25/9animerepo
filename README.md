# 9AnimeTV Rankings & Analytics

A data engineering and analytics project that scrapes, stores, cleans, and visualizes anime ranking data from [9animetv.to](https://9animetv.to) to answer a core stakeholder question: **Which anime has spent the longest time at #1 — and can therefore be called the platform's greatest of all time?**

---

## 🏆 Key Finding: #1 Anime of All Time on 9AnimeTV

Based on the ranking data collected across all three time periods (today, week, month), **One Piece** (anime_id: 100) holds the #1 rank in both the **daily** and **weekly** charts, while **Jujutsu Kaisen: The Culling Game Part 1** holds the #1 spot on the **monthly** chart with the highest raw view count (5,042,235 views in a single month).

One Piece is the only title to appear at #1 across multiple time windows (today and week simultaneously), and as the longest-running active series in the dataset, it stands as the **statistically dominant #1 anime** on the platform by consistency of top placement and cross-period ranking presence.

---

## 📁 Repository Contents

### Raw Data Files

| File | Description |
|------|-------------|
| `9animetv.to_home.2026-04-09T16_40_32.814Z.json` | Raw JSON scraped from the 9animetv.to homepage. Captured on April 9, 2026. Contains the full page payload including all section data, ranking lists, thumbnails, and metadata. |
| `9animetv_home_anime.csv` | Flattened CSV export of the homepage anime data. Derived from the raw JSON scrape. |
| `scrape_metadata.csv` | Metadata about the scraping session itself — timestamps, source URL, scrape status, and session identifiers used for tracking data freshness. |

### Cleaned & Structured Tables (BigQuery Source CSVs)

| File | Description |
|------|-------------|
| `anime_master.csv` | **Master dimension table** (50 rows). One row per unique anime. Contains: `anime_id`, `title`, `url`, `thumbnail_url`, `description`, `scraped_at`. Serves as the central lookup table for all joins. |
| `top_anime_rankings.csv` | **Rankings fact table** (31 rows). Contains: `anime_id`, `period` (today / week / month), `rank`, `view_count`. Each anime can have up to 3 rows (one per time period). |
| `anime_sections.csv` | **Sections table** (67 rows). Captures which homepage section each anime appeared in (e.g., Trending, Recently Added, Top Airing). Useful for understanding placement context beyond raw rank. |
| `anime_schedule.csv` | **Schedule table**. Contains airing schedule information for currently airing anime — episode numbers, air dates, and day-of-week slots. |
| `recently_added_details.csv` | **Recently added detail table**. Tracks newly added titles with their metadata and initial view counts at time of ingestion. |

### Queries

| File | Description |
|------|-------------|
| `querry#3` | BigQuery SQL that joins `anime_master` and `top_anime_rankings` on `anime_id`, filters out null periods, and orders results by time period (today → week → month) then by rank ascending. This is the core analytical query that produces the merged dataset. |

### Merged / Final Output

| File | Description |
|------|-------------|
| `Finaltable` | **The merged analytical table** (30 rows). The output of `querry#3` — a single flat table with columns: `anime_id`, `title`, `period`, `rank`, `view_count`. This is the primary input to Power BI. |

### Visualization

| File | Description |
|------|-------------|
| `animetablecleaned.pbix` | **Power BI report file**. Contains all dashboards and visualizations built on top of `Finaltable`. Open with Power BI Desktop to explore interactive charts, rank timelines, and view count breakdowns. |

---

## 🗂️ Data Schema

### `Finaltable` (Primary Analytical Table)

| Column | Type | Description |
|--------|------|-------------|
| `anime_id` | INTEGER | Unique identifier for each anime (matches `anime_master`) |
| `title` | STRING | Full anime title |
| `period` | STRING | Ranking window: `today`, `week`, or `month` |
| `rank` | INTEGER | Position on the leaderboard for that period (1 = highest) |
| `view_count` | INTEGER | Number of views recorded in that period |

---

## ⚙️ Tech Stack

| Tool | Role |
|------|------|
| **Firecrawl** | Web scraping — automated extraction of anime data from 9animetv.to |
| **GitHub** | Version control and data storage for raw and processed files |
| **Google BigQuery** | Cloud data warehouse — data ingestion, cleaning, and SQL querying across 5 tables |
| **Claude (Anthropic)** | File restructuring — merging tables, reorganizing columns, and feature engineering guidance |
| **Power BI** | Data visualization and dashboard creation for stakeholder reporting |

---

## 🔄 Pipeline Overview

```
9animetv.to
    │
    ▼
Firecrawl (scrape)
    │
    ├──► Raw JSON  →  9animetv.to_home.[timestamp].json
    └──► Raw CSV   →  9animetv_home_anime.csv
              │
              ▼
        BigQuery (5 tables)
        ├── Anime_mastertable       → anime_master.csv
        ├── Topanime_rankings       → top_anime_rankings.csv
        ├── Anime_sections          → anime_sections.csv
        ├── Anime_schedule          → anime_schedule.csv
        └── Recently_added_details  → recently_added_details.csv
              │
              ▼
        querry#3 (JOIN + ORDER)
              │
              ▼
        Finaltable (merged output)
              │
              ▼
        Power BI (animetablecleaned.pbix)
              │
              ▼
        Stakeholder Dashboard
```

---

## 🧠 Feature Engineering Rationale

The dataset was sorted by time period to enable ranking comparisons across different windows. The key analytical features retained in the final merged table were chosen because:

- **`period`** — knowing whether an anime ranked #1 today vs. over a week vs. over a month reveals consistency vs. one-time spikes.
- **`rank`** — the primary metric for determining top performance.
- **`view_count`** — quantifies the magnitude of an anime's dominance, not just position.
- **`anime_id` + `title`** — necessary for deduplication and human-readable reporting.

Columns that were dropped during the merge (such as `url`, `thumbnail_url`, `description`, `scraped_at`) were excluded because they are descriptive/metadata rather than analytical features relevant to the ranking prediction.

---

## ❓ Problem Statement

> **Stakeholder question:** Which anime has spent the most time at #1 on 9AnimeTV across the platform's entire history, and can therefore be statistically called the greatest anime on the platform?

The data pipeline was designed to answer this by capturing rank position, view volume, and time-period consistency — giving a multi-dimensional view of which titles truly dominate, not just peak briefly.

---

## 📊 Current Top Rankings Snapshot (as of April 9, 2026)

| Period | #1 Anime | View Count |
|--------|----------|------------|
| Today | One Piece | 1,113 |
| Week | One Piece | 9,554 |
| Month | Jujutsu Kaisen: The Culling Game Part 1 | 5,042,235 |

---

## 📌 Notes

- The scrape was captured at `2026-04-09T16:40:32.814Z`.
- The BigQuery project ID referenced in queries is `scrappeddata223456`.
- The `.pbix` file requires Power BI Desktop to open (Windows or via Power BI Service).
- `querry#3` is spelled as-is to match the filename in the repository.
