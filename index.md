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

## Data cleaning

We began by renaming columns for improved readability and dropped many columns related to climate, sales, and outage-specific details that were less relevant to assessing infrastructure resilience from an economic and investment perspective. For example, columns related to outage duration drivers, demand-side metrics like total price or customers, and some state economy indicators were removed. The resulting columns retained include key variables such as year, state, climate region, outage duration, economic indicators, population metrics, and state-level outage summaries. The remaining columns are listed here.

Index(['year', 'state', 'climate_region', 'anomaly_level', 'climate_cat',
       'cause_cat', 'cause_detail', 'duration', 'pc_realgsp_state',
       'pi_util_of_usa', 'population', 'pop_pct_urban', 'pop_pct_uc',
       'popden_urban', 'popden_uc', 'popden_rural', 'area_pct_urban',
       'area_pct_uc', 'yearly_outage_count_bystate',
       'yearly_avg_duration_bystate'],
      dtype='object')

Small changes we made include:

- Replacing `state` column with `posta_code`, to be consistent with external dataframe.
- Dropping columns with `NaN` state and years to maintain data integrity.
- Converted numerical columns to numerical, like `duration`, and cleaned the text of `cause_detail` for EDA later.

# Feature Engineering

We created new columns for **state specific total outage counts**, and **annual state-wise average outage duration of outage**.


# Assessment of Missingness

Upon inspection, we found that from our cleaned dataframe, the columns with the most amount of missing data are  
`cause_detail`, `duration`, `popden_rural`, `popden_uc`, and `anomaly_level`.

## NMAR Assessments

### Addressing Missingness of Cause Detail

The missingness of the `cause_detailed` col is most likely NMAR since the details might give sensitive information about an individual. Usually when an outage occurs, we can expect to see a recording or failsafe that indicates what was broken. That said, if the outage occurs in under-resourced areas then there is a much higher chance that it is not recorded at all due to lack of infrastructure.

### Addressing Missingness of Duration

On the other hand, we have reason believe that `duration` not NMAR since even though we can make a case that if a duration is too low or too high it might not be recorded. Why we believe that reasoning is flawed is because the `duration` column has 78 entries that only have 0 duration. Since, duration is non negative and the max duration is rather high, we can say that this missingness cannot be determined by the value within the column itself. If anything, if an outage lasts very long then there is more reason that it should be recorded for record keeping. So it is not NMAR.

### Addressing Missingness of Climate Category and Region

It doesn’t make much sense to label this as NMAR because all geographical regions in the country are labeled with their respective category and region, so it is unlikely that an outtage would happen in an area that cannot be classified as one of them. Thus, it cannot be that the missingness is dependent on the value of the entry within the coclumn itself.

### Addressing Missingness of popden_uc, popden_rural

There is no reason to believe that these columns are NMAR since the population level, urban or not, is highly dependent on the state and its population, both of which are features in our data.

# MAR Test
## Set up

Cause detail is the column with the most missingness in our cleaned dataset. We find it interesting to examine whether the missingness of this column depends on other observable variables.

We performed **permutation testing** to compare distributions of other variables conditional on the presence or absence of `cause_detail`.

- **For numeric variables:** We used mean differences as the test statistic.
- **For categorical variables:** We used Total Variation Distance (TVD).




# Hypothesis Testing

## Set up

We investigate whether states with more diverse energy sources tend to experience less severe power outages.

> **Is there evidence that higher fuel diversity is associated with shorter average power outage durations?**

This analysis uses previously engineered features:

- **Fuel Diversity Score**  
  - *Capacity Entropy* — reflects infrastructure potential  
  - *Generation Entropy* — reflects actual energy usage patterns  
- **Average Outage Duration by State** — proxy for grid robustness and outage severity  

---

# Hypothesis

We divide states into **high** and **low** diversity groups using the **median** fuel diversity index.

### **Null Hypothesis (H₀):**  
No difference in average outage duration between high- and low-diversity states.

### **Alternative Hypothesis (H₁):**  
High-diversity states have **shorter** average outage durations.

---

# Methodology

### **Test Statistic**
We use **difference in group means** because the hypothesis is directional and compares two groups.

### **Significance Level**
We choose **α = 0.05**, a standard choice for hypothesis testing.

# Framing a Prediction Problem

To get a closer look into the relationship between our features and outage duration, we have decided to make our task be about predicting the average outage duration per state per year through regression. By predicting this response variable, we aim to provide a model that can derive average outage duration data catered to each state more accurately. We will propose a baseline model, starting with linear regression, as it is one of the simpler models, with percent RMSE, the percentage of RMSE over the mean of the average duration per state per year feature.

Our features of interest include  
`climate_region`, `cause_cat`, `population`, `capacity_fuel_diversity_index`, and `generation_fuel_diversity_index`.  
These features are either fixed (e.g., geographic or categorical labels) or collected prior to the prediction year through public data sources, ensuring they would be available at the time of prediction. This respects the temporal constraint of not using information from after the event we aim to predict.

# Baseline Model

First, we attempted Linear Regression using the covariates identified during the prediction problem formulation to predict the average outage duration per year per state. However, the results were not promising, as the RMSE was quite high. This is likely due to the nature of our test set. While the model learns from the training data with access to all covariates, it is important that its performance on the test set does not degrade significantly compared to training. If the model’s accuracy on the test set remains close to that on the training set, we consider it a good model because it demonstrates robustness to new, potentially independent data outside the training distribution.

# Fairness Analysis

To assess model fairness, we choose group the cause category by severe weather or no severe weather. Since our model focuses on average duration based on the predictive power of population, geographical, and energy generation/storage factors, we want to know if the model is actually performing well on severe weather or not. This is an interesting way to answer the question of “How good is the model in identifying natural causes and appropriate assign weights to those causes so that it is predicting average duration fairly?”. We will see whether or not our model is robust under why the outage happened.

It is fitting to test whether or not the model performs well on different groups of climate categories. Specifically, we want to see if the average outage duration is well modeled on severe weather and no severe weather in hopes to bring up awareness of risks that come with outages.

