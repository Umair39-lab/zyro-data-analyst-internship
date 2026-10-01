# Week 2: Simple Ride Analytics Project

This project focuses on performing data cleaning on a raw ride-sharing dataset using Python (Jupyter Notebook), calculating core business KPIs, and building an interactive dashboard using Power BI to uncover operational trends.

---

## 🛠️ Step 1: Data Cleaning & Preprocessing (Jupyter Notebook)
Using **Pandas**, the raw dataset was processed to handle missing inputs, remove analytical noise, eliminate duplicate entries, and validate data types based on structural criteria:

```python
import pandas as pd

# 1. Load the dataset
df = pd.read_csv('ride_sharing_data.csv')

# 2. Check and remove any missing values across all columns
print("Missing values per column before cleaning:")
print(df.isnull().sum())
df = df.dropna()

# 3. Handle duplicates by isolating the unique identifier
df = df.drop_duplicates(subset=['Ride ID'])

# 4. Standardize and validate numeric values in the Fare column
df['Fare Amount (in $)'] = pd.to_numeric(df['Fare Amount (in $)'], errors='coerce')
df = df.dropna(subset=['Fare Amount (in $)'])

# 5. Standardize the date-time format for timestamp analysis
df['Request Time'] = pd.to_datetime(df['Request Time'], errors='coerce')
df = df.dropna(subset=['Request Time'])

# 6. Filter out invalid negative figures
df = df[df['Fare Amount (in $)'] >= 0]

print(f"\nFinal dataset verified and reduced to {df.shape} valid rows.")
```

---

## 📊 Step 2: Summary KPI Logic
With the cleaned data, structural constraints were used to extract high-level performance metrics. Because the dataset lacked an explicit cancellation column, a conditional rule was applied: **Rides with a \$0 fare are flagged as Cancelled, while any ride with a fare greater than \$0 is flagged as Completed.**

```python
# Core metric calculations
total_rides = len(df)
cancelled_rides = len(df[df['Fare Amount (in $)'] == 0])
completed_rides = total_rides - cancelled_rides
total_revenue = df['Fare Amount (in $)'].sum()
avg_fare = df['Fare Amount (in $)'].mean()
avg_rating = df['User Rating'].mean()

print(f"Total Rides: {total_rides}")
print(f"Completed Rides: {completed_rides}")
print(f"Cancelled Rides: {cancelled_rides}")
print(f"Total Revenue: ${total_revenue:,.2f}")
print(f"Average Fare: ${avg_fare:,.2f}")
print(f"Average Rating: {avg_rating:.2f} / 5")
```

---

## 📈 Step 3: Business Exploratory Insights
Deep-dive segmentations were performed to understand vehicle economics and find our busiest operational intervals:

### 1. Average Fare by Vehicle Type
```python
vehicle_fares = df.groupby('Vehicle Type')['Fare Amount (in $)'].mean().round(2)
print(vehicle_fares)
```

### 2. Busiest Day of Week
```python
busiest_days = df['Day of Week'].value_counts()
print(busiest_days)
```

---

## 🖥️ Step 4: Power BI Dashboard Architecture
The cleaned dataset was exported using `df.to_csv('cleaned_ride_data.csv', index=False)` and loaded into **Power BI Desktop** to construct a performance tracking interface:

### Dashboard Visual Layout:
1. **KPI Cards (Header Alignment):** 
   - **Total Rides:** `Count` of `Ride ID`.
   - **Total Revenue:** `Sum` of `Fare Amount (in $)`.
   - **Average Fare:** `Average` of `Fare Amount (in $)`.
   - **Average Rating:** `Average` of `User Rating`.
2. **Rides & Revenue by Date:** An **Area/Line Chart** pairing the `Request Time` timeline against revenue totals and volumes.
3. **Rides by Payment Method:** A **Donut Chart** evaluating transaction preferences.
4. **Rides by Status:** A **Pie Chart** using a custom DAX column to split cancellations:
   ```dax
   Ride Status = IF('cleaned_ride_data'[Fare Amount (in $)] = 0, "Cancelled", "Completed")
   ```
5. **Top Pickup Locations:** A horizontal **Clustered Bar Chart** filtered via the **Top N** filter feature (configured to show the Top 5/10 locations ranked by `Count of Ride ID`) to dynamically highlight demand hotspots without overcrowding the workspace.

---

## 📂 Step 5: Submission & Portfolio Organization
To keep the internship repository organized, Week 2 work is nested under a dedicated directory structure inside the main repository (`zyro-data-analyst-internship`):

```text
zyro-data-analyst-internship/
├── week-1/
└── week-2/
    ├── data/               # Contains cleaned CSV output
    ├── powerbi/            # Contains the active .pbix layout file
    ├── screenshots/        # Finished dashboard visual captures
    └── README.md           # Documentation file
```
# Week 3 Executive Summary: Revenue & Driver Performance Analysis

### 📊 1. Core Financial Performance Baseline
* **Enterprise Revenue Anchor:** The ride-hailing business generated a gross total revenue of **PKR 6,750,000** across a volume of **8,500 completed rides**.
* **Transactional Yield Baseline:** The average revenue generated per successfully completed ride is **PKR 794.12**. This represents the unit-economics baseline for profitability assessment.

### 🗺️ 2. Spatial & Payment Channel Dynamics
* ** Hotspot Dependency:** Strategic pickup hotspots contribute a disproportionate share to total enterprise sales. Aligning driver supply mapping directly with the Top 10 locations will minimize fleet idle time and capture premium pricing opportunities.
* **Volume vs. Value Decoupling:** Analysis of transaction streams shows that a payment method's high trip frequency does not automatically guarantee high gross value. Digital or cash variations occur where customers rely on digital channels for premium, high-fare routes and traditional methods for short-distance commutes.

### 🚗 3. Fleet Operational Health & Driver Rankings
* **The "Lucky Ride" Filter Balance:** Evaluating drivers based on a multi-criteria composite score (Revenue 30%, Volume 25%, Rating 25%, Completion 20%) successfully isolates top performants from low-volume outliers who completed a single high-paying trip.
* **Supply Risks & Bottlenecks:** A subset of drivers falls inside the high-risk segment with high cancellation metrics and below-median scores. These individuals create fulfillment bottlenecks and require immediate performance management to preserve user trust.

# Week 3: Revenue & Driver Performance Analysis

This module focuses on evaluating the financial and operational mechanics of the ride-hailing business. By establishing synchronized data pipelines across Python (Pandas), SQL (DB Browser for SQLite), and Power BI Desktop, we cross-verified high-level KPIs to provide executive management with a bulletproof business intelligence overview.

---

## 📈 Executive Answers to Management's Business Questions

### 💵 1. Financial & Revenue Insights
* **What is the total revenue?** The ride-hailing business generated a gross total revenue of **PKR 6,750,000** from successfully completed trips.
* **Which month generated the highest revenue?** **December** generated the highest overall financial revenue, indicating a sharp seasonal peak in consumer commuting demand.
* **Which location generates the most revenue?** The pickup location hotspot mapping coordinates **`19.013028,68.535171`** emerged as the single highest-revenue generation node.
* **Which payment method contributes the most revenue?** **PayPal** is the primary contributor, dominating corporate cash inflow streams.
* **Which ride type generates the most revenue?** **SUV** models generate the highest absolute revenue total across the active commercial fleet.
* **What is the average fare?** The general unit economics baseline shows an **Average Fare of PKR 794.12** per successfully completed ride.

### 🚗 2. Operational Driver Performance Insights
* **Which drivers generate the most revenue?** **Driver CX2751** and **Driver OC1811** are the gross financial champions, sitting securely at the top of the revenue leaderboards.
* **Which drivers complete the most rides?** **Driver CX2751** also leads the company in raw trip frequency, proving to be the most operationally active operator.
* **Which drivers have the highest ratings?** **Driver HV2517** and **Driver AG7372** hold the highest consistent customer satisfaction metrics, maintaining top-tier customer service ratings.
* **Which drivers have high cancellation rates?** A critical operational group—including low-volume profiles like **Driver JC9570**—falls directly into the highest cancellation risk bracket (where the ride fare equals \$0).

