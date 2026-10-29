# Predicting FDM Print Quality from Process Settings
## Summary:
In this project, I aimed to predict the print quality given process settings. After analysis, baseline MAE was 7.93 ± 1.14; the lowest mean MAE, from gradient boosting (not separable from the other two models), was 4.39 ± 1.41 was found for tensile strength. This analysis was conducted using 5-fold cross-validation repeated 10 times with unverified units.
## Problem Statement:
3D print settings affect the strength, roughness, and elongation of the final print. Trial-and-Error printing processes result in wasted materials and time. The goal of this project was to predict quality of final prints from given print settings, which was then used to search for the settings producing the best final print quality.
## Data:
Kaggle “3D Printer Dataset for Mechanical Engineers,” Ultimaker S5, 50 rows, 9 settings, 3 outputs (roughness, tensile strength, elongation), whole-number outputs.
Units are not verified.
## Methods:
I used tensile strength as the target and measured error with Mean Absolute Error (MAE). One-hot encoding and scaling happen inside each model pipeline, so no test data leaks into training. I compared four models on the same folds with a fixed seed (42): a mean baseline, linear regression, random forest, and gradient boosting. The standard deviation across the overlapping folds gives a rough sense of the noise. For feature importance, I used permutation importance on an 80/20 split, which leaves only 10 rows for testing. For the optimization step, I used differential_evolution.
## Results:
All three models beat the baseline by more than the fold-to-fold spread.

| Model | MAE (mean ± std) |
|---|---|
| Baseline (mean) | 7.93 ± 1.14 |
| Linear regression | 5.14 ± 1.11 |
| Random forest | 4.91 ± 1.14 |
| Gradient boosting | 4.39 ± 1.41 |

Gradient boosting had the lowest MAE at 4.39, but I can't say it's better than the other two. The gaps between them are smaller than the spread across folds, and gradient boosting also has the largest spread. In an earlier notebook (archive/), tuning the random forest moved MAE from 4.89 to 4.87. That number is slightly optimistic because tuning and evaluation used the same folds. Wall thickness (1.32) and infill density (0.94) had the highest permutation importance, which makes sense since both add load-bearing material. Material mattered very little. With only 10 test rows, the middle bars are noisy.

![Model comparison](figures/model_comparison.png)

Takeaway: All three models beat the baseline, and the gaps among them are smaller than the fold spread.

![Permutation importance](figures/permutation_importance_tension_strenght.png)

Takeaway: Wall thickness and infill density, the settings that add load-bearing material, matter most.

## Predict-Then-Optimize:
I searched combinations of values seen in the data for each setting, treating bed speed and fan speed as pairs that occur together. The search used differential_evolution with a roughness cap of 165.5 (the median roughness, units unverified). The best predicted run was PLA with a grid pattern (strength 33.24, roughness 164.84). That's lower than the best real run in the data, ABS with a grid pattern (strength 37, roughness 75), so the search didn't find better settings. The roughness model is also the weaker one, since it reuses the tensile strength hyperparameters and was never cross-validated. The gap of about 3.8 between predicted and historical best is smaller than the CV MAE, and changing the seed flipped the top pick from ABS to PLA, so I wouldn't trust these recommendations.
## Limitations:
With only 50 rows, estimates are noisy and a single split isn't reliable. Units haven't been verified. ABS was printed at 220–250 °C and PLA at 200–220 °C, so the model may be using temperature in place of material, but I haven't tested this. Bed speed and fan speed move together (r = 1.00), so their separate effects can't be told apart. The roughness cap also relies on untuned tensile hyperparameters and has no cross-validation of its own.
## Next Steps:
1. Verify units for every column in the dataset.
2. Tune and cross-validate separate models for roughness and elongation.
3. Collect more ABS and PLA data at overlapping temperatures.
4. Print the recommended settings to check the predictions.

## How to reproduce:
- Open FinalNotebook in Google Colab
- Upload data.csv to /content
- Runtime → Restart session and run all
- Local runs: pip install -r requirements.txt
