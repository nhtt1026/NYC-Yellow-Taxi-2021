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

## Recommendations:

### 1) Prioritise taxi stands and curb pickup space in the highest two-way demand corridor (Zones **236 ↔ 237**):
Provision additional **taxi stands**, **dedicated curb pickup space**, and **clear wayfinding** in zones **236** and **237**. Because demand is consistently high in both directions, this location is likely to deliver the largest operational benefit like reduced curbside congestion, improved pickup flow, or better driver turnover.

### 2) Treat **Zone 161** as a commuter “pressure point” & Improve peak-hour drop-off handling:
Around Zone **161** during morning rush, consider operational controls such as:
- designated taxi loading zones (pick-up/drop-off separation),
- short-stay holding areas,
- signage and lane markings to reduce double-parking and bottlenecks.
This is especially relevant where the same zones dominate both pickup and drop-off activities. Implement these **drop-off prioritisation and curb separation** will have reduce bottlenecks and improve pedestrian safety around these heavy commuter movements.

### 3) Use time-based curb rules for recurring peak hotspots (Zones **162, 170, 142, 163, 234, 239**):
Instead of building permanent taxi infrastructure everywhere, apply **peak-hour curb management** in zones that repeatedly appear in the top 10 hotspot lists. This targets demand when it is highest, while keeping curb space available outside peak periods.

**Recurring hotspot zones:**
- **Morning rush pickups (7–10AM):** `236`, `237`, `186`, `141`, `170`, `239`, `238`, `140`, `142`, `162`  
- **Evening peak pickups (4–7PM):** `237`, `236`, `161`, `162`, `142`, `170`, `132`, `163`, `239`, `234`  
- **Evening peak drop-offs (4–7PM):** `236`, `237`, `239`, `142`, `141`, `238`, `48`, `170`, `263`, `79`  

**Recommended Actions:**
- time-window taxi loading zones (7–10AM and 4–7PM)
- pickup vs drop-off separation
- short-stay holding rules to reduce double-parking and curb conflicts

This approach improves traffic flow and passenger safety during peak periods, without permanently removing curb capacity for other uses.

### 4) Investigate extreme fare-per-mile outliers before proposing pricing policy changes:
Fare-per-mile differs sharply by pickup zone:
- **High outliers:** `PULocationID = 1` (**~$2,048/mile**), then `84` (**~$293/mile**) and `206` (**~$145/mile**)
- **Low-fare zones:** `110`, `99`, `105`, `44`, `5` (about **$2.8–$4.2/mile**)

These extremes are likely driven by **very short trips**, where the base fare and time-based charges dominate while distance is small.
 
Before intervention, validate whether the pattern persists after robustness checks such as:
- compare **median vs mean** fare-per-mile by zone,
- exclude **ultra-short trips** (e.g., below a small distance threshold),
- reviewing the distribution of trip distances in the outlier zones.

This prevents policy decisions being driven by metric inflation from short-distance trips.

### 5) Plan operations beyond rush hours because off-peak demand is higher overall:
Trip volume is higher outside the defined peak windows:
- **OFF_PEAK:** **~17.11M** trips  
- **PEAK:** **~13.25M** trips

Meanwhile, **average fare-per-mile is stable (~$7.53)** across both buckets.

Ensure taxi-stand capacity planning and enforcement is designed for **full-day demand**, not only rush hours. Peak-time rules should be complemented by steady baseline infrastructure where demand remains consistently high across the day.

### 6) Next Step: Extend the analysis using zone names and corridor heatmaps:
For planning use, join a taxi zone lookup table to convert zone IDs to neighborhood/area names, and add a PU×DO heatmap for the highest-volume corridors. This would make recommendations easier for city planners and the public to interpret.

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

## Visualizations / Outputs:
### 1) Average fare per mile by pickup zone

<img width="1322" height="1126" alt="image" src="https://github.com/user-attachments/assets/d77dc7a1-2ecb-4b1f-a0d5-98192fe6ecf3" />

First, I calculated **average fare per mile** (`fare_amount / trip_distance`) for each **pickup zone (PULocationID)** and enriched it with **PUBorough** and **PUZone** to make the results readable.

- The results show **large variation across pickup zones**, meaning passengers can face very different “effective cost per mile” depending on where a trip starts.
- The highest values are concentrated in a small number of zones, with **Newark Airport (EWR, PULocationID = 1)** appearing as an extreme outlier (~**2048 per mile**). This pattern is most consistent with **very short trips** where dividing by a small distance inflates the per-mile metric.
- In contrast, many zones sit at much lower and more stable averages, suggesting that **fare fairness is generally consistent for most pickup locations**, with unfairness concerns mainly driven by a limited set of atypical zones.

---
To understand whether these differences are citywide or driven by a few zones, I visualised the **distribution of zone-level average fare-per-mile** and computed a box-style summary.

### Histogram (zones per band):

<img width="2056" height="638" alt="image" src="https://github.com/user-attachments/assets/7a54ed2a-429d-4b7d-b42b-7e1ff8a728c1" />

- `< 5`: 12 zones
- `5–10`: 136 zones
- `10–20`: 87 zones
- `20–50`: 18 zones
- `50–100`: 6 zones
- `100+`: 4 zones

### Box-plot summary:
<img width="1446" height="1092" alt="image" src="https://github.com/user-attachments/assets/df0855bb-6c33-4227-a459-ae99aba74155" />

<img width="1584" height="1584" alt="image" src="https://github.com/user-attachments/assets/d3350fb8-4bd0-453b-b59d-a0dfc3d6fcc1" />

- Median (typical zone): ~**9.41 per mile**
- Upper fence (outlier threshold): ~**20.92 per mile**
- Zones above upper fence: **27 zones**

### Insights:
- Most pickup zones cluster in the **5–20 per mile** range, indicating a broadly consistent pricing pattern for the majority of the city.
- The “fairness concern” is primarily concentrated in a **small right-tail of outlier zones** (above ~20.92), which should be prioritised for deeper review (e.g., checking whether outliers persist after filtering out ultra-short trips).

 ### 2) Top 10 vs Bottom 10 pickup zones by average fare-per-mile:

 <img width="1308" height="1026" alt="image" src="https://github.com/user-attachments/assets/5b7d04db-fe77-4a38-af70-b24a6e1a2b85" />

 <img width="1460" height="1074" alt="image" src="https://github.com/user-attachments/assets/6f8d82d4-5580-4c2b-8472-a7efdc2647c3" />