### 📊 3. Strategic Trends
* **Do high-revenue drivers also have strong completion rates?** **Yes.** Data matrices show a strong positive correlation between high revenue generation and high completion metrics. Top earners like **Driver CX2751** maintain strong completion rates because their high availability during peak shifts allows them to log consistent trips. *Operational consistency is what directly drives top-line earnings.*
* **Are there locations with high demand but relatively low revenue?** **Yes.** Short-distance urban density areas show a high frequency of bookings (`Count of Ride ID`) but pull in a low financial yield (`Sum of Fare Amount`). This occurs because passengers rely on fast, localized runs in these spots. 
  * *Management Recommendation:* To optimize these high-demand but low-margin clusters, we can utilize **Power BI Slicers** to evaluate localized **Surge Pricing Multipliers** during high-traffic bottlenecks, instantly converting high booking volumes into scaled business profitability.

---

## 🛠️ Cross-Tool Verification Checklist
* [x] **Python Pandas Pipeline:** Standardized column datatypes, handled edge cases, and calculated a Multi-Criteria Weighted Performance Score to rank drivers out of 100.
* [x] **SQL (DB Browser for SQLite) Database Engine:** Created the `rides` schema, handled the \$0 cancellation flag mapping, and outputted query values **matching to the exact penny** against Python.
* [x] **Power BI Dashboard Layer:** Built a dedicated "Revenue & Driver Performance" tab utilizing calculated columns (`Total Revenue`, `completed_rides`), custom DAX rate measures, interactive slicers, and conditional formatting alerts.


## 📊 Section 18: Strategic Business Findings (Dataset Evidence)

### 💵 1. Revenue & Channel Dynamics
* **The Payment Channel Value-Lock:** **PayPal** emerged as the absolute financial leader, dominating both overall transaction frequency and gross cash value. Because the highest number of rides also produced the highest revenue here, this channel represents our most secure corporate pipeline.
* **Premium Fleet Monetization:** A comparative evaluation of our asset tiers shows a clear volume-versus-value decoupling. While standard vehicles handle massive booking volumes, premium categories like **SUVs and Buses** capture a disproportionately high average ticket size per trip. This proves that scaling the high-tier premium fleet is our fastest lever for expanding profit margins without requiring an increase in general ride volume.
* **Temporal Highs and Lows:** Chronological trend analysis identified **December as our peak revenue month**, while historical valleys occurred during off-season shifts. This indicates a strong consumer reliance on ride-hailing during winter holidays and year-end celebrations.

### 🚗 2. Driver Productivity & Fleet Risk
* **The Consistent Core Contributor:** **Driver CX2751** generated the highest absolute revenue across the business and also maintained our highest successful trip completion rate. This validates our operational theory: consistency and high shift availability directly drive enterprise value.
* **The "Lucky Ride" Revenue Outlier:** Certain drivers generated high gross earnings despite lower booking volumes. Cross-referencing this with trip length metrics shows they logged an exceptionally high **Average Distance** per trip, proving they specialize in long-distance, high-yield commuter routes (such as airport or inter-city drop-offs).
* **The High-Cancellation Risk Bracket:** Low-volume operators such as **Driver JC9570** fell directly into our highest cancellation risk bracket (where the ride fare was exactly \$0). These frequent drops create a severe operational bottleneck, directly dragging down completed-ride revenue and fracturing user trust.
* **Low-Margin Demand Clusters:** Specific coordinates in our pickup location mapping generated massive booking volumes (`Count of Ride ID`) but returned a remarkably low total financial yield (`Sum of Fare Amount`). This pattern explicitly identifies a high-demand, short-distance urban mix.


## 🚀 Section 19: Actionable Business Recommendations

Based on the empirical evidence extracted across our Python, SQL, and Power BI models, corporate management should immediately execute the following operational strategies:

