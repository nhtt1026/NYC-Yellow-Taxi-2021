## Project Summary:

NYC city planners are concerned about uneven travel costs and curbside congestion across neighborhoods. Using the NYC TLC Yellow Taxi 2021 trip records, this project:
- Quantifies “fare fairness” using **average fare per mile** by pickup zone
- Checks whether pricing patterns change between **peak vs off-peak** periods
- Identifies **high-demand pickup/drop-off hotspots and recurring PU×DO corridors** to support taxi-stand placement and time-window curb management.

To make results stakeholder-friendly, zone IDs are enriched with **borough and neighborhood zone names** using the NYC TLC taxi zone lookup table. The corridor heatmaps highlight where demand is **corridor-shaped** (repeated movements between paired zones), which is more actionable for curb planning than zone-only rankings.

**Tools Used:**
- **Databricks (Spark SQL):** primary tool for data cleaning, aggregations, joins (trip table ↔ taxi zone lookup), corridor extraction, and KPI calculations (fare-per-mile, peak vs off-peak, PU×DO counts).
- **Python (PySpark):** used for **data ingestion and table/view setup** (loading the 12 monthly Parquet files, creating the combined 2021 DataFrame, and registering a consistent SQL table/view used across all queries).
- **Databricks Visualizations:** built-in charts for quick stakeholder-ready outputs
    
**Method:**
1. **Ingest + standardize the dataset (2021):** load 12 monthly Yellow Taxi Parquet files and register a consistent SQL source (`yellow_taxi_2021_temp`) for all downstream analysis.
2. **Apply basic validity filters:** exclude invalid records (e.g., `trip_distance <= 0`, `fare_amount <= 0`, missing PU/DO IDs) to avoid distorted averages and misleading corridor counts.
3. **Fare fairness (zone-based):** compute **average fare per mile** by pickup zone and review distribution (histogram) + spread (box-style summary) to identify outliers.
4. **Extreme zones comparison:** extract **Top 10 vs Bottom 10** pickup zones by average fare-per-mile and visualize with bar charts for fast comparison.
5. **Peak vs off-peak check:** split trips into **peak vs off-peak** time buckets (using pickup hour) and compare trip volume + average fare-per-mile between buckets.
6. **Hotspot planning (zone-based):** count trips by pickup and drop-off zones, then identify **Top 10 pickup/drop-off hotspots** for:
   - **Morning rush (7–10AM)**
   - **Evening peak (4–7PM)**
7. **Corridor planning (PU×DO):** compute high-volume **pickup→drop-off pairs**, enrich with zone names, and visualize **PU×DO heatmaps** for morning and evening to highlight recurring corridors that drive curb demand.
   
## Objectives:
- **Measure fare fairness by pickup zone** using average fare per mile (`fare_amount / trip_distance`) and identify unusually high/low zones.
- **Reduce misinterpretation from outliers** by interpreting extreme values in the context of short-distance trips (base fare + time charges dominating small distances).
- **Compare peak vs off-peak patterns**, separating pricing stability from demand concentration effects.
- **Identify rush-hour hotspots (morning 7–10AM, evening 4–7PM)** for both pickups and drop-offs to locate recurring pressure points.
- **Build PU×DO corridor heatmaps (top corridors)** using zone names to reveal the highest-volume movements that drive curb congestion during commute windows.
- **Translate findings into planning recommendations**: fixed stands where demand is persistent, flexible curb rules where demand is time-windowed.

## Key Findings:

### 1) Fare-per-mile varies by pickup zone, but the “fairness concern” is concentrated in a small set of outliers:
Most zones cluster in a reasonable band, while a limited tail of zones shows extreme average fare-per-mile values. These extremes are most consistent with **very short trips** where the base fare and time-based charges dominate, inflating “per mile” metrics.

---

### 2) Peak vs off-peak pricing is broadly stable, but demand is not:
Average fare-per-mile is similar across time buckets, suggesting the primary time-of-day effect is **demand concentration** rather than systematic price inflation. Planning implications therefore skew toward **capacity and curb management**, not pricing policy changes.

---

### 3) Rush-hour hotspots concentrate in a small Manhattan core and repeat across windows:
The same zones recur in top-10 pickup and drop-off lists across commuter peaks, indicating **stable mobility nodes** suitable for targeted taxi infrastructure.  
- Morning rush (7–10AM): pickups are led by **Upper East Side North (236)** and **Upper East Side South (237)**; drop-offs are led by **Midtown Center (161)** alongside **236/237**.
- Evening peak (4–7PM): pickups and drop-offs remain dominated by **236/237** (& nearby Midtown/Manhattan zones).

