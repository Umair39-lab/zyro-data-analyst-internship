# Ride Analytics & Revenue Intelligence Platform

A Power BI and Python analytics project on a ride-hailing dataset (ZYROO Data Analytics Internship, Weeks 2-6). It turns ride-level data into an interactive dashboard, validated KPIs, scenario estimates and evidence-based recommendations.

## Overview
- **Data:** 50 rides, 50 drivers, Jan 2022 to Dec 2024, in USD.
- **Goal:** show what happened, why it matters, and what the business should investigate next.
- **Headline results:** $31,788.97 revenue, 50 completed rides, $635.78 average fare, 2.80 average rating.

## Features
- Executive Overview page with KPI cards and key findings
- Demand, Revenue, Customers (satisfaction), Drivers and Operations pages
- Drill-through (driver and segment detail) and report tooltips
- What-if scenario on fare and ride volume (estimates, revenue only)
- Data-quality audit and KPI validation across Power BI, pandas and SQL
- Full written report with limitations and future improvements

## Tools
Python (pandas, matplotlib), SQL (SQLite), Power BI Desktop (DAX, date table, what-if parameters), Git and GitHub.

## Workflow
Raw data -> cleaning and audit (Python) -> validation (SQL) -> data model (FactRides, DimDriver, DimDate) -> DAX measures -> dashboard pages -> advanced analysis and scenarios -> recommendations -> final report.

## Screenshots
![Executive Overview](screenshots/week-06/executive-overview.png)
![Demand](screenshots/week-06/demand.png)
![Revenue](screenshots/week-06/revenue.png)
![Drivers](screenshots/week-06/drivers.png)
![Operations](screenshots/week-06/operations.png)

## Key Findings
- Bus and Sedan generate 61% of revenue; Visa and Debit Card generate 54%.
- Rides over $700 are 46% of rides but 66.2% of revenue.
- 46% of rides are rated 1-2 stars (worth 46.5% of revenue). Low ratings concentrate in high traffic (61.5%), Sundays (88.9%, 9 rides), mornings and Sedan.
- Saturday is the busiest weekday (13 rides); the 18-23 band earns the most ($9,118).
- Driver scores reflect single trips; revenue and rating are unrelated.
- No cancellations are recorded (100% completion).

## Recommendations
1. Investigate the causes of low ratings, starting with high traffic, Sundays, mornings and Sedan.
2. Capture Customer ID, real ride status and cancellation reason, validated distance and trip cost.
3. A/B test any fare change (a +5% fare breaks even if volume drops under 4.8%).
4. Review driver availability on Saturdays and in the evening band.
5. Protect quality on long, high-fare rides and top vehicle types.
6. Keep card payments reliable.
7. Hold driver incentives until drivers have multiple rides.
8. Track cancellations with a real status field.
9. Measure repeat rate and retention once Customer ID exists.

Details and evidence are in [reports/Final_BI_Report.md](reports/Final_BI_Report.md).

## Repository Structure
```text
data/raw, data/processed   datasets
notebooks/                 Python analysis
sql/                       SQL validation queries
powerbi/                   dashboard (.pbix)
reports/                   final report
presentation/              presentation materials
screenshots/               dashboard images by week
archive/                   earlier weekly documentation
```

## Limitations
Distances are implausible (all over 500 miles); the stored Day of Week conflicts with timestamps (weekday is derived from `Request Time`); there is no Customer ID, no cancellation data and no cost data; each driver has one ride; sample size is 50. Scenario results are estimates, and no profitability claims are made.

## How to Reproduce
1. Place `Ride_Sharing_Dataset.csv` in `data/raw/`.
2. Run the notebook in `notebooks/`.
3. Open the `.pbix` file in Power BI Desktop and refresh the data source path.
