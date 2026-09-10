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
