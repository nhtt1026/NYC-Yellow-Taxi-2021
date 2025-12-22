## Project Summary:

NYC city planners are concerned about unfair fare variation across pickup zones. Using the NYC TLC Yellow Taxi 2021 dataset, this project quantifies **fare fairness** using **average fare per mile** by pickup zone, evaluates **temporal effects** (peak vs off-peak), and identifies **pickup/drop-off hotspots** to recommend where additional taxi infrastructure (taxi stands, curb pickup zones) should be provisioned.

**Tools Used:**
- **Databricks (Spark SQL):** primary tool for querying, aggregations, and insight generation  
- **Python (PySpark):** used mainly for data ingestion (loading 12 monthly Parquet files and registering a SQL view)  
- **Databricks Visualizations:** bar charts for hotspot comparisons and demand patterns  

## Objectives:
City planners need evidence to answer two practical questions:
	1.	**Fare fairness**: Do riders pay noticeably different rates depending on where trips start (pickup zones)?
	2.	**Taxi stand planning**: Where should the city allocate additional taxi infrastructure based on demand hotspots?
This project provides both pricing and demand insights to support policy and infrastructure decisions.

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

## Approach:
