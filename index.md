---
layout: page
title: "DSC 80 Final Project: Power Outages and Fuel Diversity"
---

# DSC 80 Final Project: Power Outages and Fuel Diversity

## Introduction

### Motivation

Major power outages can disrupt daily life, damage local economies, and threaten critical services such as hospitals, transportation, and communication systems. Although some events are unavoidable due to extreme or unexpected weather, the way the power grid is designed and managed can make these outages shorter and less severe—or longer and more damaging. The resilience of the electric grid—its ability to absorb shocks and return to normal operation quickly—is therefore essential for maintaining reliable access to electricity in the face of natural hazards, equipment failures, and other disruptions.

Most studies of power outages emphasize demand-side factors, electricity prices, or short-term regional weather patterns. In contrast, this project focuses on the **supply side** of the grid: how much generation capacity a state has, which fuels it uses, and how diverse its portfolio of generators is. A more diverse and flexible generation mix may help the grid continue operating even when one source is temporarily unavailable. Understanding whether such diversity is actually related to outage outcomes can provide useful insight for infrastructure investment and energy-policy decisions.

### Project Objective

This project explores the relationship between electricity generation characteristics and major outage events in the United States. In particular, we connect:

- measures of generation capacity, fuel-type diversity, and actual electricity production (from EIA-860 and EIA-923), with  
- measures of outage frequency, duration, and impact (from a DOE power outage dataset).

More concretely, in this project we:

- Conduct an initial exploratory analysis of major outage events.
- Investigate the question: **“Do states with more diverse energy portfolios tend to experience fewer or shorter outages?”**
- Build a predictive model that uses grid and regional features to estimate the duration of an outage.

Understanding how and why large outages occur is especially important in a climate-stressed and electricity-dependent world. By combining detailed U.S. outage records with state-level economic, environmental, and generation information, this project offers a way to examine how resilience varies across states. Our goal is to develop a simple, interpretable model of average outage duration using features that are known ahead of time, providing a small but concrete step toward using data to better understand and improve grid reliability.

## Data

### Major Power Outage Events Dataset

The primary dataset used in this project comes from a ScienceDirect article titled *Data on major power outage events in the continental U.S.*  

The raw dataset contains **1534 rows and 56 columns**, where each row corresponds to a major outage event in a given state and month.

The most relevant columns for our analysis include:

| Column Name(s)                     | Description                                                                                                  |
|-----------------------------------|--------------------------------------------------------------------------------------------------------------|
| `year`, `month`, `state`          | Calendar year, month, and U.S. state in which the outage occurred.                                          |
| `climate_region`                  | U.S. climate region of the state, as defined by the National Centers for Environmental Information.          |
| `climate_cat`, `anomaly_level`    | Indicators of El Niño/La Niña climate episodes and temperature anomalies during the period of the event.    |
| `cause_cat`, `cause_detail`       | Categorical and textual descriptions of the cause of the outage (e.g., severe weather, equipment failure).  |
| `duration`                        | Duration of the outage in minutes; this is the main outcome variable in our study.                          |
| `pc_realgsp_state`, `pi_util_of_usa` | Economic indicators such as per-capita gross state product and the share of U.S. utility-sector income.  |
| `population`                      | Total population of the state in the given year.                                                             |
| `pop_pct_urban`, `pop_pct_uc`     | Percentage of the state’s population living in urban areas and in urban clusters.                           |
| `popden_urban`, `popden_uc`, `popden_rural` | Population density (persons per square mile) in urban areas, urban clusters, and rural areas.   |
| `area_pct_urban`, `area_pct_uc`   | Percentage of the state’s land area classified as urban areas and urban clusters.                           |

These features allow us to connect outage characteristics with climate conditions and basic demographic and economic context.

### EIA-860 Annual Electric Generator Report

To describe the **structure of electricity generation** in each state, we use the EIA-860 survey, which provides generator-level information about existing and planned electric generating units in the U.S.

The cleaned dataset we work with contains **62282 rows and 5 columns**. Key columns include:

| Column Name        | Description                                                                                      |
|--------------------|--------------------------------------------------------------------------------------------------|
| `producer_type`    | Type of electricity producer (e.g., utility, industrial, commercial).                           |
| `fuel_source`      | Primary fuel type used by the generator (e.g., coal, natural gas, nuclear, solar).              |
| `generators`       | Number of generator units at a plant (may contain missing values).                              |
| `facilities`       | Identifier or count of facilities associated with generation (may contain missing values).      |
| `nameplate_capacity`, `summer_capacity` | Rated maximum output of the generator under standard and summer peak conditions, in megawatts (MW). |

From these variables, we construct state-level summaries of capacity and fuel-mix diversity.

