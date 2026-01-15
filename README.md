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

### 1) Fare fairness varies substantially by pickup zone (with a small set of extreme outliers):
Average fare-per-mile differs widely across pickup zones. The highest values are concentrated in a small number of zones and are most consistent with **very short-distance trips** in dense areas, where the fixed base fare and time-based components dominate the per-mile metric. 

---

### 2) Peak vs off-peak: per-mile pricing is stable, but demand differs materially:
Citywide, average fare-per-mile is effectively the same across time buckets (~$7.53/mile), suggesting **no strong time-of-day price uplift** in fare-per-mile terms. The main temporal effect is **volume**:  
- **PEAK trips (7–10AM and 4–7PM): ~13.25M**  
- **OFF_PEAK trips (all other hours): ~17.11M**  

This points to **capacity + curb management** as the main operational issue, not time-based overcharging.

---

### 3) Rush-hour hotspots concentrate in a small Manhattan core and repeat across windows:
The same zones recur in top-10 pickup and drop-off lists across commuter peaks, indicating **stable mobility nodes** suitable for targeted taxi infrastructure.  
- Morning rush (7–10AM): pickups are led by **Upper East Side North (236)** and **Upper East Side South (237)**; drop-offs are led by **Midtown Center (161)** alongside **236/237**.
- Evening peak (4–7PM): pickups and drop-offs remain dominated by **236/237** (plus nearby Midtown/Manhattan zones).

## Recommendations:

### 1) Prioritise taxi stands and managed curb pickup space along the highest two-way demand corridor (236 ↔ 237):
The strongest recurring flows and the corridor heatmaps point to **Upper East Side North (236)** and **Upper East Side South (237)** as a high-leverage corridor for reducing curb friction. Focus on **clear stand placement, wayfinding, and active enforcement** to reduce double-parking and random curb stops.

---

### 2) Treat Midtown Center (161) as a commuter “pressure point” in the morning and manage drop-off friction:
Morning drop-offs concentrate strongly around Midtown (especially **161**), suggesting peak-hour congestion risk from drop-offs competing with pickups and through-traffic. Implement **pickup vs drop-off separation**, short-stay loading rules, and signage/lane markings near key Midtown arrival points to improve throughput and pedestrian safety.

---

### 3) Use time-window curb rules for recurring peak hotspots, building “surge operations” and not citywide permanent buildout:
Rather than placing permanent infrastructure everywhere, apply **peak-hour curb management (7–10AM, 4–7PM)** in zones that repeatedly appear in hotspot lists. This targets demand when it is highest while preserving curb flexibility outside peak periods.

---

### 4) Plan corridor-first (PU×DO) interventions, not only zone-first interventions:
Heatmaps show demand clusters into a limited set of corridors. For the highest-volume corridors, small operational improvements (consistent enforcement, clear loading rules, curb signage) can reduce congestion without large construction. Place and manage stands where they relieve **both ends of the corridor** (origins and destinations), not only at the “top pickup zone.”

---

### 5) Design beyond rush hours because off-peak demand is higher overall:
Because OFF_PEAK trips exceed PEAK trips (~17.11M vs ~13.25M), baseline taxi-stand capacity and enforcement should support **full-day demand**, with peak-time rules layered on top for commuter surges.

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