---

### 4) Corridor heatmaps show demand is corridor-shaped, not just zone-shaped:
Both morning and evening heatmaps surface a dominant, bidirectional Upper East Side corridor (**236 ↔ 237**) & repeated Midtown-linked connections. Evening patterns show higher corridor volumes and more destination variety (expanding to additional Midtown/Upper West Side links), reflecting “after-work” trip purposes beyond commute reversal.

## Recommendations:

### 1) Focus taxi stands on the busiest pickup-to-drop-off routes, not only within single zones:
Many trips repeatedly happen between the same pairs of areas. If only the pickup area has a taxi stand, the drop-off area can still get overcrowded. Placing or managing stands along both ends of these common routes helps reduce curbside congestion where it actually builds up.

---

### 2) Treat the Upper East Side pair (236 & 237) as a coordinated operating cluster during commute windows:
Because demand is consistently high in both directions (and within-zone trips remain meaningful), planning should coordinate stand placement, signage, and enforcement across both zones to prevent spillover stopping and double-parking.

---

### 3) Apply time-window curb management around Midtown pressure points (e.g., 161/162/163) during the morning commute:
Use pickup/drop-off separation, short-stay rules, and clear loading guidance to protect throughput and pedestrian safety where arrivals concentrate.

---

### 4) Use a hybrid approach for the evening peak: fixed stands in the core corridors plus flexible loading rules in adjacent zones:
Evening demand spreads into more varied destinations, so combining stable infrastructure in the top corridors with adaptable curb rules nearby helps absorb spikes without overbuilding permanent capacity.

---

### 5) Design beyond rush hours because off-peak demand is higher overall:
Because OFF_PEAK trips exceed PEAK trips (~17.11M vs ~13.25M), baseline taxi-stand capacity and enforcement should support **full-day demand**, with peak-time rules layered on top for commuter surges.

---

### 6) Validate extreme fare-per-mile zones before proposing pricing interventions:
Check whether outliers persist under robustness filters (e.g., excluding ultra-short trips, comparing median vs mean). This ensures decisions are not driven by metric inflation from small-distance denominators.

## Dataset:
**Data source:** [NYC Taxi & Limousine Commission (TLC) – Trip Record Data](https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page)

**Timeframe:** Full year 2021 (12 months)

**Key fields used:**
  - `PULocationID`, `DOLocationID`
  - `fare_amount`, `trip_distance`
  - `tpep_pickup_datetime`, `tpep_dropoff_datetime`
- **Basic validity filters applied in analysis:**
  - `trip_distance > 0`
  - `fare_amount > 0`
  - non-null pickup/drop-off zone IDs

**Lookup dataset (readability + mapping):** NYC TLC Taxi Zone Lookup  
- Join key: `LocationID` ↔ (`PULocationID`, `DOLocationID`)  
- Adds: `Borough`, `Zone` (for interpretation)

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
 
 ### Top 10 pickup zones:

 <img width="1308" height="1026" alt="image" src="https://github.com/user-attachments/assets/5b7d04db-fe77-4a38-af70-b24a6e1a2b85" />

<img width="2046" height="590" alt="image" src="https://github.com/user-attachments/assets/c0e1a55d-621d-4c4a-9dc2-0ed42eb69aa8" />

- The chart is dominated by an extreme outlier: **EWR – Newark Airport (PULocationID 1)** at **~$2,048 per mile**.
- This does **not** mean taxis “charge $2,048/mile”. It usually happens when a trip has a **very short recorded distance**, so the **starting fare + time/traffic waiting cost** becomes large compared to miles traveled.
- Other high-fare zones (many in **Staten Island**) show a similar pattern: **short trips + congestion/wait time** can make “cost per mile” look high.

### Bottom 10 pickup zones:
 <img width="1460" height="1074" alt="image" src="https://github.com/user-attachments/assets/6f8d82d4-5580-4c2b-8472-a7efdc2647c3" />
 
 <img width="2054" height="602" alt="image" src="https://github.com/user-attachments/assets/cc5e785a-983e-432e-bfa1-a95699d1c633" />

- The lowest zones are much more consistent, around **~$2.9–$4.2 per mile**.
- These areas likely have **longer trips on average**, so the **fixed starting fare** is spread across more distance, giving a lower per-mile cost.
- A few zones have **very small trip counts** (e.g., only 1 trip), so they should be treated as **less reliable** and checked before drawing strong conclusions.

 ### 3) Peak vs Off-peak comparison:

<img width="1346" height="778" alt="image" src="https://github.com/user-attachments/assets/920e05b2-94b2-48e9-8091-57d3df9d82da" />

Trips were split into:
- **PEAK (7–10AM and 4–7PM):** **17,113,798 trips**, average fare-per-mile **$7.5347**
- **OFF_PEAK (all other hours):** **13,254,928 trips**, average fare-per-mile **$7.5262**
  
### Insights:
- The average fare-per-mile is **almost identical** between peak and off-peak. 
  This suggests there is **no clear time-of-day price uplift** in the 2021 Yellow Taxi data when measured using fare-per-mile.
- The main difference is **demand volume**, not pricing: **OFF_PEAK has more trips overall**, while **PEAK still represents a very large share** of city travel during commute windows.  
  This points more toward **capacity and curb management** issues (queues, pickup efficiency) rather than time-based overcharging.

 ### 4) Hotspot Analysis for Taxi Stand Planning (Pickup/Drop-off Demand):

 
This section identifies **where taxis are most frequently picked up and dropped off** to prioritize taxi stands, dedicated curb space, and wayfinding in the highest-demand zones.

---

### 4.1. Highest-volume pickup & drop-off pairs during the day:

<img width="1770" height="1558" alt="image" src="https://github.com/user-attachments/assets/a4d83592-5ded-4861-be76-ee44c90f5a6c" />

Most trips cluster in a few repeat **Manhattan corridors**, especially around the Upper East Side and Midtown.

**Key patterns from the top pairs:**
- **Strong two-way corridor:** **`237 (Upper East Side South) ↔ 236 (Upper East Side North)`** is the highest-volume movement (both directions appear at the top).
- **High “within-zone” trips:** **`237 → 237`** and **`236 → 236`** also rank highly, suggesting many short local trips still competing for curb space.
- **Midtown-linked connections repeat:** Some pairs involving **`161 (Midtown Center)`** and **`162 (Midtown East)`** show up frequently alongside Upper East Side zones.

These recurring corridors are good for **fixed taxi stands** or **consistently managed curb pickup space**, because demand is sustained across the day rather than limited to a single time window.

### 4.2. Morning rush hotspots (7–10AM):

<img width="1366" height="1202" alt="image" src="https://github.com/user-attachments/assets/5841e298-0eb7-4df6-b9f8-b1473ae35481" />

<img width="2090" height="602" alt="image" src="https://github.com/user-attachments/assets/c223e1b4-6370-4916-aeb4-f8cf3296945e" />

**Top pickup zones (7–10AM)** are concentrated in Manhattan residential and commuter-connected areas:

- `236 — Upper East Side North` (304,282)
- `237 — Upper East Side South` (263,793)
- `186 — Penn Station / Madison Sq West` (218,313)
- `141 — Lenox Hill West` (170,297)
- `170 — Murray Hill` (163,671)
- `239 — Upper West Side South` (163,373)
- `238 — Upper West Side North` (160,349)
- `140 — Lenox Hill East` (157,007)
- `142 — Lincoln Square East` (155,550)
- `162 — Midtown East` (151,281)

<img width="1312" height="1198" alt="image" src="https://github.com/user-attachments/assets/389babb5-9cac-4a20-bfe6-849a3fb35e88" />

<img width="2074" height="588" alt="image" src="https://github.com/user-attachments/assets/fb77b808-c23a-4ae3-8a37-3ed9be2fb948" />

**Top drop-off zones (7–10AM)** indicate where people are arriving during commuting hours:

- `161 — Midtown Center` (304,260)
- `237 — Upper East Side South` (266,977)
- `236 — Upper East Side North` (261,453)
- `162 — Midtown East` (212,060)
- `170 — Murray Hill` (178,313)
- `140 — Lenox Hill East` (168,378)
- `163 — Midtown North` (151,281)
- `234 — Union Sq` (128,465)
- `141 — Lenox Hill West` (128,141)
- `230 — Times Sq / Theatre District` (121,483)

**Morning rush (7–10AM) corridor heatmap:**

