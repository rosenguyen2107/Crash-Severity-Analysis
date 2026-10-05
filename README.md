# Characteristics Associated with Severe and Fatal Motor Vehicle Crash Outcomes in the United States, 2022-2024

## Project overview

Most police-reported crashes in the United States cause no serious harm. About 3% result in a death or a suspected serious injury, roughly 194,000 crashes per year. This project examines:

> **Which behavioral and environmental characteristics are most strongly associated with severe or fatal outcomes in U.S. police-reported crashes from 2022 to 2024?**

The analysis cleans and merges three years of national crash data and explores severity patterns with survey-weighted descriptive statistics and visualizations. It then estimates a weighted logistic regression to identify which characteristics remain associated with a severe or fatal outcome when the others are held constant.

**Predictors:**
- **Behavioral:** speeding, alcohol involvement
- **Time:** night (21:00–05:59), weekend
- **Environment:** weather (lighting is used in the EDA only)
- **Roadway:** urban/rural, junction type, interstate
- **Control:** year

### Key findings

| Characteristic | Adjusted odds ratio (95% CI) |
|---|---|
| Alcohol involved | 2.77 (2.41–3.17) |
| Speeding involved | 2.00 (1.67–2.40) |
| Night vs day/evening | 1.48 (1.34–1.63) |
| Rural vs urban | 1.41 (1.20–1.67) |
| Weekend vs weekday | 1.26 (1.21–1.32) |
| Rain vs clear | 0.69 (0.62–0.76) |
| Snow/ice vs clear | 0.41 (0.34–0.50) |

- Alcohol and speeding show the strongest associations with severe outcomes, and they compound each other. 13.9% of crashes involving both are severe or fatal, compared with 2.5% of crashes involving neither.
- Crash volume peaks in the weekday afternoon rush, but crash severity peaks between midnight and 05:00.
- Adverse weather is associated with less severe crashes, plausibly because drivers slow down.
- The patterns are stable across 2022, 2023 and 2024.

All results are associations, not causal effects.

## Data source

**NHTSA Crash Report Sampling System (CRSS), 2022–2024**
National Highway Traffic Safety Administration, National Center for Statistics and Analysis
<https://www.nhtsa.gov/crash-data-systems/crash-report-sampling-system>

CRSS is a nationally representative probability sample of police-reported crashes of all severities. NHTSA deliberately oversamples severe crashes, so every record carries a sampling weight (`WEIGHT`) that indicates how many crashes nationwide it represents. All estimates in this project are weighted. Without weights, 13.2% of sampled crashes are severe or fatal, compared with 3.2% nationally.

Two files are used for each year, so there are six files in total:

| File | One row per | Used for |
|---|---|---|
| `accident.csv` | crash | Outcome, alcohol, time, lighting, weather, urban/rural, junction, interstate, sampling weight |
| `vehicle.csv` | vehicle | Speeding, aggregated to one value per crash |

After cleaning, the analytic dataset contains 155,308 crashes. The regression uses 148,966 crashes after excluding those with unknown speeding or interstate status.

