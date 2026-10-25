# Predicting FDM Print Quality from Process Settings
## Summary:
In this project, I aimed to predict the print quality given process settings. After analysis, baseline MAE was 7.93 ± 1.14; the lowest mean MAE, from gradient boosting (not separable from the other two models), was 4.39 ± 1.41 was found for tensile strength. This analysis was conducted using 5-fold cross-validation repeated 10 times with unverified units.
## Problem Statement:
3D print settings affect the strength, roughness, and elongation of the final print. Trial-and-Error printing processes result in wasted materials and time. The goal of this project was to predict quality of final prints from given print settings, which was then used to search for the settings producing the best final print quality. 
## Data:
Kaggle “3D Printer Dataset for Mechanical Engineers,” Ultimaker S5, 50 rows, 9 settings, 3 outputs (roughness, tensile strength, elongation), whole-number outputs.
Units are not verified. 
## Methods:
Tensile strength was evaluated as the target variable for print quality using Mean Absolute Error (MAE). Feature preprocessing (one-hot encoding and feature scaling) was embedded inside model pipelines to prevent data leakage. Baseline (mean), linear regression, random forest, and gradient boosting models were compared across identical folds using a fixed seed (42), with standard deviations from overlapping folds serving as a rough noise gauge. Feature importances were calculated via permutation importance on an 80/20 train/test split (10-row test split). Parameter search for optimization was conducted using differential_evolution.
## Results:
All three trained models beat the baseline by more than the fold spread.

| Model | MAE (mean ± std) |
|---|---|
| Baseline (mean) | 7.93 ± 1.14 |
| Linear regression | 5.14 ± 1.11 |
| Random forest | 4.91 ± 1.14 |
| Gradient boosting | 4.39 ± 1.41 |

Gradient boosting achieved the lowest mean MAE (4.39) for tensile strength, though the three models are not separable from each other because their performance gaps are smaller than the fold spread. Furthermore, gradient boosting's standard deviation (1.41) is the largest among the trained models, so it can’t be ranked above random forest or linear regression. In an earlier notebook (archive/), tuning random forest shifted MAE from 4.89 to 4.87, though this score is slightly optimistic because tuning and evaluation reused the same folds. Permutation importances calculated from the test split showed wall thickness (1.32) and infill density (0.94) had the largest permutation importance, which plausibly aligns with adding load-bearing material, while material importance was near zero. The importance split has only 10 test rows, so mid-ranked bars are noisy.

![Model comparison](figures/model_comparison.png) 

Takeaway: All three models beat the baseline; the gaps among the three are smaller than the fold spread.

![Permutation importance](figures/permutation_importance_tension_strenght.png)

Takeaway: Load-bearing structural parameters (wall thickness and infill density) have the largest permutation importance.

## Predict-Then-Optimize:
Searching combinations of observed values per setting (treating bed and fan speed as observed pairs) via differential_evolution under a roughness cap of 165.5 (median roughness, units unverified) yielded a top predicted run of PLA with a grid pattern (predicted strength 33.24, roughness 164.84). The predicted best strength (33.24) sits below the best observed historical run of ABS with a grid pattern (strength 37, roughness 75), meaning the search found no better settings. Additionally, the predicted roughness (164.84) sits just under the cap, and the roughness model is the weaker one because it reuses tensile hyperparameters and was never cross-validated. The gap of about 3.8 between predicted and historical best is smaller than the CV MAE, and recommendations proved unstable as changing the seed flipped the top choice from ABS to PLA.
## Limitations:
Small sample size (n = 50) makes estimates noisy and single-split evaluation unreliable. Units are not verified. ABS was printed at 220–250 °C while PLA was printed at 200–220 °C, creating a confounding effect where it is hypothesized that the model uses temperature in place of material, though this remains untested. Bed and fan speeds move together (r = 1.00), meaning their individual effects cannot be separated. Additionally, the roughness constraint relies on untuned tensile hyperparameters without dedicated cross-validation.
## Next Steps:
1. Verify units across all dataset attributes.
2. Tune and cross-validate separate models for roughness and elongation.
3. Collect additional data with ABS and PLA at overlapping printing temperatures.
4. Print the recommended settings physically to validate model predictions.

## How to reproduce:
- Open FinalNotebook in Google Colab
- Upload data.csv to /content
- Runtime → Restart session and run all
- Local runs: pip install -r requirements.txt