<img width="2052" height="726" alt="image" src="https://github.com/user-attachments/assets/f2dc06f5-736e-43a5-a198-8acb6fcec8c0" />

#### Insights for city planners:

1) **Demand concentrates in a small Manhattan core, not spread evenly:**

The highest-volume corridors cluster around **Upper East Side North (236)**, **Upper East Side South (237)**, and nearby **Midtown zones (161/162/163)**. This matches the morning hotspot results (top pickup and drop-off zones are overwhelmingly Manhattan) and points to a consistent commuter pattern rather than scattered demand.

2) **Two-way corridors are prominent, showing repeated back-and-forth movement:**

Both **236 → 237** and **237 → 236** appear among the strongest corridors. This is a key operational signal: these are not “drop-only” destinations. Pickup and drop-off pressure exists on both sides, so curb management needs to work in **both directions**.

3) **Short “within-zone” trips remain high during the rush window:**
   
Strong within-zone flows such as **236 → 236** and **237 → 237** indicate meaningful local movement even in peak hours. Planning should not assume everyone is only commuting out of a zone; **local trips still compete for curb space**.

4) **A small number of corridors explains a large share of morning activity:**
   
Focusing the heatmap on the **top corridor pairs** is appropriate for planning because it highlights where targeted interventions will have the largest impact, instead of spreading resources thinly across low-volume routes.

---

### Key takeaway for morning planning:

- **Prioritize managed pickup space along the Upper East Side ↔ Midtown commuter spine, not just individual zones:**  
The demand pattern is corridor-shaped. Stand placement that only targets one zone may not reduce congestion at the paired destination curb.

- **Treat 236 and 237 as one operating cluster during 7–10AM:**  
Because flows are strong in both directions and within-zone trips are high, these zones benefit from coordinated stand placement, signage, and enforcement rather than isolated decisions.

- **Design for reliability, not only capacity:**  
For the highest-volume corridors, practical measures (clearly marked stands, short stopping rules, consistent enforcement) can reduce random curb stopping and improve throughput without major new infrastructure.

### 4.3. Evening peak hotspots (4–7PM):

<img width="1248" height="1204" alt="image" src="https://github.com/user-attachments/assets/b844a7c3-20fb-44cb-a27f-6b25752cd3c1" />

<img width="2070" height="634" alt="image" src="https://github.com/user-attachments/assets/52b581d5-e843-4f62-ab47-e2fe47210df5" />

**Top pickup zones (4–7PM)** show the strongest departure areas in the evening:

- `237 — Upper East Side South` (444,859)
- `236 — Upper East Side North` (381,812)
- `161 — Midtown Center` (376,246)
- `162 — Midtown East` (294,832)
- `142 — Lincoln Square East` (281,367)
- `170 — Murray Hill` (269,075)
- `132 — JFK Airport` (264,431)
- `163 — Midtown North` (255,445)
- `239 — Upper West Side South` (253,660)
- `234 — Union Sq` (247,846)

<img width="1296" height="1202" alt="image" src="https://github.com/user-attachments/assets/735ae13b-62ce-4303-bf2b-f9c15118f779" />

<img width="2096" height="618" alt="image" src="https://github.com/user-attachments/assets/9df6905c-9814-4846-a74f-a9252450a9bf" />

**Top drop-off zones (4–7PM)** show where riders are arriving later in the day:

- `236 — Upper East Side North` (383,631)
- `237 — Upper East Side South` (358,887)
- `239 — Upper West Side South` (275,330)
- `142 — Lincoln Square East` (266,567)
- `141 — Lenox Hill West` (264,939)
- `238 — Upper West Side North` (239,275)
- `48 — Clinton East` (227,166)
- `170 — Murray Hill` (226,602)
- `263 — Yorkville West` (210,387)
- `79 — East Village` (203,580)

**Evening peak (4–7PM) corridor heatmap:**

<img width="2044" height="708" alt="image" src="https://github.com/user-attachments/assets/aa6ce34d-53da-4a33-b452-2835c4b15a5c" />

### Insights for city planners:

1) **Evening demand stays concentrated in Manhattan, but intensifies around a few core corridors:**  

The strongest corridor cells continue to cluster in the same Manhattan core zones (Upper East Side North/ South **236/237**, plus nearby Midtown-linked zones such as **161/162/163**). This matches the evening top pickup and drop-off hotspot lists and suggests a predictable “commute + after-work” pattern rather than dispersed demand.

