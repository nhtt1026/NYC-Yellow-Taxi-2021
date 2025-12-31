## Project Summary:

NYC city planners are concerned about unfair fare variation across pickup zones. Using the NYC TLC Yellow Taxi 2021 dataset, this project quantifies **fare fairness** using **average fare per mile** by pickup zone, evaluates **temporal effects** (peak vs off-peak), and identifies **pickup/drop-off hotspots** to recommend where additional taxi infrastructure (taxi stands, curb pickup zones) should be provisioned.

**Tools Used:**
- **Databricks (Spark SQL):** primary tool for querying, aggregations, and insight generation  
- **Python (PySpark):** used mainly for data ingestion (loading 12 monthly Parquet files and registering a SQL view)  
- **Databricks Visualizations:** bar charts for hotspot comparisons and demand patterns

**Method:**
Load 12 months (2021) → Register as SQL view (yellow_taxi_2021_temp) → Compute fare fairness by pickup zone → Compare top/bottom zones → Repeat for peak vs off-peak → Identify and visualize rush-hour pickup/drop-off hotspots

## Objectives:
City planners need evidence to answer 2 practical questions:
1.	**Fare fairness**: Do riders pay noticeably different rates depending on where trips start (pickup zones)?
2.	**Taxi stand planning**: Where should the city allocate additional taxi infrastructure based on demand hotspots?
This project provides both pricing and demand insights to support policy and infrastructure decisions.

## Key Findings:

### 1) Fare fairness varies substantially by pickup zone:
Average fare per mile (`fare_amount / trip_distance`) differs widely across pickup zones (`PULocationID`). A small number of zones show extremely high values, meaning riders starting in those areas face a much higher effective cost per mile.

- **High outlier examples:** `PULocationID = 1` (**~$2,048/mile**), followed by `84` (**~$293/mile**) and `206` (**~$145/mile**).
- **Low-fare examples:** `PULocationID` values such as `110`, `99`, `105`, `44`, and `5` are much lower (about **$2.8–$4.2/mile**).

These extreme high values are most consistent with short-distance trips in dense areas where the fixed starting fare and waiting-time component become a large share of the total fare.

> Note: This project uses zone IDs (e.g., 236, 237). A zone lookup table can be joined later to convert IDs into neighborhood names for stronger stakeholder readability.

---

### 2) Peak vs off-peak: per-mile pricing is stable, but demand differs:
Citywide, the average fare per mile is essentially the same across time buckets:
- **PEAK:** ~**$7.53/mile**
- **OFF_PEAK:** ~**$7.53/mile**

However, trip volume changes substantially:
- **PEAK trips:** ~**13.25M**
- **OFF_PEAK trips:** ~**17.11M**

This indicates the main temporal effect is **demand concentration**, not time-based price inflation.

---

### 3) Rush-hour hotspots concentrate in a small set of zones:
The same zones repeatedly appear in the top 10 lists for both pickups and drop-offs during commuter peaks, indicating stable “mobility nodes” suitable for taxi stand investment.

**Morning rush (7AM–10AM):**
- **Top pickups:** `PULocationID 236` (**~304,282** trips), `237` (**~263,793** trips)
- **Top drop-offs:** `DOLocationID 161` (**~304,260** trips), `237` (**~266,977**), `236` (**~261,453**)

**Evening peak (4PM–7PM):**
- **Top pickups:** `PULocationID 237` (**~444,859** trips), `236` (**~381,812** trips)
- **Top drop-offs:** `DOLocationID 236` (**~383,631** trips), `237` (**~358,887** trips)

These patterns suggest strong commuter-style flows that repeatedly connect the same core zones. For infrastructure planning, this consistency is more valuable than one-off spikes because it indicates persistent demand across the year.

## Dataset:
**Data source:** [NYC Taxi & Limousine Commission (TLC) – Trip Record Data](https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page)

**Timeframe:** Full year 2021 (12 months)

**Key fields used:**
- `PULocationID` (pickup zone)
- `DOLocationID` (drop-off zone)
- `fare_amount`
- `trip_distance`
- `tpep_pickup_datetime`
- `tpep_dropoff_datetime`