The raw data files are not included in this repository. See [Steps to run the analysis](#steps-to-run-the-analysis) for how to obtain them.

## Repository contents

```
Crash-Severity-Analysis/
├── README.md                     
├── requirements.txt              
├── .gitignore                    
├── notebooks/
│   └── crash_severity_analysis_simplified.ipynb   
├── data/
│   ├── raw/                      
│   └── processed/                
│       ├── crss_crash_analytic_2022_2024.csv      
│       └── logistic_regression_odds_ratios.csv    
└── figures/                     
```

### Figures produced

| File | Shows |
|---|---|
| `01_severity_distribution.png` | Severity distribution: unweighted sample vs weighted national estimate |
| `02_by_year.png` | Crash volume and severe/fatal share by year |
| `03_speeding_alcohol.png` | Severe/fatal share by speeding and alcohol |
| `04_speeding_alcohol_combined.png` | Combinations of speeding and alcohol |
| `05_by_hour.png` | Crash volume and severity by hour of day |
| `06_heatmap_day_hour.png` | Severe/fatal share by day of week × hour |
| `07_night_weekend.png` | Night vs day/evening and weekend vs weekday |
| `08_environment_roadway.png` | Lighting, weather, junction, urban/rural, interstate |
| `09_profile_severe_vs_other.png` | Share of severe vs other crashes with each characteristic |
| `10_unadjusted_ranking.png` | Unadjusted ranking of characteristics |
| `11_stability_by_year.png` | Unadjusted ratios by year |
| `12_model_building.png` | Odds ratios across the three model steps |
| `13_forest_adjusted_vs_crude.png` | Final adjusted vs crude odds ratios |
| `14_predicted_probabilities.png` | Predicted severe/fatal probability for example crashes |

## Required software and packages

- **Python** 3.9 or later
- **Jupyter** (Jupyter Notebook, JupyterLab, or VS Code with the Jupyter extension)
- Python packages, all listed in `requirements.txt`:

| Package | Used for |
|---|---|
| `pandas` | Loading, cleaning and aggregating data |
| `numpy` | Weighted averages and numerical operations |
| `matplotlib` | All charts |
| `statsmodels` | Weighted logistic regression |
| `ipython` | Displaying tables in the notebook |

## Steps to run the analysis

### 1. Get the repository

```bash
git clone https://github.com/rosenguyen2107/Crash-Severity-Analysis.git
cd Crash-Severity-Analysis
```

### 2. Download the CRSS data

1. Go to the [NHTSA CRSS page](https://www.nhtsa.gov/crash-data-systems/crash-report-sampling-system) and download the CSV data files for 2022, 2023 and 2024.
2. From each year's download, copy the `accident.csv` and `vehicle.csv` files into `data/raw/`.
3. Because every year uses the same file names, rename them so they do not overwrite each other. Each accident file must keep the same suffix as its vehicle file:

```
data/raw/
├── accident.csv      vehicle.csv      ← 2022
├── accident_1.csv    vehicle_1.csv    ← 2023
└── accident_2.csv    vehicle_2.csv    ← 2024
```

Any names work as long as they start with `accident` / `vehicle` and the suffixes match. The notebook will read the year from the `YEAR` column inside each file, not from the file name.

### 3. Install the packages

```bash
pip install -r requirements.txt
```

With Anaconda, you can use `conda install pandas numpy matplotlib statsmodels jupyter` instead.

### 4. Run the notebook

```bash
jupyter notebook notebooks/crash_severity_analysis_simplified.ipynb
```

Then select **Kernel → Restart & Run All**. A full run takes about one minute on a typical laptop. The notebook creates `data/processed/` and `figures/` automatically and saves all outputs there.

## Methods in brief

- **Weights:** All percentages use the CRSS sampling weight. `WEIGHT / 3` gives average crashes per year across the three pooled years.
- **Speeding** is aggregated from vehicle to crash: *Yes* if any vehicle was speeding, *Unknown* if none was but at least one had unknown status, otherwise *No*.
- **Model:** The regression is a logistic regression (`statsmodels` GLM, binomial family) with normalized CRSS weights. Standard errors are cluster-robust by primary sampling unit (`PSU_VAR`). The model is built in three steps (behavioral → + time → full) to show how each association changes as other characteristics are added.
- **Night vs lighting:** The model uses *night* rather than *lighting* because the two overlap almost completely: 95% of night crashes occur in the dark.

## Limitations

- The analysis is observational, so results are associations, not causal effects.
- Only police-reported crashes are included.
- The regression uses a simplified version of NHTSA's full complex-survey variance method.
- Alcohol (imputed by NHTSA when unknown), speeding and injury severity depend on police reporting.
- About 4% of crashes with unknown speeding or interstate status are excluded from the model.
- Driver characteristics, vehicle type and posted speed limit are not included.

## References

- National Highway Traffic Safety Administration. *Crash Report Sampling System (CRSS)*, 2022–2024 data files. National Center for Statistics and Analysis, U.S. Department of Transportation.
- National Highway Traffic Safety Administration. *CRSS Analytical User's Manual*. National Center for Statistics and Analysis, U.S. Department of Transportation.
- Seabold, S., & Perktold, J. (2010). statsmodels: Econometric and statistical modeling with Python. *Proceedings of the 9th Python in Science Conference*.