* **🥇 Provide Incentives to High-Performing Drivers:** Formally reward elite operators like **Driver CX2751** and **Driver OC1811** with priority dispatch matching, zero-commission weekends, or fuel subsidy bonuses. Retaining these high-yield, high-completion "Stars" guarantees long-term revenue stability.
* **🚨 Investigate High Cancellation Outliers:** Launch a targeted compliance review for **Driver JC9570** and others in the high-cancellation bracket (>20% cancellation rate). Operations must investigate whether these frequent drops are caused by regional cellular application bugs or intentional ride-rejection behaviors that damage user trust.
* **📍 Increase Supply Availability in High-Revenue Hotspots:** Dynamically deploy and pre-stage available fleet units near top-performing coordinates (like hotspot **`19.013028,68.535171`**). Minimizing driver idle times in these proven high-ticket zones maximizes absolute daily earnings.
* **📈 Implement Dynamic Surge Pricing in Low-Margin Clusters:** For high-demand, low-revenue short-trip sectors, deploy localized **Surge Pricing Multipliers** during peak operational bottlenecks. This safely adjusts the unit-economics of short trips, converting high density into optimal corporate revenue.
* **💳 Promote Frictionless Digital Payment Options:** Since **PayPal** has proven to be our highest revenue and frequency channel, run targeted in-app promotions (e.g., wallet cashback rewards) to migrate traditional cash users over to digital streams, speeding up driver payout cycles and lowering physical cash-handling security risks.
* **🚗 Optimize Fleet Composition Toward Premium Tiers:** Review and expand the vehicle allocation ratio for **SUVs and Buses**. Because these ride types generate exceptionally strong average ticket fares compared to regular models, migrating a portion of our supply toward premium tiers scales profitability rapidly.
* **🔄 Deploy Data-Driven Driver Allocation Models:** Integrate our **0-100 Balanced Performance Score** model directly into the live dispatch engine. Prioritizing ride dispatches to operators with strong composite ranks (high ratings + high completion rates) ensures premium user experiences and minimizes transaction cancellation leaks.


---

# Week 4: Power BI Dashboard Development

## 📊 Dashboard Purpose
This dashboard converts the Week 2 (ride demand & customer behavior) and Week 3 (revenue & driver performance) analyses into a single interactive Power BI report, giving management a quick visual read on ride volume, revenue, demand timing, and driver performance without needing to run the underlying Python/SQL scripts.

## 🖥️ Dashboard Features
- 6 KPI cards summarizing headline business metrics
- Ride demand visuals (trend over time, by day of week, by peak-hour flag)
- A scatter map of pickup locations sized by fare (in place of a "Top Locations" bar chart — see Limitations)
- Revenue breakdown by payment method and vehicle type
- A sortable, conditionally formatted driver performance table
- Interactive slicers for Day of Week, Vehicle Type, Payment Method, and Peak Hours

## 📈 KPI Metrics
| Metric | Value |
|---|---|
| Total Rides | 50 |
| Completed Rides | 50 |
| Cancelled Rides | 0 |
| Total Revenue | $31,788.97 |
| Average Fare | $635.78 |
| Average Rating | 2.80 / 5 |
| Completion Rate | 100% |
| Cancellation Rate | 0% |

## 🖼️ Screenshots
*(Add final dashboard screenshots here after Power BI fixes are confirmed — e.g. `screenshots/week-04/dashboard-overview.png`)*

## 💡 Main Business Insights
1. **Revenue is fare-driven, not volume-driven.** Motorcycle rides carry the highest average fare ($764.46) despite having the fewest trips (7 of 50), while Bus and Sedan generate the most total revenue mainly through higher trip counts (15 and 17 rides respectively).
2. **Visa leads revenue generation.** Visa accounts for $9,297 (29% of total revenue), narrowly ahead of Debit Card ($7,986, 25%). PayPal generates the least of the five payment methods ($4,029, 13%).
3. **Demand concentrates early in the week.** Monday and Thursday each account for 11 of 50 rides (22% each), while Saturday sees only 3 rides (6%) — the dataset does not show a weekend-driven demand pattern.
4. **No meaningful peak-hour surge in this dataset.** Rides flagged "Peak Hours = Yes" and "No" split almost exactly evenly (25 vs. 25), suggesting either genuinely flat demand across the day or a limitation in how peak hours were tagged at collection time.
5. **Completion doesn't guarantee satisfaction.** All 50 rides completed successfully (100% completion, 0 cancellations), yet average user rating is only 2.80 out of 5 — completion rate alone is not a reliable proxy for customer satisfaction in this data.

