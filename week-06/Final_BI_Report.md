# Ride Analytics & Revenue Intelligence Platform: Final Business Intelligence Report

**Programme:** ZYROO Data Analytics Internship, Weeks 2-6
**Tools:** Python (pandas), SQL (SQLite), Power BI Desktop
**Data period:** 2 Jan 2022 to 28 Dec 2024 | **Currency:** USD

---

## 1. Business Problem
A ride-hailing business needs a quick, trustworthy view of how many rides happen, how much revenue they bring in, when demand is highest, how drivers perform, and where service quality needs attention. This project turns a ride-level dataset into an interactive Power BI dashboard and a set of evidence-based recommendations.

Questions answered: How many rides and how much revenue? Which vehicle types and payment methods earn most? When is demand highest? How do drivers compare with a benchmark? Where are riders least satisfied? What would a fare or volume change do to revenue?

## 2. Dataset
| Item | Detail |
|---|---|
| Main file | `Ride_Sharing_Dataset.csv`: 50 rides, 17 columns, one row per ride |
| Key fields | Ride ID, Request Time, Pickup/Dropoff coordinates, Ride Distance, Fare Amount (in $), Payment Method, Driver ID, Vehicle Type, Traffic Condition, Peak Hours, Day of Week, User Rating |
| Driver file | `Driver_Performance_Analysis_FIXED.csv`: 50 drivers, built from the ride file (headers repaired) |
| Not in the data | Customer ID, ride status or cancellation reason, trip costs |

## 3. Data Preparation
- Python/pandas checks for missing values, duplicates, dates, fares, distances and ratings (Section 4).
- The driver file had broken headers (commas inside column names shifted the data). Columns were realigned and checked against the ride file: all 50 drivers matched, with 0 fare and 0 rating mismatches.
- `Longtitude Dropoff` renamed to `Longitude Dropoff`.
- **Ride Status (proxy):** Completed if fare > $0, otherwise Cancelled. No status column exists.
- **Weekday and hour are derived from `Request Time`**, because the stored `Day of Week` column matches the timestamps on only 18% of rows.
- No rows were excluded; all 50 passed every check except the distance and weekday issues below.

## 4. Data Quality Audit
| Check | Result |
|---|---|
| Missing values / duplicate rows / duplicate Ride IDs / duplicate Driver IDs | 0 / 0 / 0 / 0 |
| Invalid dates | 0 |
| Zero or negative fares | 0 (range $72.51 to $1,233.21) |
| Invalid ratings | 0 (integers 1-5) |
| Category consistency | Clean (4 vehicle types, 5 payment methods, 3 traffic levels) |
| **Impossible distances** | **All 50 rides exceed 500 miles (525 to 12,132); unreliable** |
| **Day of Week vs timestamp** | **Only 18% match; weekday derived from timestamp** |
| Pickup locations | 50 unique worldwide coordinates, so no location ranking is possible |

**Documentation correction:** early drafts quoted PKR 6,750,000 revenue, 8,500 rides, PayPal and SUV as leaders, and December as peak month. These did not match the dataset and were replaced by the verified figures in this report.

## 5. Data Model (Power BI)
- **FactRides** (ride data), **DimDriver** (driver scores), **DimDate** (calendar table marked as date table).
- Relationships: DimDate[Date] to FactRides[Request Date], and DimDriver[Driver ID] to FactRides[Driver ID]. Both are one-to-many, single direction. No bidirectional or many-to-many relationships.
- All measures live in one `_Measures` table, grouped in display folders (Core, Time, Segment, Scenario).
- An earlier fan-out bug (Total Rides showed 650 instead of 50) was caused by duplicated data/relationships and fixed in the model.

## 6. Methodology (in simple terms)
1. **KPIs** are calculated with DAX measures and checked against pandas and SQL.
2. **Segments** (vehicle type, payment method, traffic, time band, weekday) are compared on rides, revenue, average fare and rating.
3. **Satisfaction signal:** because there are no cancellations, "low-rated share" (rides rated 1-2 stars) is used as the attention indicator.
4. **Driver score** = 30% revenue + 25% volume + 25% rating + 20% completion, each scaled 0-1. Every driver has one ride, so volume and completion are identical for all, and the score effectively measures revenue and rating only (maximum possible 55, not 100).
5. **Scenarios** use the identity revenue = completed rides x average fare. They are estimates.
6. **Association is not causation:** comparisons show where something happens, not why.