2) **The Upper East Side pair (236 ↔ 237) remains the dominant corridor, indicating sustained bidirectional pressure:**

Both directions (236 → 237 and 237 → 236) appear among the highest volumes again in the evening. This is important for planning because it signals demand is not “one-way home” only. Curb space needs to handle pickups and drop-offs on both sides of the corridor during the same time window.

3) **Cross-neighborhood links become more visible in the evening, reflecting more varied destinations:**
     
Compared with the morning rush, the evening heatmap shows a broader spread from Manhattan core zones into additional Midtown-adjacent and neighborhood zones (for example, Lincoln Square 142 143 and Upper West Side–related zones 238 239 appearing in the matrix). This suggests the evening peak is not purely a commute reversal; it includes trips to dining, entertainment, and transfers.

4) **A small set of corridors still explains a large share of evening peak activity:**

Even with more destination variety, the highest-volume pairs remain concentrated in a limited corridor set. This supports a **focus on the top corridors** planning approach that the most benefit comes from managing the most repeated PU×DO movements rather than trying to intervene everywhere.

---

### Key takeaway for evening planning:

- **Prioritize evening-managed taxi stands along the Manhattan core corridors, especially 236/237 & Midtown-linked zones:**  
Evening peak activity is corridor-shaped, so placing a stand in only one zone will not fully relieve pressure if the paired destination curb remains unmanaged.

- **Plan 236 & 237 as a coordinated evening cluster, not isolated curb segments:**  
Because flows are consistently high in both directions, stand placement, signage, and enforcement should be coordinated across both zones to prevent spillover stopping and double-parking.

- **Design for variability: combine fixed stands in the top corridors with flexible loading rules in adjacent zones:**  
Evening trips spread into more mixed destinations. A hybrid approach (fixed stands in the core corridors, plus short-term loading/clearway rules nearby) helps absorb demand spikes without overbuilding permanent infrastructure.

## Limitations & Next Steps:

### Limitations:
- **“Fare per mile” can be inflated by short trips:**  
  Very short rides can produce unusually high $/mile because base fare and time-based charges dominate when distance is small. This means extreme outliers should be treated as a *signal to investigate*, not immediate evidence of unfair pricing.

- **Zone granularity can hide street-level pinch points:**  
  Results are aggregated to TLC taxi zones, so recommendations identify *where demand concentrates* but cannot pinpoint the exact curb segments, intersections, or loading locations that cause the bottlenecks.

- **Time windows are simplified and do not account for day-of-week or seasonal patterns:**  
  Morning (7–10AM) and evening (4–7PM) windows capture commute peaks, but they do not separate weekday vs weekend, seasonal effects, holidays, or event-driven spikes.

- **External drivers are not modeled:**  
  Weather, events, transit disruptions, and construction can create short-term surges that a year-aggregated view will smooth out.

- **Corridor heatmaps prioritize the top pairs:**  
  Focusing on the top corridors is appropriate for planning, but lower-volume corridors may still matter for equity, neighborhood coverage, or localized congestion.

---

### Next Steps:
- **Robustness checks for fare fairness (reduce outlier bias):**  
    Add median fare-per-mile by zone, exclude ultra-short trips (e.g., < 0.5 miles), and compare results to reduce the risk of decisions driven by metric inflation.

- **Segment demand by weekday vs weekend + seasonality:**  
  Repeat hotspot and corridor analysis by day-of-week and month to identify whether the “top corridors” are stable year-round or driven by specific periods.

- **Turn corridor heatmaps into a planning shortlist:**  
  Create a ranked “corridor watchlist” that flags: (1) strongest two-way corridors, (2) strongest within-zone flows, and (3) corridors linked to Midtown pressure points, so interventions can be targeted and time-windowed.

- **Add map-based context for planner-friendly visualization:**
  Join TLC taxi zone geometries and visualize the top corridors on a city map (lines connecting PU→DO centroids). This makes the results easier to interpret than a matrix alone.

- **Build a simple forecasting layer for staffing or curb rules:**  
  Train a lightweight model (or time-series baseline) to predict corridor volumes by hour/day. This supports dynamic enforcement and temporary loading rules during predictable surges.

- **Expand to multi-modal context for completeness:**  
  Incorporate other TLC datasets (e.g., FHVs) to measure whether taxi corridor hotspots match or differ from ride-hail patterns, strengthening policy relevance for curb management.
