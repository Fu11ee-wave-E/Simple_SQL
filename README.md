# Simple SQL Queries for Global Temperature Anomaly Analysis (1950 - 2025)

<div align="justify">

## Project Overview
* This short project analyses over 70 years of global surface weather observations to calculate and visualize global climate warming trends. 
* Using **Google BigQuery** and the **NOAA Global Surface Summary of the Day (GSOD)** public dataset, the analysis extracts historical baselines and tracks annual temperature anomalies. 

## Key Insights & Methodology
- **Data Source:** NOAA GSOD Public Dataset (`bigquery-public-data.noaa_gsod`), tracking data from 9,000+ global weather stations.
- **Data Cleansing:** Filtered out erroneous data flags (`999.9` used by NOAA for missing metrics) using SQL conditional functions and converted values from Fahrenheit to Celsius.
- **Analytical Metrics:** Leveraged **SQL Window Functions** to calculate a rolling 10-year historical baseline average. Annual temperature variations are calculated as an **anomaly deviation** from this dynamic benchmark.


## 🛠️ Tech Stack
- **Database Engine:** Google BigQuery (SQL)
- **Environment:** Google Colab / Jupyter Notebooks
- **Libraries:** `google-cloud-bigquery`, `pandas`, `matplotlib`


## 💻 SQL Query 1 : Anomaly Tracking
The following query handles data ingestion, unit transformation, rolling baseline calculation, and anomaly detection entirely inside BigQuery:

```sql
WITH YearlyGlobalAvg AS (    
  SELECT 
    _TABLE_SUFFIX AS year_string,
    CAST(_TABLE_SUFFIX AS INT64) AS year,
    -- Clean the 999.9 error flags and convert F to C
    ROUND(AVG(IF(temp = 999.9, NULL, (temp - 32) * 5/9)), 2) AS avg_temp_celsius
  FROM `bigquery-public-data.noaa_gsod.gsod*`
  WHERE _TABLE_SUFFIX BETWEEN '1950' AND '2025'
  GROUP BY year_string, year
)
SELECT 
  year,
  avg_temp_celsius,
  -- Calculate a 10-year rolling historical baseline average
  ROUND(AVG(avg_temp_celsius) OVER(ORDER BY year ROWS BETWEEN 10 PRECEDING AND 1 PRECEDING), 2) AS baseline_10yr,
  -- Calculate the anomaly (deviation from the baseline)
  ROUND(avg_temp_celsius - AVG(avg_temp_celsius) OVER(ORDER BY year ROWS BETWEEN 10 PRECEDING AND 1 PRECEDING), 2) AS anomaly
FROM YearlyGlobalAvg
ORDER BY year ASC;
```

📊 Visualization

## 💻 SQL Query 2 : Identifying Extreme Weather Hotspots
The following query joins the weather logs with the stations table to see which countries experienced the highest number of extreme heat days (days over 38°C / 100°F).

 ```sql
SELECT 
  meta.country,
  meta.name AS station_name,
  COUNT(*) AS extreme_heat_days
FROM `bigquery-public-data.noaa_gsod.gsod2025` AS weather
JOIN `bigquery-public-data.noaa_gsod.stations` AS meta
  ON weather.stn = meta.usaf AND weather.wban = meta.wban
WHERE weather.temp >= 100 AND weather.temp != 999.9
GROUP BY meta.country, station_name
HAVING extreme_heat_days > 5
ORDER BY extreme_heat_days DESC;
```
📊 Visualization

</div>
