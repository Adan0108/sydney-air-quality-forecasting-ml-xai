# Sydney Air Quality Forecasting with Machine Learning and XAI

This repository contains the complete notebook workflow for forecasting next-hour PM2.5 concentration at Rozelle, Sydney. The project evaluates whether neighbouring air-quality observations and available traffic information improve forecasting beyond local pollutant, meteorological and temporal data.

The study compares persistence, ARIMA, Random Forest, XGBoost, LSTM, GRU and an RF-ARIMA hybrid. Explainable AI is used to examine how the strongest model makes predictions and which variables contribute most strongly.

## Research questions

The project investigates:

1. Which statistical, machine-learning, deep-learning or hybrid model provides the most accurate next-hour PM2.5 forecasts at Rozelle?
2. Does neighbouring-station information improve forecast accuracy beyond local information?
3. Does available traffic information provide further improvement after neighbouring information is included?
4. Which variables have the greatest influence on the strongest forecasting model?
5. How reliable are the models during high-PM2.5 and NSW Poor-or-worse alert hours?

## Experimental design

The hourly dataset covers 2019-2024 and uses a chronological split:

| Partition | Period | Purpose |
| --- | --- | --- |
| Training | 2019-2022 | Fit preprocessing, feature screening and candidate models |
| Validation | 2023 | Select imputation, outlier treatment, scaling and hyperparameters |
| Test | 2024 | Final model evaluation |

The target is Rozelle PM2.5 one hour ahead.

Three controlled input scenarios are evaluated:

| Scenario | Information included |
| --- | --- |
| A Local | Rozelle pollutant, meteorological, temporal, lag and rolling features |
| B Neighbours | Scenario A plus information from five neighbouring Sydney monitoring stations |
| C Traffic | Scenario B plus eligible Rozelle and St Marys traffic variables |

Preprocessing, imputation, scaling and feature screening are fitted using training data only. The 2024 test set is reserved for final evaluation.

## Repository contents

```text
.
|-- README.md
|-- Sydney_Air_Quality_Research.ipynb
|-- merge_sydney_station_files.ipynb
|-- prepare_nsw_traffic_scenario_c_.ipynb
`-- data/
    `-- exports/
        `-- research_final_all_predictors/