## 7. KPIs
| KPI | Definition | Value |
|---|---|---|
| Total Rides | All ride records | 50 |
| Completed Rides | Fare > $0 | 50 |
| Cancelled Rides | Total minus completed | 0 |
| Total Revenue | Sum of fares | $31,788.97 |
| Average Fare / Revenue per Completed Ride | Revenue / completed rides | $635.78 |
| Average Rating | Mean rating, 1-5 | 2.80 |
| Completion Rate / Cancellation Rate | Share of total rides | 100% / 0% |

**Validation:** pandas, SQL and the driver file agree on rides (50), revenue ($31,788.97), average fare ($635.78) and average rating (2.80).

## 8. Analysis
**Revenue by vehicle type**
| Vehicle | Rides | Revenue | Avg fare |
|---|---|---|---|
| Bus | 15 | $9,756.79 | $650.45 |
| Sedan | 17 | $9,631.78 | $566.58 |
| SUV | 11 | $7,049.18 | $640.83 |
| Motorcycle | 7 | $5,351.22 | $764.46 |

**Revenue by payment method:** Visa $9,297.30 | Debit Card $7,986.17 | Apple Pay $5,330.14 | Cash $5,146.41 | PayPal $4,028.95 (Visa + Debit = 54%).

**Demand (from timestamps):** rides by weekday: Saturday 13, Sunday 9, Wednesday 7, Thursday 6, Tuesday 6, Friday 5, Monday 4. Monthly revenue is volatile (28 of 36 months have rides; $96.15 to $3,109.66; peak June 2022), with no clear seasonal pattern. The Peak Hours flag splits 25 / 25.

**Customer experience:** rating distribution 1 star: 13, 2: 10, 3: 8, 4: 12, 5: 7. 46% of rides (worth $14,780.67, 46.5% of revenue) are rated 1-2.

**Ride segments (not customers):** rides over $700 are 46% of rides and 66.2% of revenue; $300-$700: 38% of rides, 29.6% of revenue; under $300: 16% of rides, 4.2% of revenue.

## 9. Dashboard
Pages: **Executive Overview**, **Demand**, **Revenue**, **Customers** (satisfaction only), **Drivers**, **Operations**, plus drill-through pages (Driver Detail, Segment Detail), a report tooltip, and an archive of the Week 4-5 pages. Slicers: Date, Vehicle Type, Payment Method.

*Screenshots (add after the final Power BI check):*
![Executive Overview](screenshots/week-06/executive-overview.png)
![Demand](screenshots/week-06/demand.png)
![Revenue and scenario](screenshots/week-06/revenue.png)
![Drivers](screenshots/week-06/drivers.png)
![Operations](screenshots/week-06/operations.png)
![Data model](screenshots/week-06/model-view.png)

## 10. Advanced Analysis
**Drivers:** benchmark score 25.81 (median 25.80, std 11.91); 25 of 50 below it. Top: OC1811 (46.8), ET1257 (45.5), UB9840 (43.7). Bottom: UN8110 (0.0), GW0500 (4.6), CB8894 (7.6). Revenue and rating are unrelated (correlation -0.03).

**Where low ratings concentrate (share of rides rated 1-2, overall 46%)**
| Segment | Rides | Low-rated share |
|---|---|---|
| High traffic | 13 | 61.5% |
| Medium / Low traffic | 14 / 23 | 35.7% / 43.5% |
| Night 00-05 | 11 | 27.3% |
| Morning 06-11 | 13 | 53.8% |
| Afternoon 12-17 | 12 | 50.0% |
| Evening 18-23 | 14 | 50.0% |
| Sunday | 9 | 88.9% |
| Sedan | 17 | 52.9% |

Small samples (Sunday has 9 rides): treat as leads, not conclusions. Time bands are an analyst grouping.

