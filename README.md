# 📊 Dynamic Pricing for Urban Parking Lots (Model 2)

**Consulting & Analytics Club × Pathway**  
**Summer Analytics 2025**  
**Author:** Debabrata Paul  
**Model Version:** Model 2 – Contextual Demand-Based Dynamic Pricing

---

## ✅ Objective

The goal of this project is to dynamically price urban parking lots based on **contextual demand** using real-time data streams. This pricing system aims to:

- Optimize utilization of parking infrastructure.
- Reduce congestion in urban environments.
- Adjust prices responsively based on factors like occupancy, traffic, vehicle type, and special conditions.

---

## 🧠 Methodology

### 🔁 Step-by-Step Workflow

#### 1. Data Ingestion
- Loaded historical parking data from `/content/dataset.csv`.
- Combined `LastUpdatedDate` and `LastUpdatedTime` into a single `Timestamp`.

#### 2. Data Preparation
- Cleaned and selected relevant features:  
  `Occupancy`, `Capacity`, `QueueLength`, `TrafficLevel`, `VehicleType`, `IsSpecialDay`, etc.
- Exported the cleaned data to `parking_stream.csv` for Pathway streaming simulation.

#### 3. Schema Definition
- Defined a `ParkingSchema` using `pw.Schema` with appropriate datatypes.

#### 4. Streaming Setup
- Used `pw.demo.replay_csv()` to simulate real-time streaming ingestion at 1000 rows/sec.

#### 5. Feature Engineering
- Parsed `Timestamp` and derived:
  - `t`: Full datetime.
  - `day`: Daily window anchor (`%Y-%m-%dT00:00:00`).
- Created feature mappings:
  - **Traffic Level:** `{"Low": 0.2, "Medium": 0.5, "High": 1.0}`
  - **Vehicle Type:** `{"bike": 0.3, "car": 0.6, "truck": 1.0}`
- Computed:
  - `occupancy_rate = Occupancy / Capacity`

#### 6. Windowing and Aggregation
- Applied **1-day tumbling windows** grouped by `day`.
- Aggregated:
  - `occ_sum`, `queue_sum`, `traffic_sum`, `vehicle_sum`, and `special`

#### 7. Demand Function

```text
norm_demand = (
    1.0 * occ_sum +
    0.3 * queue_sum -
    0.4 * traffic_sum +
    0.5 * special +
    0.3 * vehicle_sum
) / 3.0
```
### 8. 🧮 Dynamic Pricing Formula

```python
price = max(5, min(20, 10 + 10 * norm_demand))
```
### 📈 Bokeh Visualization

The model generates a real-time line chart using **Bokeh** and **Panel**.

| Element        | Description                        |
|----------------|------------------------------------|
| **X-axis**     | Time (daily)                       |
| **Y-axis**     | Price in USD                       |
| **Blue Line**  | Dynamic pricing trend              |
| **Red Dots**   | Actual price points emitted        |

---

### 🔍 Assumptions Made

| Feature          | Assumption                                                                 |
|------------------|----------------------------------------------------------------------------|
| **Occupancy**     | Higher occupancy indicates higher demand.                                 |
| **Queue Length**  | Long queues reflect unmet demand.                                         |
| **Traffic Level** | High traffic may reduce desirability (acts as a congestion disincentive).|
| **Vehicle Type**  | Larger vehicles (e.g., trucks) increase space demand.                     |
| **Special Days**  | Events/festivals increase parking demand.                                 |

---

### 💡 Demand Function Explained

This is a **linear weighted demand model** that uses several contextual features:

- **Occupancy**: Primary driver of demand  
- **Queue Length**: Pressure from waiting vehicles  
- **Traffic Level**: High traffic reduces desirability  
- **Vehicle Type**: Bigger vehicles demand more space  
- **Special Days**: Events increase parking needs  

Each factor is assigned a weight and contributes to `norm_demand`, which is then used to compute the price.

---

### 💸 Price Response to Demand and Competition

#### 📈 Price Increases With:
- Higher **occupancy**
- Longer **queues**
- Presence of **special days**
- More **demand-heavy vehicles** (e.g., `truck`)

#### 📉 Price Decreases With:
- **High traffic** (lowers convenience)

> ⚠️ **Note:** This version does **not** yet include competitor pricing or nearby lot comparison. That can be added in a future **Model 3**.

---

### ✅ Conclusion

Model 2 presents a real-time, **context-aware dynamic pricing** model for urban parking lots using:

- **Pathway** for streaming data processing  
- **Bokeh** for live visualization  
- A **weighted demand model** with real-time responsiveness  

This approach enables fine-grained, adaptive pricing that outperforms simple heuristics or static pricing models.