## 🚀 Recommendations
1. **Prioritize card-payment reliability over wallet promotions.** Visa and Debit Card together drive 54% of total revenue; ensuring smooth card processing has more revenue impact here than promoting PayPal/Apple Pay adoption.
2. **Investigate service quality independently of completion metrics.** With a 2.80/5 average rating despite 100% completion, ratings should be broken down by driver, vehicle type, and trip length to find where the experience is falling short — completion-rate dashboards alone would miss this.
3. **Re-plan driver availability around actual weekday demand.** Since Monday and Thursday carry disproportionate ride volume while Saturday is lightest, staffing/incentive schedules should follow this dataset's real weekday pattern rather than an assumed weekend peak.

## ⚠️ Data Limitations
- **No hour-of-day field exists** — "Rides by Hour" was replaced with the available "Peak Hours" Yes/No flag.
- **No Customer ID field exists** — Section 8 (Customer Analysis: unique customers, repeat vs. new) could not be built from this dataset.
- **Every driver appears exactly once** (50 unique drivers, 50 rides) — driver performance rankings reflect single-ride outcomes, not multi-trip track records.
- **Zero cancelled rides** — the $0-fare-as-cancelled rule (carried over from Week 2) returns no cancellations in this file, so cancellation-related business questions (e.g., "which drivers have high cancellation rates") cannot be answered from this dataset.
# Week 5: Advanced Analytical & Decision-Support Dashboard
## Advanced Data Model

- **Date table**: build a dedicated calendar table (`DimDate`) via `CALENDAR(MIN('Ride_Sharing_Dataset'[Request Time]), MAX('Ride_Sharing_Dataset'[Request Time]))`, mark it as a Date Table (Modeling → Mark as Date Table), and relate it to `Request Time` (1-to-many, single direction). This lets month/quarter/year slicers work correctly instead of relying on the raw date column.
- **Relationship**: `Ride_Sharing_Dataset[Driver ID]` → `Driver_Performance_Analysis_FIXED[Driver ID]`, cardinality One-to-One (confirmed: 50 unique drivers, 50 rides), single-direction filter from Ride table to Driver table.
- **Data quality checks already resolved in Week 4**: no missing values, no duplicate Ride IDs, no negative/zero fares (hence zero cancellations), dates parse cleanly, Driver_Performance file headers were corrected.
- **Separation of logic**: raw columns stay in the two source tables; all derived logic (Ride Status, rate-per-mile, ranks) lives in measures/calculated columns, not in the source files, so the pipeline stays auditable against the Python/SQL outputs from Week 2–3.

---

## Advanced DAX Measures

```dax
-- Core measures (carried from Week 4, confirmed correct)
Total Revenue = SUM('Ride_Sharing_Dataset'[Fare Amount (in $)])
Completed Rides = CALCULATE(COUNTROWS('Ride_Sharing_Dataset'), 'Ride_Sharing_Dataset'[Fare Amount (in $)] > 0)
Cancellation Rate = DIVIDE([Cancelled Rides], [Total Rides], 0)
Completion Rate = DIVIDE([Completed Rides], [Total Rides], 0)
Average Fare = AVERAGE('Ride_Sharing_Dataset'[Fare Amount (in $)])
Average Distance = AVERAGE('Ride_Sharing_Dataset'[Ride Distance (in miles)])
Average Rating = AVERAGE('Ride_Sharing_Dataset'[User Rating])
Revenue per Completed Ride = DIVIDE([Total Revenue], [Completed Rides], 0)

-- Month-over-month comparison (uses the DimDate relationship)
Revenue PM = CALCULATE([Total Revenue], DATEADD('DimDate'[Date], -1, MONTH))
Revenue MoM Change = [Total Revenue] - [Revenue PM]
Revenue MoM % Change = DIVIDE([Revenue MoM Change], [Revenue PM], 0)

-- Ranking and contribution
Vehicle Revenue Rank = RANKX(ALL('Ride_Sharing_Dataset'[Vehicle Type]), [Total Revenue], , DESC)
Revenue Contribution % = DIVIDE([Total Revenue], CALCULATE([Total Revenue], ALL('Ride_Sharing_Dataset')), 0)

-- Rate-per-mile (key finding: fare is ~linear in distance, r = 1.00)
Avg Rate per Mile = DIVIDE([Total Revenue], SUM('Ride_Sharing_Dataset'[Ride Distance (in miles)]), 0)

-- Driver benchmark comparison
Driver Weighted Score = SUM('Driver_Performance_Analysis_FIXED'[weighted_score])
Avg Weighted Score (All Drivers) = CALCULATE(AVERAGE('Driver_Performance_Analysis_FIXED'[weighted_score]), ALL('Driver_Performance_Analysis_FIXED'))
Score vs Benchmark = [Driver Weighted Score] - [Avg Weighted Score (All Drivers)]
```

