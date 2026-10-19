# Ottawa-EV-Charging-Analysis

EV charging analysis project focused on data cleaning, usage patterns, forecasting, infrastructure characteristics, and charging-demand insights for Ottawa.

## Project Overview

Electric vehicle adoption is increasing the importance of reliable, accessible, and well-utilized public charging infrastructure.

This project analyses publicly available **Electric Vehicle Charging Stations Usage Data** from the **City of Ottawa Open Data platform**, accessed through the City's ArcGIS REST API.

The original usage dataset contains **22,455 charging-session records and 25 columns**, including charging-session dates and times, charging duration, site and station information, energy delivered, session end reasons, and geographic coordinates.

The project also explores separate station, port, and plug infrastructure datasets to better understand charger capacity, charging level, network operators, power ratings, and pricing structures.

## Project Objective

**Evaluate the utilization and performance of Ottawa's public EV charging network, identify differences across sites and stations, and determine whether historical charging-session data can support near-term demand forecasting.**

## Core Business Questions

1. **How is EV charging demand changing over time?**
2. **Which sites and stations are the busiest, and which appear underutilized?**
3. **What does a typical charging session look like in terms of duration and energy delivered?**
4. **What factors appear to explain differences in performance across sites and stations?**
5. **Can historical charging-session data be used to forecast near-term charging demand?**

## Repository Structure

### `01_Data_Acquisition_and_Cleaning.ipynb`

Data acquisition, validation, cleaning, and feature engineering.

Key steps include:

- Retrieve City of Ottawa charging-session data through the ArcGIS REST API
- Inspect schema, missing values, identifiers, timestamps, and coordinates
- Investigate inconsistent session end-date fields
- Reconstruct reliable session timestamps
- Create session duration and time-based features
- Investigate zero-duration and extreme observations
- Standardize address information
- Validate cleaned records
- Export cleaned CSV and Parquet datasets

### `02_EV_Charging_Analysis.ipynb`

Core exploratory analysis, statistical analysis, forecasting, and financial estimation.

Topics include:

- Charging demand over time
- Site and station performance
- Session duration and energy distributions
- Duration vs energy correlation
- Quartiles and outlier analysis
- Day-of-week and hour-of-day usage
- Charging rate and session-duration patterns
- Charging end reasons
- Chi-square testing
- Weekly demand forecasting
- Baseline, Linear Regression, and ARIMA model comparison
- Estimated charging revenue and electricity cost

### `03_Infrastructure_Exploration.ipynb`

Exploration and validation of station, port, and plug infrastructure data.

Topics include:

- Canada and Ottawa station filtering
- Station / port / plug relationship validation
- Actual port-capacity analysis
- Charger levels and connector types
- Charging power, voltage, and amperage
- Level 2 vs DC Fast infrastructure
- Charging network operators
- Pricing structure categorization
- Location-name and site-context exploration

## Key Findings

### Charging Demand

Charging demand increased substantially over the study period.

Weekly charging activity shows a clear long-term upward trend, although individual weeks remain variable.

Usage also varies by day of week:

- **Saturday recorded the highest number of charging sessions**
- Weekend charging activity was generally stronger than early-week demand
- Total energy delivered followed a similar weekly pattern

### Typical Charging Session

The median charging session was approximately:

- **2.0 hours**
- **10.28 kWh**

Session duration and energy delivered were strongly related:

**Pearson correlation: `r ≈ 0.87`**

Longer sessions generally delivered more energy, although substantial variation remained between individual sessions.

### Outliers

Using the 1.5 × IQR rule:

- Duration upper outlier threshold: approximately **7.56 hours**
- Energy upper outlier threshold: approximately **46.83 kWh**
- Duration outliers: **1,666 sessions**
- Energy outliers: **1,298 sessions**
- Outliers in both duration and energy: **1,064 sessions**

Outliers were retained because unusual charging sessions are not necessarily data errors and may represent legitimate charging behaviour.

### Time-of-Day Behaviour

Charging activity was concentrated during daytime and early evening hours.

The analysis also identified an important late-night pattern:

- Late-night sessions tended to remain connected much longer
- Average charging rate remained relatively stable throughout most of the day
- Higher late-night energy delivery therefore appears to be driven primarily by **longer session duration**, rather than substantially higher charging power

For behavioural comparison, late night was defined as approximately **9:00 PM to 5:59 AM**.

### Charging End Reasons

The two dominant session end reasons were:

- Charging cable disconnected and returned to the station
- Charging ended by the vehicle

A chi-square test identified a statistically significant association between time group and charging end reason:

**Chi-square ≈ 52.8, p < 0.001**

Late-night charging sessions were more likely to end because the vehicle stopped charging and less likely to end through manual cable disconnection.

This relationship is descriptive and does not establish causation.

## Forecasting

Weekly demand was used to evaluate whether historical charging behaviour contained practical predictive value.

The time series contained **131 weekly observations** from January 2022 through June 2024.

The final 12 weeks were held out as an unseen chronological test period.

### Weekly Session Forecast Results

| Model | MAE | RMSE |
|---|---:|---:|
| 2-Week Baseline | 65.58 | 68.37 |
| Linear Regression | **23.12** | **26.97** |
| ARIMA (1,1,1) | 67.19 | 69.93 |
| ARIMA (1,1,1) with Drift | 50.89 | 53.13 |

