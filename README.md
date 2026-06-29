<h1 align="center"> Location Analysis for DRT Services in Gyeonggi Province </h1>
<h3 align="center"> Deriving New Service Zones Using a PySpark-Based Machine Learning Classification Model </h3>  

<img src="https://raw.githubusercontent.com/andreasbm/readme/master/assets/lines/rainbow.png" width="100%">
  
## 📌 Overview

**Ddokbus (똑버스)** is a Demand Responsive Transit (DRT) service introduced by Gyeonggi Province to guarantee mobility rights in areas underserved by conventional public transportation. Unlike fixed-route buses, it operates flexibly based on demand and can be called through the **'Ddokta' app**. 
Ddokbus is only available within **designated service zones** — not everywhere in the province. However, no clear data-driven criteria exist for determining which areas should be designated as service zones. With the population aged 65 and over in Gyeonggi Province reaching 18.2% as of April 2026 — approaching the super-aged society threshold of 20% — the need for flexible, accessible transit for mobility-vulnerable residents is growing.
This project aims to **learn the locational characteristics of existing Ddokbus service zones and identify candidate areas for new service introduction** using a data-driven approach based on 1km × 1km grid units across Gyeonggi Province.

<p align="center">
  <img width="312" alt="image" src="https://github.com/user-attachments/assets/bfd98895-b2cc-4944-820b-16c114a18371" />
</p>

<img src="https://raw.githubusercontent.com/andreasbm/readme/master/assets/lines/rainbow.png" width="100%">


## 🗂️ Repository Structure

```
gyeonggi-drt-location-analysis-pyspark/
│
├── codes/            
│   ├── 01_data_preprocessing-pyspark.ipynb
│   ├── 02_modeling-visualization-pyspark.ipynb
│   └── data_preprocessing-colab.ipynb
├── report/
│   └── final_report.pdf
├── visualization/
│   ├── input-visualization.html
│   └── output-visualization.html
└── README.md
```

<img src="https://raw.githubusercontent.com/andreasbm/readme/master/assets/lines/rainbow.png" width="100%">

## 🛠️ Tech Stack

| Category | Tools |
|----------|-------|
| Distributed Computing | Hadoop (Pseudo-distributed mode), HDFS, PySpark |
| Machine Learning | XGBoost (SparkXGBClassifier), scikit-learn |
| Spatial Data Processing | GeoPandas, Shapely |
| Visualization | Folium (Leaflet.js) |
| Geocoding | Kakao Developers Address-to-Coordinate API |
| Development Environment | Jupyter Notebook, Google Colab, WSL2 (Ubuntu) |

<img src="https://raw.githubusercontent.com/andreasbm/readme/master/assets/lines/rainbow.png" width="100%">

## 📊 Data

> The unit of analysis is the **1km × 1km grid cell across Gyeonggi Province (10,847 cells total)**.  
> Public datasets were mapped to each grid cell by latitude/longitude to construct the feature set.