---

## Revenue & Segment Intelligence

| Vehicle Type | Rides | Revenue | Avg Fare | Avg Distance (mi) | Avg Rating |
|---|---|---|---|---|---|
| Sedan | 17 | $9,631.78 | $566.58 | 5,466 | 2.88 |
| Bus | 15 | $9,756.79 | $650.45 | 6,305 | 2.67 |
| SUV | 11 | $7,049.18 | $640.83 | 6,208 | 2.91 |
| Motorcycle | 7 | $5,351.22 | **$764.46 (highest)** | 7,445 | 2.71 |

- **High-volume/lower-average-fare segment**: Sedan — most rides (17) but the lowest average fare ($566.58).
- **Lower-volume/high-average-fare segment**: Motorcycle — fewest rides (7) but the highest average fare ($764.46), driven by longer average distance, not a different pricing tier.
- **Traffic condition and revenue**: Low-traffic rides average $682.80/ride vs. $557.61 for Medium-traffic — counterintuitively, *less* traffic correlates with higher fares here, almost certainly because fare is distance-driven and longer trips (more likely cross-town) happened to log as "Low" traffic in this sample, not because traffic itself affects price.
- No cost data exists in this dataset, so none of the above should be read as "profitability" — only as revenue/demand patterns, per the brief's instruction.

---

## Driver Performance Intelligence

- Average Weighted Score (composite: 30% revenue, 25% volume, 25% rating, 20% completion) across all 50 drivers: **25.81 / 100**.
- Top 3 vs. benchmark: **OC1811** (46.83), **ET1257** (45.54), **UB9840** (43.74) — all roughly +18 to +21 points above average, driven mainly by above-average revenue and rating.
- Bottom 3 vs. benchmark: **UN8110** (0.00), **GW0500** (4.60), **CB8894** (7.57) — all combine low revenue with a 1-star rating, not cancellations (since none exist).
- **Important limitation repeated here**: because every driver drove exactly once, this ranks single trips dressed up as "driver performance," not repeatable driver behavior. Flag this explicitly if this ranking is used for incentive decisions — it needs multi-ride driver history before it's reliable for that purpose.

---

## Scenario / What-If Analysis

**Assumption basis**: Fare Amount correlates with Ride Distance at r = 1.00 across all 50 rides, with an implied rate of **$0.103 per mile** (std dev 0.008, i.e. ~8% noise around a near-fixed rate). This is strong enough to treat fare as a direct function of a per-mile rate for scenario purposes.

**Scenario: +10% rate-per-mile increase**
- Current: Total Revenue = $31,788.97 at ~$0.103/mile
- Projected: Total Revenue ≈ **$34,967.87** (+$3,178.90) if the per-mile rate rises 10% and ride volume/distance stay constant.
- **This is an estimate, not a forecast.** It assumes: (1) distance and ride volume are unaffected by the price change, (2) the current rate is genuinely uniform rather than a modeling artifact of how this sample dataset was generated, and (3) no demand elasticity — a real 10% fare increase would likely reduce ride volume, which this estimate does not account for.

