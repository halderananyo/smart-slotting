# Smart Slotting: A Hybrid Machine Learning and Optimization Approach to Reduce Picker Travel and Labor Time

Published at the 9th European IEOM Conference, Barcelona, Catalonia, Spain, July 20–22, 2026.

**Authors:** Ananya Halder, Suvrojit Chanda, Anik Kumar Paul, Pratim Dash
**Departments:** Industrial and Production Engineering, BUET · Mechanical Engineering, MIST

## Overview

Order picking is the most labour-intensive warehouse activity, and travel — not retrieval —
consumes most of the picking cycle. This project couples a demand forecast directly to a
storage-slotting optimizer:

1. **Forecasting stage** — LightGBM and XGBoost predict 20-business-day-horizon picking
   demand per (Reference, Size) combination, benchmarked against Ridge regression and naive
   persistence. LightGBM (Tweedie objective) is the best model (R² = 0.849 on a strict
   temporal holdout).
2. **Optimization stage** — A genetic algorithm assigns 2,456 forecasted combinations across
   2,292 real storage locations under the warehouse's actual 18-unit-per-location capacity,
   using aisle-aware (rectilinear, corridor-based) distance rather than straight-line
   distance, plus a co-picking affinity bonus mined from real picking waves.
3. **Evaluation** — Layouts are scored by replaying real, unmodified picking waves. A key
   finding of this work is a **replica-volume confound**: layouts holding more stock per
   product appear to win purely from having more copies to pick from, independent of
   placement quality. Under a fair, equalized-stock-budget comparison, the forecast-driven
   layouts reduce travel by **22.5% (GA-Ridge)** and **20.0% (GA-ML)** against random
   storage — roughly **0.9–1.0 FTE** of labour saved per year.

Applied to a public real-world order-picking dataset from a footwear distribution centre in
Sherbrooke, Quebec (de Assis et al., *Data in Brief*, 2025).

## Repository structure

```
├── notebooks/
│   ├── 01_Feature_Engineering.ipynb          # Raw orders -> 26-feature forecasting panel
│   ├── 02_EDA.ipynb                          # Exploratory analysis
│   ├── 03_Baseline_Ridge.ipynb               # Naive persistence + Ridge baseline
│   ├── 04_Model_LightGBM_XGBoost.ipynb       # LightGBM (Tweedie) / XGBoost (Poisson)
│   ├── 05_Distance_Setup.ipynb               # Aisle-aware distance network (corridor waypoints)
│   ├── 06_GA_Optimizer.ipynb                 # Genetic algorithm: replicated-stock slotting
│   ├── 07_Wave_Evaluation.ipynb              # Replay real picking waves; Original vs. Fair metrics
│   └── 08_Labor_Efficiency_Analysis.ipynb    # Convert distance saved -> labour hours / FTE
├── data/
│   ├── forecasting_features.parquet          # Output of notebook 01 (428,401-row feature panel)
│   ├── model_results.csv                     # LightGBM/XGBoost predictions (output of notebook 04)
│   ├── baseline_results.csv                  # Naive/Ridge predictions (output of notebook 03)
│   ├── Storage_Location.csv                  # 2,292 warehouse slots (x, y, z)
│   └── Picking_Wave.csv                      # Real picking history (215,192 tasks, 9,707 waves)
├── requirements.txt
├── LICENSE
└── .gitignore
```

## Data note — files you'll need to add

This repo ships the intermediate/derived files needed to jump into the optimization stage
(`model_results.csv`, `baseline_results.csv`, `Storage_Location.csv`, `Picking_Wave.csv`,
`forecasting_features.parquet`). To re-run the pipeline **from scratch**, download the full
public dataset from Mendeley Data (CC BY 4.0):

> de Assis, R. F., de Paula Ferreira, W. and Ouhimmou, M. *Order picking dataset from a
> warehouse of a footwear manufacturing company.* Mendeley Data, DOI:
> [10.17632/pf2w725pw3.1](https://doi.org/10.17632/pf2w725pw3.1)

and place these additional files in `data/` alongside the ones already here:

| File | Used by |
|---|---|
| `Customer_Order.csv`, `Product.csv` | Notebook 01 (feature engineering) |
| `Support_Points_Navigation.csv` | Notebook 05 (corridor waypoints for aisle-aware distance) |
| `Random_Storage.csv`, `Dedicated_Storage.csv`, `Class_Based_Storage.csv`, `Hybrid_Storage.csv` | Notebook 07 (benchmark layout comparison) |

## Running the pipeline

```bash
pip install -r requirements.txt
jupyter notebook notebooks/01_Feature_Engineering.ipynb
```

Run the notebooks in numeric order (01 → 08) — each stage writes a CSV/parquet file the next
stage reads (`DATA_DIR = "."` in each notebook, so run them from the `data/` folder or adjust
the path). Notebooks 04, 06 and 07 regenerate `model_results.csv`, `ga_solution_*.csv`, and
`wave_evaluation_results.csv` respectively if you want to reproduce those from the raw data
rather than use the versions already included here.

## Key results

| Model | MAE | RMSE | R² |
|---|---|---|---|
| **LightGBM (Tweedie)** | **2.002** | 4.316 | **0.849** |
| XGBoost (Poisson) | 2.090 | 4.983 | 0.799 |
| Naive persistence | 2.312 | 5.267 | 0.776 |
| Ridge regression | 2.633 | 5.472 | 0.758 |

| Layout | Fair travel saving vs. Random | Labour saved (FTE-years/yr) |
|---|---|---|
| GA-Ridge | +22.5% | +1.02 |
| GA-ML | +20.0% | +0.90 |
| Class-Based | +6.5% | +0.30 |
| Random | 0.0% (reference) | 0.00 |
| Hybrid | −2.9% | −0.13 |
| Dedicated | −30.2% | −1.36 |

Full methodology, the replica-volume confound analysis, significance testing, and parameter
sensitivity sweep are in the published paper (IEOM Barcelona 2026 proceedings).

## Citation

> Halder, A., Chanda, S., Paul, A. K. and Dash, P. *Smart Slotting: A Hybrid Machine Learning
> and Optimization Approach to Reduce Picker Travel and Labor Time.* Proceedings of the 9th
> European International Conference on Industrial Engineering and Operations Management,
> Barcelona, Catalonia, Spain, July 20–22, 2026.

## License

MIT — see [LICENSE](LICENSE). Note this covers the code in this repository; the underlying
warehouse dataset is separately licensed CC BY 4.0 by its original authors (see Data note
above).