| Feature | Source |
|---------|--------|
| 1km × 1km Grid (SHP), Elderly population (aged 65+) | [National Geographic Information Institute](https://map.ngii.go.kr) |
| Community service centers | [Public Data Portal](https://www.data.go.kr) |
| Senior welfare facilities, hospitals, pharmacies, markets, restaurants, bus stops, enterprises, factories | [Gyeonggi Data Dream](https://data.gg.go.kr) |
| Ddokbus operation status (target variable) | [Gyeonggi Transportation Corporation](https://www.gtrans.or.kr) |

<img src="https://raw.githubusercontent.com/andreasbm/readme/master/assets/lines/rainbow.png" width="100%">

## ⚙️ Pipeline

```
Collect raw CSV & SHP files locally
        ↓
Copy to Ubuntu (WSL2) → Load into HDFS
        ↓
PySpark-based Preprocessing
  · spark.read.csv → .toPandas() conversion
  · Standardize lat/lon column names across all datasets
  · Geocode address-only records using Kakao API
  · Generate 1km × 1km grid & aggregate features per cell
        ↓
XGBoost Binary Classification
  · bus=1 (existing service zone) / bus=0 (no service)
  · Compute predict_proba for candidate ranking
        ↓
Folium-based Map Visualization
```

<img src="https://raw.githubusercontent.com/andreasbm/readme/master/assets/lines/rainbow.png" width="100%">

## ⚠️ Troubleshooting

- **Datanode abnormal termination** — Resolved by modifying the `.Go` initial startup script to fix cluster ID mismatch
  ```.Go
  # Stop all services
  stop-all.sh
  
  # Remove temporary files only
  rm -rf /tmp/hadoop-ubuntu
  
  # Match datanode VERSION to namenode clusterID
  sudo sed -i 's/clusterID=.*/clusterID=CID-d705fcba-73f9-4a95-b6fa-a72d1292288f/' /data/HDFS/datanode/current/VERSION
  
  # Restart
  start-all.sh
  hadoop dfsadmin -safemode leave

  # Verify
  jps
  ```

- **OOM (Out of Memory) error** — Occurred when processing a 14MB restaurant CSV. Spark consumes 2–3GB of memory by default regardless of data size, which exceeded the capacity of the local environment (8GB RAM, 4-core CPU running in pseudo-distributed mode). Resolved by migrating the preprocessing step to Google Colab

<p align="center">
<img width="400" alt="image" src="https://github.com/user-attachments/assets/8b42c950-017e-41b9-8a0e-c7eec44ace55" />
</p>

<img src="https://raw.githubusercontent.com/andreasbm/readme/master/assets/lines/rainbow.png" width="100%">

## 📈 Results

### Model Performance

| Metric | Value |
|--------|-------|
| Accuracy | 0.7866 |
| Precision | 0.7756 |
| Recall | 0.7865 |
| F1-score | 0.7291 |
| ROC-AUC | 0.6614 |

### Feature Importance

| Rank | Feature | Importance |
|------|---------|------------|
| 1 | Elderly population | 0.1943 |
| 2 | Number of enterprises | 0.1462 |
| 3 | Number of factories | 0.1135 |
| 4 | Number of senior centers | 0.1130 |
| 5 | Number of bus stops | 0.0839 |

The results show that Ddokbus placement is driven not only by population size, but by a **combination of elderly mobility demand and industrial/daily-life facility-based travel demand**.

### Top 10 Candidate Zones (by predict_proba)

| Rank | GRID_CD | Pred. Prob. | Elderly Pop. | Senior Centers | Restaurants | Bus Stops | Enterprises | Factories |
|------|---------|-------------|-------------|----------------|-------------|-----------|-------------|-----------|
| 1 | 다사7351 | 0.8879 | 9955 | 4 | 219 | 4 | 0 | 0 |
| 2 | 다사7563 | 0.8678 | 0 | 0 | 0 | 0 | 33 | 10 |
| 3 | 다사7928 | 0.8600 | 0 | 0 | 0 | 0 | 20 | 3 |
| 4 | 다사6582 | 0.8545 | 0 | 0 | 13 | 3 | 13 | 3 |
| 5 | 다바7987 | 0.8444 | 0 | 3 | 33 | 4 | 205 | 3 |

### Final Candidate Zones

Since a real service zone cannot be determined by a single grid cell alone, candidate zones were evaluated beyond prediction probability — considering distance from existing service areas, spatial clustering of high-probability cells, and satellite imagery of the surrounding environment. **Two clustered candidate zones** were selected as final recommendations.

- **Fig. 1** — A zone with a large elderly population and dense daily-life facilities (hospitals, restaurants). High potential for short-distance mobility demand among older residents.
- **Fig. 2** — A zone where factories and small businesses are scattered around hilly terrain. Geographic constraints make conventional fixed-route transit difficult, making DRT a strong fit.

<p align="center">
<img width="450" alt="image" src="https://github.com/user-attachments/assets/807b9b69-f2ac-4be6-9352-5a6d7ddcc5c6" />
</p>

<img src="https://raw.githubusercontent.com/andreasbm/readme/master/assets/lines/rainbow.png" width="100%">

## 🗺️ Visualization

> XGBoost prediction results are visualized on an interactive map of Gyeonggi Province.  
> - 🔵 Blue: Existing Ddokbus service zones  
> - 🔴 Red (intensity): Predicted probability for new candidate zones (darker = higher probability)
>
> 👉 **[View Interactive Map](https://nayeongpark.github.io/gyeonggi-drt-location-analysis-pyspark/visualization/output-visualization.html)**  
> *(Hover over each grid cell to see feature values such as elderly population and bus stop count)*

<img src="https://raw.githubusercontent.com/andreasbm/readme/master/assets/lines/rainbow.png" width="100%">

## 📄 Report

The final project report is available in the [`report/`](./report/) folder.

<img src="https://raw.githubusercontent.com/andreasbm/readme/master/assets/lines/rainbow.png" width="100%">

## 💡 Key Takeaways

- Hands-on experience showed that PySpark preprocessing is **heavily dependent on the execution environment**. The OOM error made clear why cloud infrastructure scalability matters in practice.
- **Infrastructure design and environment optimization** are just as critical as code quality in data engineering workflows.
- Model predictions alone are insufficient for final decisions — **prediction probability, spatial clustering, and real-world site context** must all be interpreted together.
