# Week 5: Advanced Analytical & Decision-Support Dashboard
## Advanced Data Model

- **Date table**: build a dedicated calendar table (`DimDate`) via `CALENDAR(MIN('Ride_Sharing_Dataset'[Request Time]), MAX('Ride_Sharing_Dataset'[Request Time]))`, mark it as a Date Table (Modeling → Mark as Date Table), and relate it to `Request Time` (1-to-many, single direction). This lets month/quarter/year slicers work correctly instead of relying on the raw date column.
- **Relationship**: `Ride_Sharing_Dataset[Driver ID]` → `Driver_Performance_Analysis_FIXED[Driver ID]`, cardinality One-to-One (confirmed: 50 unique drivers, 50 rides), single-direction filter from Ride table to Driver table.
- **Data quality checks already resolved in Week 4**: no missing values, no duplicate Ride IDs, no negative/zero fares (hence zero cancellations), dates parse cleanly, Driver_Performance file headers were corrected.
- **Separation of logic**: raw columns stay in the two source tables; all derived logic (Ride Status, rate-per-mile, ranks) lives in measures/calculated columns, not in the source files, so the pipeline stays auditable against the Python/SQL outputs from Week 2–3.

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

## Driver Performance Intelligence

- Average Weighted Score (composite: 30% revenue, 25% volume, 25% rating, 20% completion) across all 50 drivers: **25.81 / 100**.
- Top 3 vs. benchmark: **OC1811** (46.83), **ET1257** (45.54), **UB9840** (43.74) — all roughly +18 to +21 points above average, driven mainly by above-average revenue and rating.
- Bottom 3 vs. benchmark: **UN8110** (0.00), **GW0500** (4.60), **CB8894** (7.57) — all combine low revenue with a 1-star rating, not cancellations (since none exist).
- **Important limitation repeated here**: because every driver drove exactly once, this ranks single trips dressed up as "driver performance," not repeatable driver behavior. Flag this explicitly if this ranking is used for incentive decisions — it needs multi-ride driver history before it's reliable for that purpose.


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
## Business Questions — Answered

1. **High demand + high cancellation periods/locations?** Not answerable — 0 cancellations exist in this dataset.
2. **Segments generating the largest share of completed-ride revenue?** Sedan and Bus combined generate 61% of total revenue ($19,388.57 of $31,788.97); since all rides are completed, this equals their completed-ride revenue share.
3. **Customer segments contributing most to rides/revenue?** Not answerable — no Customer ID field exists.
4. **Drivers differing most from benchmark?** OC1811 (+21.0), UN8110 (−25.8) are the largest positive and negative deviations from the 25.81 average weighted score.
5. **High demand, weaker revenue-per-completed-ride locations?** Not meaningfully testable — every pickup location is a unique GPS coordinate (see Week 4 notes), so no location repeats enough to compare demand vs. revenue-per-ride at a location level.
6. **Areas needing further investigation before the final project?** Three: (a) why fare is near-perfectly linear in distance — confirm this isn't a dataset generation artifact before using it for pricing recommendations; (b) the missing Customer ID and cancellation data, which block two of the six core Week 5 questions; (c) the 1-ride-per-driver structure, which limits driver analysis to single-trip snapshots.

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


## Business Recommendations

**Supported directly by this dataset:**
1. **Prioritize Sedan and Bus fleet availability** — they already generate 61% of revenue through volume; small availability gains here move more revenue than equivalent gains in lower-volume vehicle types.
2. **Treat the current fare structure as effectively distance-based ($0.103/mile)** and test rate adjustments through the what-if model above before committing to a pricing change, since the current model does not yet account for demand elasticity.
3. **Re-verify the 1-ride-per-driver structure before using the weighted-score ranking for incentive programs** — the current scores reflect single trips, not sustained performance, and could reward/punish drivers based on one data point.

**Require additional data before acting on:**
4. **Add Customer ID and real cancellation-reason fields to future data collection.** Two of the six core Week 5 business questions (customer segmentation, cancellation hotspots) cannot be answered without this — this is a data-engineering recommendation, not an analytics one.
5. **Collect multi-ride driver history** before deploying any driver ranking into dispatch or incentive systems, since the current dataset cannot distinguish a consistently strong driver from a single lucky trip.
