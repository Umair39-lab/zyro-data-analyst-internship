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