```

### `merge_sydney_station_files.ipynb`

Combines the station-level air-quality files into an hourly multi-station dataset aligned by timestamp.

### `prepare_nsw_traffic_scenario_c_.ipynb`

Prepares the NSW traffic variables used to construct Scenario C.

### `Sydney_Air_Quality_Research.ipynb`

Contains the complete research workflow:

- Data loading and auditing
- Missingness and temporal-gap analysis
- Physical-validity checks
- Exploratory data analysis
- Imputation validation
- IQR and coherence-aware extreme-value experiments
- Lag, rolling, temporal and spatial feature engineering
- Scenario-specific feature screening
- Chronological train-validation-test separation
- Hyperparameter selection
- Final 30-seed model evaluation
- Paired t-tests and Wilcoxon signed-rank tests
- Holm multiple-comparison correction
- High-PM2.5 performance analysis
- NSW hourly alert-threshold evaluation
- Learning curves and residual diagnostics
- Random Forest feature importance
- Global, directional and local SHAP analysis
- Export of tables and reproducibility evidence

## Models

The evaluated models are:

- Persistence
- ARIMA
- Random Forest
- XGBoost
- LSTM
- GRU
- RF-ARIMA hybrid

Random Forest, XGBoost, LSTM, GRU and the hybrid model are evaluated across 30 matched seeds where applicable. LSTM and GRU use 24-hour input sequences and early stopping based on validation loss.

## Main findings

Neighbouring-station XGBoost produced the strongest test result:

- RMSE: 2.372 +/- 0.006 micrograms per cubic metre
- MAE: 1.706 +/- 0.003 micrograms per cubic metre
- R squared: 0.720 +/- 0.001

The main engineering findings were:

- Neighbouring information reduced XGBoost RMSE by 0.332% and Random Forest RMSE by 0.241%. Both improvements remained statistically significant after Holm correction, although their practical magnitude was small.
- Adding the available traffic variables did not provide a statistically reliable improvement to the strongest model.
- Automatically masking every 1.5-times-IQR candidate increased validation RMSE from 3.025 to 5.268, or approximately 74.2%. This showed that many statistical extremes were valid pollution signals rather than errors.
- XGBoost outperformed the recurrent and hybrid alternatives under the tested dataset and feature design.
- SHAP showed that current Rozelle PM2.5 and PM10 dominated the forecast, with smaller contributions from CO, recent PM2.5 history, NO2 and neighbouring mean PM2.5.
- Performance weakened during high-PM2.5 hours.
- Only one 2024 test hour exceeded the implemented NSW Poor-or-worse threshold. The precision and recall results therefore demonstrate the scoring procedure but are insufficient for assessing operational warning reliability.

These findings describe predictive associations and model behaviour. They do not establish environmental causation.

## Statistical analysis

Scenario comparisons use the same 30 seeds for the before-and-after conditions. Paired t-tests and Wilcoxon signed-rank tests compare:

- Scenario B against Scenario A
- Scenario C against Scenario B

Eight comparisons are conducted within each statistical-test family. Holm correction is applied separately to the paired t-test and Wilcoxon p-values to control the family-wise error rate.

The corresponding outputs are:

```text
scenario_significance_tests.csv
scenario_significance_tests_holm.csv
```

The unadjusted test statistics and p-values are retained for reproducibility, while Holm-adjusted p-values are used for the main statistical interpretation.

## Reproducing the analysis

### 1. Clone the repository

```bash
git clone https://github.com/Adan0108/sydney-air-quality-forecasting-ml-xai.git
cd sydney-air-quality-forecasting-ml-xai
```

### 2. Install the required Python packages

The notebooks require packages for data processing, statistical analysis, machine learning, deep learning and explainability:

```bash
pip install numpy pandas scipy statsmodels scikit-learn xgboost tensorflow shap matplotlib seaborn jupyter
```

The exact package versions from the final environment can be recorded with:

```bash
pip freeze > requirements.txt
```

### 3. Prepare the input data

Run the preparation notebooks if the merged input files have not already been created:

1. `merge_sydney_station_files.ipynb`
2. `prepare_nsw_traffic_scenario_c_.ipynb`

The raw source files are not necessarily stored in this repository because of file size and redistribution considerations. They should be placed in the paths documented inside the preparation notebooks.

### 4. Run the research notebook

Open `Sydney_Air_Quality_Research.ipynb` and run it from the project root so its relative data and export paths resolve correctly.

The complete workflow includes computationally expensive 30-seed LSTM and GRU experiments. Previously generated CSV evidence is included where practical, but reproducing every recurrent run may require substantial execution time.

## Reproducibility outputs

The final notebook exports evidence to:

```text
data/exports/research_final_all_predictors/
```

Important outputs include:

```text
all_model_runs.csv
best_model_by_scenario.csv
scenario_significance_tests.csv
scenario_significance_tests_holm.csv
high_pm25_model_runs.csv
high_pm25_model_summary.csv
imputation_comparison_by_feature.csv
extreme_validation_all_predictors.csv
traffic_feature_coverage.csv
deep_training_run_summary.csv
deep_learning_curve_summary.csv
```

These files support the reported model rankings, scenario comparisons, preprocessing decisions, stability analysis and high-PM2.5 evaluation.

## Data sources

The project uses publicly available New South Wales air-quality, meteorological and traffic observations. Full source details, citations and access information are provided in the associated capstone report and notebook.

Users reproducing the study should confirm the current dataset terms, variable definitions and official air-quality category thresholds at the time of use.

## Limitations

The study evaluates one target station, one forecast horizon and one final test year. The available traffic variables do not represent the complete Sydney road network, fleet composition or verified pollutant-transport pathways.

The 2024 test data contained only one hour above the implemented NSW Poor-or-worse PM2.5 boundary. The alert analysis must therefore be treated as exploratory rather than evidence of operational warning reliability.

SHAP explains the fitted model under correlated inputs. It identifies model reliance and predictive association, not causal environmental effects.

## Responsible use

This project is a research forecasting and explainability study. It is not a certified air-quality warning system and should not be used as a substitute for official NSW air-quality information or public-health advice.

## Author

Quang Anh Le  
Bachelor of Engineering Honours in Software Engineering  
University of Technology Sydney

## Repository

https://github.com/Adan0108/sydney-air-quality-forecasting-ml-xai