**Linear Regression produced the best forecast accuracy.**

The result shows that the strong long-term growth trend contained useful predictive information and that a more complex model did not automatically produce better forecasts.

### Weekly Energy Forecast

Linear Regression also substantially improved weekly energy forecasting.

| Model | MAE | RMSE |
|---|---:|---:|
| Baseline | 1,097.91 kWh | 1,160.65 kWh |
| Linear Regression | **390.71 kWh** | **460.94 kWh** |

An LSTM model was not pursued because the weekly dataset provided only 119 training observations after the chronological train/test split, which was considered limited for a neural-network forecasting approach.

## Ottawa Charging Infrastructure

The infrastructure analysis identified **218 Ottawa charging locations**.

### Charger Categories

| Charger Category | Stations | Total Ports | Avg. Ports / Station |
|---|---:|---:|---:|
| Level 2 | 189 | 717 | 3.79 |
| DC Fast | 28 | 110 | 3.93 |
| Level 1 | 1 | 1 | 1.00 |

Ottawa's charging infrastructure is therefore strongly dominated by **Level 2 charging**, with a smaller DC fast-charging network.

Typical Level 2 charging power was approximately **6–7 kW**, while DC fast chargers ranged considerably higher, including stations rated at 50 kW, 150 kW, 200 kW, 250 kW, and 325 kW.

### Infrastructure Validation

Station capacity was compared independently across the station, port, and plug datasets.

**217 of 218 Ottawa stations had matching capacity counts across all three sources.**

This provided strong confidence in the infrastructure records used for the exploratory analysis.

## Charging Networks

Major charging networks represented in the Ottawa infrastructure data included:

- FLO
- ChargePoint
- SWTCH
- Tesla
- Circuit Électrique
- Ivy
- Electrify Canada
- ChargeLab
- EVStart
- Petro-Canada
- AmbuEnergy
- Hypercharge

FLO and ChargePoint represented the largest number of plug records in the Ottawa subset.

## Pricing Structures

Raw pricing descriptions were highly varied and included:

- Hourly charging
- Free charging
- Per-kWh pricing
- Per-minute pricing
- Flat session pricing
- Complex / tiered tariffs
- Free charging with paid parking
- Parking-only fees
- Variable external pricing
- Unknown pricing

The original pricing text was retained while a simplified analytical pricing category was created for comparison.

This infrastructure information may support future **tariff-informed estimated revenue analysis**.

## Estimated Charging Revenue

An optional financial analysis estimated charging revenue and electricity cost using simplified assumptions.

The results should be interpreted as **analytical estimates rather than audited City revenue or profit**.

Estimated gross charging margin subtracts only assumed electricity cost and does not include expenses such as:

- equipment installation
- maintenance and repairs
- networking/software fees
- payment processing
- administration
- taxes
- depreciation
- demand charges

Future analysis could improve these estimates by applying station-specific tariff information from the infrastructure dataset.

## Tools & Technologies

- **Python**
- **Pandas**
- **NumPy**
- **Matplotlib**
- **Plotly**
- **SciPy**
- **Scikit-learn**
- **Statsmodels**
- **ArcGIS REST API**
- **Jupyter Notebook / VS Code**
- **CSV / Parquet**

## Data Files

The repository includes both charging-session and infrastructure datasets used in the analysis.

Key files include:

- `ev_charging_usage.csv`
- `ev_charging_usage_cleaned.parquet`
- `stations_detail.csv`
- `ports_detail.csv`
- `plugs_detail.csv`

The primary charging-session data originates from the **City of Ottawa Open Data platform**.

## Limitations

Several limitations should be considered when interpreting the results:

- Observational relationships do not establish causation
- Weather, holidays, and major Ottawa events were not incorporated into the final forecasting models
- Some infrastructure pricing information is missing or complex
- Current station pricing may not represent historical tariffs for every session in the 2022–2024 usage dataset
- Infrastructure status and available-port information may represent a snapshot rather than historical availability
- Estimated revenue does not represent audited financial results
- Geographic context and nearby amenities require additional geospatial analysis

## Potential Future Work

Possible extensions include:

- Geospatial analysis of nearby businesses, hotels, parking facilities, and amenities
- Weather and event-based demand modelling
- Station-level utilization using verified physical port capacity
- Pricing vs usage analysis
- Charger power vs session-duration analysis
- Tariff-informed revenue modelling
- Interactive dashboard development
- Longer-term forecasting as additional historical data becomes available

## Conclusion

The project demonstrates that Ottawa's public EV charging data contains meaningful operational and predictive information.

Charging demand increased substantially over time, site performance varied considerably, and session duration was strongly associated with energy delivered. Time-of-day and day-of-week patterns also revealed distinct charging behaviours.

A simple Linear Regression time-trend model substantially outperformed both recent-history baselines and ARIMA models for near-term weekly demand forecasting.

The infrastructure analysis added another layer of context by identifying actual port capacity, charger power, charging level, network operators, and pricing structures.

Together, the three notebooks provide an end-to-end analytical workflow covering:

**Data acquisition → Data cleaning → Exploratory analysis → Statistical testing → Forecasting → Infrastructure analysis → Business interpretation**
