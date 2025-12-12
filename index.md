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

