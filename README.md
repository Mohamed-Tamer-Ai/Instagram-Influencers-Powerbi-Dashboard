# Top Instagram Influencers Dashboard

> An interactive Power BI dashboard analyzing the top 200 Instagram influencers globally,
> uncovering patterns in engagement, follower reach, geographic distribution, and content performance.

---

## Table of Contents

- [Project Overview](#project-overview)
- [Business Problem](#business-problem)
- [Dataset Description](#dataset-description)
- [Data Cleaning Process](#data-cleaning-process)
- [Data Modeling](#data-modeling)
- [Dashboard Features](#dashboard-features)
- [Key Insights](#key-insights)
- [Tools Used](#tools-used)
- [Screenshots](#screenshots)
- [How to Use](#how-to-use)
- [Repository Structure](#repository-structure)
- [Future Improvements](#future-improvements)
- [Author](#author)

---

## Project Overview

This project presents a fully interactive Power BI dashboard built on a dataset of the top 200
most-followed Instagram accounts worldwide. The dashboard was designed to go beyond surface-level
follower counts and answer more meaningful questions: Who actually drives engagement? Which countries
produce the most influential accounts? And does posting volume have any real relationship with likes?

The project was completed as part of a Data Analytics training program and is structured to
demonstrate end-to-end analytical thinking — from raw data ingestion and transformation, through
data modeling and DAX measure creation, to visual storytelling and insight delivery.

---

## Business Problem

Social media marketing teams and brand strategists routinely face the challenge of identifying
the right influencers for campaigns. Raw follower counts are misleading. An account with 400 million
followers but a 0.2% engagement rate delivers far less real impact than one with 50 million followers
and an 8% engagement rate.

This dashboard was built to answer four core business questions:

1. Which influencers have the highest genuine audience engagement, not just the largest reach?
2. How does geographic distribution shape the influencer landscape?
3. Is there a measurable relationship between posting frequency and average likes received?
4. How do influence scores distribute across the top 200 accounts, and what separates elite accounts from the rest?

---

## Dataset Description

| Field | Description | Type |
|---|---|---|
| `rank` | Global ranking (1 to 200) | Integer |
| `channel_info` | Instagram handle / username | Text |
| `influence_score` | Composite influence score (0–100) | Decimal |
| `posts` | Total number of posts published | Integer |
| `followers` | Total follower count | Decimal (M) |
| `avg_likes` | Average likes per post | Decimal (M/K) |
| `60_day_eng_rate` | Engagement rate over the last 60 days | Percentage |
| `new_post_avg_like` | Average likes on the most recent posts | Decimal |
| `total_likes` | Cumulative lifetime likes | Decimal (B/M) |
| `country` | Country of origin | Text |

**Dataset size:** 200 rows, 10 columns  
**Notable stat:** 62 records had missing country values, handled during the cleaning phase.

---

## Data Cleaning Process

The dataset arrived mostly clean but required several transformations before it could be
used reliably in Power BI visuals and DAX calculations.

**1. Numeric Suffix Conversion (Power Query)**

The `followers`, `posts`, `avg_likes`, `new_post_avg_like`, and `total_likes` columns
stored values as text strings with suffixes such as `475.8m`, `3.3k`, and `29.0b`.
A custom column formula was written in Power Query M to convert these into proper numeric values:

```m
if Text.End([followers], 1) = "k"
    then Number.From(Text.Start([followers], Text.Length([followers]) - 1)) * 1000
else if Text.End([followers], 1) = "m"
    then Number.From(Text.Start([followers], Text.Length([followers]) - 1)) * 1000000
else if Text.End([followers], 1) = "b"
    then Number.From(Text.Start([followers], Text.Length([followers]) - 1)) * 1000000000
else Number.From([followers])
```

The same logic was applied to all affected columns.

**2. Handling Missing Country Values**

62 records had blank country fields. These were replaced with `"Unknown"` in Power Query
to prevent null-related issues in the country slicer and geographic map visual.

**3. Data Type Enforcement**

All newly created numeric columns were explicitly cast to `Decimal Number` or `Whole Number`
in the Power Query type step to ensure correct aggregation behavior in visuals.

**4. Engagement Rate Formatting**

The `60_day_eng_rate` column contained percentage strings (e.g., `1.39%`). The `%` character
was removed and the column was stored as a decimal, then formatted as a percentage in the
report layer.

---

## Data Modeling

All transformations were performed in Power Query, resulting in a single clean table loaded
into the Power BI data model. The following DAX measures were created to support the KPI
cards and visual calculations:

```dax
-- Total count of influencers in the filtered context
Total Influencers = COUNTROWS('top_insta_influencers_data')

-- Average followers across filtered influencers
Average Influencers Followers = AVERAGE('top_insta_influencers_data'[followers])

-- Average composite influence score
Average Influence Score = AVERAGE('top_insta_influencers_data'[influence_score])

-- Average likes per post
Avg_likes = AVERAGE('top_insta_influencers_data'[avg_likes])

-- Country with the most influencer representation
Most Country Contains Influencers =
MAXX(
    TOPN(
        1,
        SUMMARIZE(
            'top_insta_influencers_data',
            'top_insta_influencers_data'[country],
            "CountryCount", COUNTROWS('top_insta_influencers_data')
        ),
        [CountryCount], DESC
    ),
    'top_insta_influencers_data'[country]
)

-- Sum of total lifetime likes
Sum of Total Likes = SUM('top_insta_influencers_data'[total_likes])

-- Sum of total posts
Sum of Posts = SUM('top_insta_influencers_data'[posts])
```

---

## Dashboard Features

The dashboard spans two pages and includes the following components:

**Page 1 — Executive Overview**

- 6 KPI Cards displaying: Total Influencers, Average Followers, Average Influence Score,
  Average Likes, Top Country, and Sum of Total Likes
- Scatter Chart plotting follower count against engagement rate to reveal the
  followers-vs-engagement relationship
- Table visual listing influencers with rank, name, country, followers, and likes
- 2 interactive Slicers for filtering by Country and Rank range
- Navigation Action Button linking to Page 2

**Page 2 — Deep Analysis**

- Line & Clustered Column Combo Chart comparing sum of posts against sum of followers
  per influencer, revealing posting frequency patterns
- Pie Chart showing country distribution across the top 200 accounts
- GIS Map (Esri ArcGIS) visualizing the geographic concentration of influencers
- 2 additional Slicers for refined filtering
- Supporting KPI Cards for Sum of Posts and Sum of Total Likes

---

## Key Insights

**1. Follower count and engagement rate are largely uncorrelated.**  
Cristiano Ronaldo leads with 475.8 million followers but records only a 1.39% engagement rate.
Meanwhile, accounts like j.m and thv — with roughly 42 and 49 million followers respectively —
achieve engagement rates of 26.41% and 25.80%. This demonstrates that audience size does not
predict audience responsiveness, and brands optimizing for campaign performance should weight
engagement rate more heavily than follower count alone.

**2. The United States accounts for nearly a third of the top 200.**  
With 66 out of 200 accounts, the US dominates the influencer landscape at 33%. Brazil follows
at 13 accounts (6.5%), and India at 12 (6%). This concentration reflects both the English-language
advantage on the platform and the maturity of the creator economy in North America.

**3. K-pop and entertainment fandoms generate the highest engagement rates on the platform.**  
Five of the top ten most-engaged accounts are affiliated with K-pop artists or youth entertainment
franchises. The pattern is consistent: tight-knit, highly active fanbases drive interaction
metrics that dwarf those of mainstream mega-celebrities, making them disproportionately
valuable for niche or youth-oriented brand campaigns.

**4. Kylie Jenner holds the highest cumulative lifetime likes in the dataset at 57.4 billion,**  
despite ranking second in followers. This is a function of both consistent long-term posting
and an audience that actively engages. Cristiano Ronaldo ranks second with 29 billion total likes,
followed by Zendaya at 20.6 billion — notable given she ranks 23rd in followers, confirming
the engagement-over-reach thesis.

**5. High posting volume does not produce higher average likes.**  
Accounts such as raffinagita1717 (17,500 posts) and several sports pages (10,000+ posts)
post at frequencies far exceeding individual celebrities, yet their average likes per post are
orders of magnitude lower. Accounts like Selena Gomez and Kendall Jenner, with under 2,000
posts each, consistently receive millions of average likes. Content quality and audience
connection matter considerably more than posting frequency.

**6. Influence score does not always reflect engagement performance.**  
The composite influence score ranges from 22 to 93 across the dataset, with an average of 81.8.
Several high-engagement accounts score in the 70s — including Billie Eilish (73, 5.02% engagement)
and Bad Bunny (83, 13.09% engagement) — while some accounts with scores above 85 record
engagement rates below 0.5%. The score appears to weight follower count and post volume
heavily, which partially decouples it from actual audience activity.

---

## Tools Used

| Tool | Purpose |
|---|---|
| Power BI Desktop | Dashboard development, data modeling, DAX |
| Power Query (M Language) | Data transformation and cleaning |
| DAX | Calculated measures and KPI logic |
| Esri ArcGIS (Power BI integration) | Geographic map visualization |
| GitHub | Version control and portfolio hosting |

---

## Screenshots

> Replace the placeholders below with actual screenshots from your dashboard.
> Export images from Power BI via File → Export → Export as Image, or use Snipping Tool.

**Page 1 — Executive Overview**

![Page 1 - Executive Overview](screenshots/page1_overview.png)

**Page 2 — Deep Analysis**

![Page 2 - Deep Analysis](screenshots/page2_analysis.png)

**KPI Cards**

![KPI Cards](screenshots/kpi_cards.png)

**Scatter Chart — Engagement vs Followers**

![Scatter Chart](screenshots/scatter_engagement.png)

**GIS Map — Geographic Distribution**

![GIS Map](screenshots/gis_map.png)

---

## How to Use

**Requirements:** Power BI Desktop (free) — download from [microsoft.com/power-bi](https://www.microsoft.com/en-us/power-bi/desktop)

**Steps:**

1. Clone or download this repository to your local machine.
2. Open `Top_Instagram_Influencers_Dashboard.pbix` in Power BI Desktop.
3. If prompted to locate the data source, point it to `data/top_insta_influencers_data.csv`
   in the repository folder.
4. Use the Country and Rank slicers on Page 1 to filter the dashboard interactively.
5. Click the navigation button on Page 1 to move to the Deep Analysis page.
6. Hover over scatter plot points to see individual influencer details.

> The GIS Map visual requires an Esri ArcGIS account or Power BI Pro license to render fully.
> All other visuals load without any additional account requirements.

---

## Repository Structure

```
instagram-influencers-dashboard/
│
├── Top_Instagram_Influencers_Dashboard.pbix    # Main Power BI file
│
├── data/
│   └── top_insta_influencers_data.csv          # Raw dataset (200 rows)
│
├── screenshots/
│   ├── page1_overview.png                      # Executive overview screenshot
│   ├── page2_analysis.png                      # Deep analysis screenshot
│   ├── kpi_cards.png                           # KPI cards close-up
│   ├── scatter_engagement.png                  # Scatter chart screenshot
│   └── gis_map.png                             # Geographic map screenshot
│
└── README.md                                   # Project documentation
```

---

## Future Improvements

- **Engagement Rate KPI Card:** Add a dedicated card showing average 60-day engagement rate
  across the filtered selection — currently the most analytically important metric missing
  from the KPI row.

- **Engagement Tier Segmentation:** Create a calculated column using DAX `SWITCH()` to classify
  influencers into tiers (Viral, High, Average, Low) based on engagement rate thresholds,
  and use this as a slicer and color legend on the scatter chart.

- **Drill-Through Page:** Build a single-influencer detail page accessible by right-clicking
  any influencer name, showing their full profile metrics in an isolated view.

- **Dynamic Titles:** Replace static chart titles with DAX-driven titles that update based on
  active slicer selections (e.g., "Top Influencers — United States").

- **Replace Pie Chart with Donut Chart:** The country distribution visual would benefit from
  a donut format, which handles multi-category data more readably than a standard pie.

- **Font and Theme Standardization:** Migrate all visuals from the current Comic Sans MS font
  to Segoe UI, and apply a consistent two-color theme throughout the report.

---

## Author

**[Your Name]**  
Data Analytics Student  

[LinkedIn Profile](https://linkedin.com/in/your-profile) |
[GitHub](https://github.com/your-username)

---

*This project was completed as part of a structured Data Analytics training program.
Dataset sourced from publicly available Instagram influencer rankings.*