### EIA-923 Power Plant Operations Report

We also use the EIA-923 dataset, which records **actual electricity production** at power plants.

The processed version used here contains **54002 rows and 8 columns**. The core variables are:

| Column Name     | Description                                                                        |
|-----------------|------------------------------------------------------------------------------------|
| `producer_type` | Same producer type categories as in the EIA-860 dataset.                          |
| `fuel_source`   | Same fuel type categories as in the EIA-860 dataset.                              |
| `generation_mwh`| Actual electricity generated, measured in megawatt-hours.                         |

By aggregating EIA-860 and EIA-923 data to the state–year level and merging them with the outage dataset, we obtain a combined dataset that describes both **how outages occur** and **how each state generates electricity**. This integrated view is the basis for the analyses and models presented in the rest of the project.

## Data Cleaning and Exploratory Data Analysis

### Data Cleaning

#### “outage” dataframe

##### Data cleaning

To analyze outages at the state–year level, I first cleaned the `outage` dataframe.

I started by simplifying the columns:

- Renamed some variables so that the names are shorter and easier to read.
- Kept columns that are relevant for my question, such as  
  `year`, `state`, `climate_region`, `anomaly_level`, `climate_cat`,  
  `cause_cat`, `cause_detail`, `duration`, `pc_realgsp_state`,  
  `pi_util_of_usa`, `population`, `pop_pct_urban`, `pop_pct_uc`,  
  `popden_urban`, `popden_uc`, `popden_rural`, `area_pct_urban`,  
  `area_pct_uc`, and the state-level summary columns.
- Dropped many other columns about detailed pricing, customer counts, or
  very specific outage mechanics that are not used later in the analysis.
- Removed rows where `state` or `year` is missing to keep the dataset consistent.
- Converted columns that should be numeric (for example `duration`) from
  strings to numeric types.
- Lightly cleaned the `cause_detail` text column (stripping spaces, unifying
  capitalization) so it can be used for later visualizations.

After these steps, the `outage` dataframe keeps the main variables we need:
time and location of the outage, climate information, cause, duration, and
basic economic and demographic context, plus state-level outage summaries.

##### Feature engineering

Next, I created summary features that aggregate outages at the state–year level:

- `yearly_outage_count_bystate`: total number of major outages in a given
  state and year.
- `yearly_avg_duration_bystate`: average duration of major outages in that
  state and year.

These features were joined back onto the `outage` dataframe so that each row
includes both the original event-level information and these state-year
summaries.

*(On the website I show the head of this cleaned and engineered `outage`
dataframe.)*

---

#### “capacity” and “generation” dataframes

The raw capacity (`EIA-860`) and generation (`EIA-923`) datasets also needed
cleaning before they could be merged with the outage data.

Key steps:

- Restricted the data to years **2000–2016**, where both datasets are mostly
  complete.
- Removed commas from capacity and generation columns and converted them to
  numeric types so that they can be used in calculations.
- Standardized fuel source names so that the same fuel has the same label in
  both datasets.
- For capacity, kept rows where the fuel source is recorded at the desired level
  (for example, treating “All Sources” as total installed capacity when needed).
- For generation, dropped rows labeled “Total” so that calculations are based on
  individual fuel types.
- Dropped rows with clearly invalid or missing numeric values to avoid
  distortions in later steps.

##### Fuel-diversity features (entropy)

To measure how diverse each state’s fuel mix is, I engineered two new features:

- `capacity_fuel_diversity_index` – diversity of **installed capacity** by fuel.  
- `generation_fuel_diversity_index` – diversity of **actual generation** by fuel.

Both are based on **Shannon entropy**. For a vector of fuel shares \(p(x)\),

\[
H = -\sum p(x)\log p(x).
\]

Interpretation:

- **Higher entropy** → electricity capacity or production is spread across many
  different fuels (more diverse mix).
- **Lower entropy** → most capacity or production comes from only a few fuels
  (less diverse mix).

After computing these indices for each state–year pair, I kept a compact
summary dataframe for capacity (`capacity_entropy`) and for generation
(`generation_entropy`). The heads of these tables are shown on the website.

---

### Merging engineered features into outages

Finally, I merged the fuel-diversity features into the outage summaries.

- Merge keys: **`state`** and **`year`**.
- Each row in the final dataframe represents a **state–year** and includes:
  - climate variables and outage causes,
  - economic and demographic variables,
  - `yearly_outage_count_bystate` and `yearly_avg_duration_bystate`,
  - `capacity_fuel_diversity_index` and `generation_fuel_diversity_index`.

This merged dataframe is what I use for the exploratory data analysis and
modeling sections that follow. On the website, I display the head of this
final `outage` dataframe to illustrate its structure.
