---
layout: page
title: "DSC 80 Final Project: Power Outages and Fuel Diversity"
---

# DSC 80 Final Project: Power Outages and Fuel Diversity

## Introduction

### Motivation

Power outages create serious risks for households, businesses, and critical services. Losing electricity for many hours can disrupt heating and cooling, medical equipment, transportation, and communication. Some outages are unavoidable because of extreme weather, but the way a power system is built can make outages either much worse or much easier to recover from.

Most discussions of power outages focus on demand, prices, or short-term weather events. In this project, I focus on the **supply side** of the grid instead: the mix of fuel sources and generation capacity that each U.S. state relies on. A more diverse energy portfolio might make the grid more flexible when one source fails, but it is not obvious how large this effect is. Understanding whether fuel diversity is actually associated with shorter outages can help inform long-term infrastructure planning and energy policy.

### Project Question

In this project, I study the following question:

> **How is a state’s electricity generation mix—especially the diversity of fuel sources—related to the average duration of major power outages?**

I explore this question by combining detailed outage records with state-level information on generation capacity and fuel types, and by building models that try to predict outage duration using these features.

### Data

My project uses one main outage dataset and two additional datasets describing electricity generation in the United States.

#### Major Power Outage Events Dataset

The primary dataset comes from a ScienceDirect article that compiles **major power outage events in the continental U.S.**  
The raw outage dataset contains **1534 rows and 56 columns**. Each row corresponds to a major outage event in a particular state and month.

For this project, the most important columns are:

| Column name(s)                      | Description                                                                                           |
|------------------------------------|-------------------------------------------------------------------------------------------------------|
| `year`, `month`, `state`           | Calendar year, month, and U.S. state in which the outage occurred.                                   |
| `climate_region`                   | Broad U.S. climate region of the state, as defined by the National Centers for Environmental Information. |
| `climate_cat`, `anomaly_level`     | Indicators of El Niño/La Niña conditions and temperature anomalies during the event period.          |
| `cause_cat`, `cause_detail`        | Categorical cause of the outage (e.g., weather, equipment failure) and a more detailed text description. |
| `duration`                         | Length of the outage in minutes. This is the main response variable in my analysis.                  |
| `pc_realgsp_state`, `pi_util_of_usa` | State-level economic indicators (per-capita gross state product and the share of utility-sector income in the U.S.). |
| `population`                       | Total population of the state in the given year.                                                     |
| `pop_pct_urban`, `pop_pct_uc`      | Percent of the state’s population living in urban areas and in urban clusters.                       |
| `popden_urban`, `popden_uc`, `popden_rural` | Population densities (persons per square mile) in urban areas, urban clusters, and rural areas.      |
| `area_pct_urban`, `area_pct_uc`    | Percent of the state’s land area classified as urban areas and as urban clusters.                    |

These variables allow me to study how outage duration relates to climate conditions, causes, and basic demographic and economic characteristics of each state.

#### EIA-860 Annual Electric Generator Report

To capture the **supply-side structure of the grid**, I use the EIA-860 generator-level survey, which describes existing electric generators in the U.S. The cleaned dataset used in this project has **62282 rows and 5 columns**, with the key variables:

| Column name        | Description                                                                                |
|--------------------|--------------------------------------------------------------------------------------------|
| `producer_type`    | Type of electricity producer (for example, utility, industrial, or commercial).           |
| `fuel_source`      | Primary fuel used by the generator (such as coal, natural gas, nuclear, or solar).        |
| `generators`       | Count of generator units at the plant (may include missing values).                        |
| `facilities`       | Number or identifier of facilities associated with generation (may include missing values).|
| `nameplate_capacity`, `summer_capacity` | Rated maximum output of the generator under standard and summer peak conditions, in megawatts. |

These columns are used to build state-level measures of generation capacity and fuel diversity.

#### EIA-923 Power Plant Operations Report

Finally, I use the EIA-923 survey, which records **actual electricity production**. The processed dataset contains **54002 rows and 8 columns**. For this project, the main variables are:

| Column name     | Description                                                                 |
|-----------------|-----------------------------------------------------------------------------|
| `producer_type` | Same producer type categories as in the EIA-860 dataset.                    |
| `fuel_source`   | Same fuel type categories as in the EIA-860 dataset.                        |
| `generation_mwh`| Actual electricity generated in megawatt-hours for the plant or generator. |

By merging these three datasets at the state and year level, I obtain features that describe both **how outages occur** and **how each state generates electricity**. This combined view supports my analysis of how fuel diversity and generation capacity relate to the duration of major power outages.
