# Food Valuation and Gut Hormones Analysis Script - README

## Table of Contents

- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Data Structure](#data-structure)
  - [Required Data Files](#required-data-files)
  - [Key Column Names](#key-column-names)
- [Workflow](#workflow)
  - [1. Data Loading](#1-data-loading)
  - [2. Data Cleaning and ID Mapping](#2-data-cleaning-and-id-mapping)
  - [3. Session Standardization](#3-session-standardization)
  - [4. Hormone Data Processing](#4-hormone-data-processing)
  - [5. Data Merging](#5-data-merging)
  - [6. Variable Creation](#6-variable-creation)
  - [7. Descriptive Statistics](#7-descriptive-statistics)
  - [8. Mixed-Effects Modeling](#8-mixed-effects-modeling)
  - [9. Model Diagnostics](#9-model-diagnostics)
  - [10. Correlation Analysis](#10-correlation-analysis)
  - [11. Statistical Output](#11-statistical-output)
- [Troubleshooting](#troubleshooting)
- [Contributing and Citation](#contributing-and-citation)

## Overview
This Jupyter notebook (`food_valuation_gut_hormones_LMM.ipynb`) contains comprehensive data analysis code for a bariatric surgery research study examining the relationship between surgery types, behavioral measures (bidding behavior), and gastrointestinal (GI) hormones across multiple time points.

## Description
The script analyzes longitudinal data from bariatric surgery patients, focusing on:
- Bidding behavior in food-related tasks
- GI hormone levels (GLP-1, PYY, Ghrelin)
- Surgery type effects across different time points
- Mixed-effects modeling and statistical analyses

## Prerequisites

### Dependencies
```python
import statsmodels.api as sm
import statsmodels.formula.api as smf
import pandas as pd
import matplotlib.pyplot as plt
from scipy.stats import zscore
import seaborn as sns
from scipy.stats import linregress
from statsmodels.formula.api import mixedlm
from statsmodels.stats.diagnostic import het_breuschpagan
from statsmodels.stats.outliers_influence import variance_inflation_factor
from scipy import stats
import numpy as np
```

### System Requirements
- Python 3.7+
- Jupyter Notebook environment
- Sufficient RAM for large datasets (recommended: 8GB+)

## Data Structure

### Required Data Files

### 1. Main Participant Data
**File:** `redcap_export_data.csv`
**Location:** `/path/to/your/data/directory/`

This is the primary RedCap export containing participant demographics and behavioral data.

### 2. GI Hormones Data  
**File:** `hormones_data.csv`
**Location:** `/path/to/your/data/directory/`

Contains hormone measurements across different time points and fed states.

### 3. Bidding Behavior Data
**File:** `bidding_behavior_data.csv`
**Location:** `/path/to/your/data/directory/`

Contains averaged bidding data for high and low calorie conditions.

### Key Column Names

#### Main DataFrame (redcap)
**Required columns:**
- `record_id`: Unique participant record identifier
- `id_recherchescan`: Research scan ID (will be converted to `id_participant`)
- `redcap_event_name`: Time point identifier (`baseline_arm_1`, `4_mois_arm_1`, `12_mois_arm_1`, `24_mois_arm_1`)
- `age`: Participant age
- `sexeF`: Gender (0=Male, 1=Female)
- `type_chx`: Surgery type (1=Gastrectomy, 2=RYGB, 3=DBP-SD)
- `imc_recherche_x`: BMI at research timepoint
- `twl_recherche_x`: Total weight loss percentage
- `poids_recherche_x`: Weight at research timepoint

#### Hormone DataFrame
**Required columns:**
- `Identification Patient`: Patient ID (uppercase format like SUB001, SUB002)
- `Temps`: Time point (`BL`, `4m`, `12m`, `24m`)
- `Colonne1`: Fed state (`0`/`T0`=fasting, `60`/`T60`=postprandial)
- `GLP-1 total`: GLP-1 hormone levels
- `PYY`: PYY hormone levels  
- `Ghrelin Unacylated`: Unacylated ghrelin levels
- `Ghrelin acylated`: Acylated ghrelin levels

#### Bidding DataFrame
**Required columns:**
- `id_participant`: Participant identifier (lowercase format)
- `session`: Time point (1=baseline, 2=4 months, 3=12 months, 4=24 months)
- `calories`: Calorie condition (`hi` or `lo`)
- `bid`: Bidding value
- `onset_bid`: Bid onset time

## Workflow

### 1. Data Loading
Load the three main CSV files containing participant demographics, hormone measurements, and bidding behavior data.

### 2. Data Cleaning and ID Mapping
If your data has missing `id_recherchescan` values, manually map them:

```python
# Example mappings (update with your specific IDs)
redcap.loc[redcap['record_id'] == 1, 'id_recherchescan'] = 'sub001'
redcap.loc[redcap['record_id'] == 6, 'id_recherchescan'] = 'sub002'
# Add more mappings as needed
```

### 3. Session Standardization
The script maps RedCap event names to numeric sessions:
- `baseline_arm_1` → 1
- `4_mois_arm_1` → 2  
- `12_mois_arm_1` → 3
- `24_mois_arm_1` → 4

### 4. Hormone Data Processing
- PYY values `<2,0` are replaced with `1`
- Fed state mapping: `0`/`T0` → `ac` (fasting), `60`/`T60` → `pp` (postprandial)

### 5. Data Merging
Merge hormone data with participant demographics and bidding behavior data based on participant ID and session.

### 6. Variable Creation
Create standardized variables for analysis:
- `session`: Numeric time point (1-4)
- `id_participant`: Standardized participant ID (lowercase)
- `fed_state`: Fed state (`ac`=fasting, `pp`=postprandial)
- `IMC_baseline`: Baseline BMI extended to all sessions
- `GLP_1_total`: Renamed from `GLP-1 total`
- `Ghrelin_Unacylated`: Renamed from `Ghrelin \nUnacylated`
- `Ghrelin_acylated`: Renamed from `Ghrelin \nacylated`

### 7. Descriptive Statistics
Generate comprehensive demographic and clinical characteristics tables using the `basic_stat()` function.

### 8. Mixed-Effects Modeling
Implement linear mixed-effects models to analyze relationships between variables across time points.

### 9. Model Diagnostics
Perform residual analysis, QQ-plots, and homoscedasticity tests to validate model assumptions.

### 10. Correlation Analysis
Calculate correlations between bidding behavior and hormone levels, stratified by session and surgery type.

### 11. Statistical Output
Generate final statistical results and export processed datasets.

## Troubleshooting

### Analysis Functions

#### `describe_column(column_name)`
Provides descriptive statistics and NaN counts for a specified column.

#### `basic_stat(df)`
Generates comprehensive demographic and clinical characteristics table across all time points including:
- Sample sizes per session
- Age (mean ± SD)
- Gender distribution (M:F)
- Surgery type counts
- BMI and weight loss percentages

### Common Issues
1. **Missing Participant IDs**: Check the ID mapping section and add any missing mappings
2. **Column Name Mismatches**: Verify column names match exactly (case-sensitive)
3. **Session Mapping**: Ensure your time point labels match the expected format
4. **Data Type Issues**: The script converts data types - ensure your data is compatible

### Usage Notes
1. **File Paths**: Update all file paths to match your directory structure
2. **Participant ID Format**: Ensure consistency between datasets (some use uppercase, others lowercase)
3. **Missing Data**: The script handles various missing data scenarios but may need adjustment for your specific dataset
4. **Surgery Types**: The script expects surgery types coded as 1, 2, 3 - adjust mappings if your coding differs
5. **Special Participants**: The script separates participants with specific identifiers in their ID - modify this logic if not applicable

### Output Files
The script generates several CSV files:
- `processed_participant_data.csv`: Processed main participant data
- `merged_hormone_data.csv`: Merged participant and hormone data  
- `missing_data_report.csv`: Participants with missing data (if applicable)
- Various correlation and analysis results

### Data Validation
- Run `describe_column()` on key variables to check data distribution
- Use `basic_stat()` to verify sample sizes across sessions
- Check for unexpected NaN patterns in merged datasets

## Contributing and Citation

This script was specifically developed for our research project and data structure. Adapting it to other datasets may require significant modifications due to differences in data organization, column naming conventions, and study design. This code is made available primarily for reproducibility purposes and as an educational resource for researchers interested in implementing linear mixed-effects modeling (LMM) in the context of food valuation studies.