**Scenarios (revenue only, estimates; baseline $31,788.97)**
| Scenario | Assumption | Estimate | Difference |
|---|---|---|---|
| Fare +5% | Volume unchanged | $33,378.42 | +$1,589.45 |
| Fare +10% | Volume unchanged | $34,967.87 | +$3,178.90 |
| Completed rides +10% | Fare unchanged | $34,967.87 | +$3,178.90 |
| Fare +10%, volume -9% | Price sensitivity | $31,820.76 | +$31.79 (break-even) |
| Fare +10%, volume -15% | Strong price sensitivity | $29,722.69 | -$2,066.28 |
| Cancellations 5% (stress test) | Cancelled rides earn $0 | $30,199.52 | -$1,589.45 |

A +5% fare breaks even if volume falls under 4.8%; +10% under 9.1%. The cancellation scenario is hypothetical because no cancellations were recorded.

## 11. Findings
**Directly supported by the data**
1. $31.8K revenue came from 50 completed rides at $635.78 average fare.
2. Bus and Sedan generate 61% of revenue; Visa and Debit Card generate 54%.
3. Rides over $700 are 46% of rides but 66.2% of revenue.
4. Average rating is 2.80 and 46% of rides are rated 1-2 stars.
5. Low ratings are highest in high traffic (61.5%), on Sundays (88.9%, 9 rides), in the morning (53.8%) and for Sedan (52.9%).
6. Saturday is the busiest weekday (13 rides); the 18-23 band earns the most ($9,118) but half its rides are low-rated.
7. Driver scores reflect single trips; revenue and rating are unrelated.
8. No cancellations are recorded; 100% completion.

**Hypotheses needing more data:** whether low-rated riders churn, why Sundays and high-traffic rides rate poorly, how demand responds to price, whether low-ranked drivers underperform consistently, and any customer, cancellation or profitability conclusions.

## 12. Recommendations
| # | Recommendation | Action | Supporting finding | More data needed? |
|---|---|---|---|---|
| 1 | Find out why rides are rated 1-2 stars, starting with high traffic, Sundays, mornings, Sedan | Investigate | Finding 4-5 | Yes (feedback reasons, wait times) |
| 2 | Capture Customer ID, real ride status and cancellation reason, validated distance, trip cost | Improve | Data audit | Prerequisite for 8-9 |
| 3 | A/B test any fare change using the break-even rule | Test | Scenarios | Yes (elasticity, costs) |
| 4 | Review driver availability on Saturdays and the 18-23 band | Monitor, test | Finding 6 | Yes (driver counts, longer period) |
| 5 | Protect quality on long high-fare rides and the top vehicle types | Monitor | Finding 2-3 | Yes (valid distances, costs) |
| 6 | Keep card payments reliable | Monitor | Finding 2 | Yes (failure rates, fees) |
| 7 | Hold driver incentives until drivers have multiple rides; rebuild the score | Improve | Finding 7 | Yes (multi-ride history) |
| 8 | Track completion and cancellations with a real status field and alert | Monitor | Finding 8 | Yes (real cancellation data) |
| 9 | Measure repeat rate and retention once Customer ID exists | Investigate, test | Finding 4 | Yes (Customer ID) |

## 13. Limitations
1. Distances are implausible (all over 500 miles), so distance-based findings are unreliable.
2. The stored Day of Week conflicts with timestamps; weekday is derived.
3. No Customer ID: no segmentation, repeat behaviour or lifetime value.
4. No cancellations; status is a fare-based proxy.
5. One ride per driver: rankings are single-trip snapshots.
6. Unique pickup coordinates: no location ranking.
7. Small sample (n = 50): segment results and monthly trends are noisy.
8. No cost data: no profitability claims.

## 14. Future Improvements
- Collect Customer ID, ride status with cancellation reason, validated distance, wait time, driver supply and trip cost.
- Use a larger multi-period dataset with several rides per driver and customer.
- Add location zones (city or area names) in place of raw coordinates.
- Rebuild the driver score using volume and completion history.
- Add a demand-sensitivity test (A/B pricing) to replace the scenario assumptions.