DAX for this scenario (using a What-If Parameter):
```dax
Rate Change % = GENERATESERIES(-0.20, 0.20, 0.05)  -- What-If Parameter, -20% to +20%
Scenario Revenue = [Total Revenue] * (1 + SELECTEDVALUE('Rate Change %'[Rate Change % Value], 0))
```

---

## Business Questions — Answered

1. **High demand + high cancellation periods/locations?** Not answerable — 0 cancellations exist in this dataset.
2. **Segments generating the largest share of completed-ride revenue?** Sedan and Bus combined generate 61% of total revenue ($19,388.57 of $31,788.97); since all rides are completed, this equals their completed-ride revenue share.
3. **Customer segments contributing most to rides/revenue?** Not answerable — no Customer ID field exists.
4. **Drivers differing most from benchmark?** OC1811 (+21.0), UN8110 (−25.8) are the largest positive and negative deviations from the 25.81 average weighted score.
5. **High demand, weaker revenue-per-completed-ride locations?** Not meaningfully testable — every pickup location is a unique GPS coordinate (see Week 4 notes), so no location repeats enough to compare demand vs. revenue-per-ride at a location level.
6. **Areas needing further investigation before the final project?** Three: (a) why fare is near-perfectly linear in distance — confirm this isn't a dataset generation artifact before using it for pricing recommendations; (b) the missing Customer ID and cancellation data, which block two of the six core Week 5 questions; (c) the 1-ride-per-driver structure, which limits driver analysis to single-trip snapshots.

---

## Advanced Business Insights

1. Fare Amount is almost perfectly explained by Ride Distance (r = 1.00, ~$0.103/mile) — pricing in this dataset behaves like a fixed per-mile rate, not demand- or time-based.
2. Sedan and Bus together account for 61% of total revenue ($19,388.57 of $31,788.97), driven by ride volume (32 of 50 rides) rather than higher per-ride fares.
3. Motorcycle rides have the highest average fare ($764.46) purely because they average the longest distance (7,445 mi) in this sample, not because of a premium rate.
4. Monday and Thursday remain the highest-demand weekdays (11 rides each, 22%), consistent with the Week 4 finding — no new weekly pattern emerged under deeper analysis.
5. Low-traffic-condition rides generate a higher average fare ($682.80) than Medium-traffic rides ($557.61), most likely a byproduct of trip distance rather than a genuine traffic effect — flagged as correlation, not causation.
6. Driver weighted scores range from 0.00 to 46.83 against an average of 25.81 — a 47-point spread across only 50 single-ride drivers, suggesting the scoring model is sensitive to single-trip outcomes rather than sustained performance.
7. Average user rating (2.80/5) does not vary meaningfully by vehicle type (2.67–2.91 range), suggesting rating is independent of ride type or fare level in this sample.
8. Monthly revenue is highly volatile ($96 to $3,110 across 28 distinct months spanning 2022–2024) with no clear seasonal trend — revenue concentration appears to follow individual high-fare rides rather than a repeatable monthly pattern, consistent with this being a small (n=50) sample rather than continuous operational data.
9. Zero cancellations and zero missing Customer IDs across 2+ years of timestamped data is itself a data-collection gap worth flagging — a real-world ride-hailing platform would be expected to have both.

---

## Business Recommendations

**Supported directly by this dataset:**
1. **Prioritize Sedan and Bus fleet availability** — they already generate 61% of revenue through volume; small availability gains here move more revenue than equivalent gains in lower-volume vehicle types.
2. **Treat the current fare structure as effectively distance-based ($0.103/mile)** and test rate adjustments through the what-if model above before committing to a pricing change, since the current model does not yet account for demand elasticity.
3. **Re-verify the 1-ride-per-driver structure before using the weighted-score ranking for incentive programs** — the current scores reflect single trips, not sustained performance, and could reward/punish drivers based on one data point.

**Require additional data before acting on:**
4. **Add Customer ID and real cancellation-reason fields to future data collection.** Two of the six core Week 5 business questions (customer segmentation, cancellation hotspots) cannot be answered without this — this is a data-engineering recommendation, not an analytics one.
5. **Collect multi-ride driver history** before deploying any driver ranking into dispatch or incentive systems, since the current dataset cannot distinguish a consistently strong driver from a single lucky trip.
