# SwiftRoute Logistics — Delivery Delay Prediction

Submission overview for the Logistics mini-coursework. The completed analysis is in [coursework.ipynb](coursework.ipynb); the fitted model is [model.pkl](model.pkl).

## Objective

Rank shipments at pickup by their risk of delivery outside the promised window, so Operations can prioritise interventions for the highest-risk **15% of shipments**. The binary target is `is_delayed`.

## Submission files

All coursework files are under [``]().

| File | Purpose |
| --- | --- |
| [coursework.ipynb](coursework.ipynb) | Main assessed notebook: written answers, analysis, modelling, evaluation and saved outputs |
| [model.pkl](model.pkl) | Selected model and fitted preprocessing pipeline, saved with joblib |
| [requirements.txt](requirements.txt) | Pinned direct dependencies for reproduction |
| [data/](data/) | Supplied `shipments.csv`, `routes.csv`, `carriers.csv` and `weather_events.csv` |
| [results.json](results.json) | Machine-readable metrics, experiment results, checks and model input contract |
| [validation_predictions.csv](validation_predictions.csv) | Held-out labels, predicted risks and intervention flags |
| [figures/](figures/) | Generated EDA, model, business, clustering and SHAP figures |
| [business_brief.md](business_brief.md) / [data_dictionary.md](data_dictionary.md) | Supplied business context and data definitions |

## Analysis and method

- Audit missing values, repeated inputs, labels and lookup integrity. Cleaning retains **10,000 shipments**, including **1,191 delayed shipments**.
- Join route and carrier attributes without multiplying shipment rows. Match weather by region and inclusive event dates, taking maximum overlapping severity separately at origin and destination.
- Explore service, carrier, route, distance, time and weather patterns; engineer pickup-time predictors.
- Use an 80/20 stratified split with seed 42: **8,000 training** and **2,000 validation** rows. Compare Logistic Regression and Random Forest using five-fold training-only cross-validation, with preprocessing fitted inside each fold.
- Select the model by mean training CV ROC-AUC, then evaluate the fixed selection on the held-out validation set.
- Evaluate a fixed top-15% intervention policy; add route clustering and SHAP explanations for the ten highest-risk validation shipments.

## Results

The selected model is **Logistic Regression with enriched features, `C=10`, and no class weighting**.

| Measure | Result |
| --- | --- |
| Mean training CV ROC-AUC | 0.8685 |
| Validation ROC-AUC | 0.8687 |
| Validation shipments flagged | 300 / 2,000 |
| Actual delays captured | 151 / 238 |
| Delay capture rate | 63.45% |
| Precision among flagged shipments | 50.33% |
| Expected capture rate under random selection | 15.00% |
| Capture lift over random selection | 4.23× |
| Potential gross avoided cost at £45 per captured delay | £6,795.00 |

The cost figure is an **upper-bound scenario**, assuming every captured delay is prevented and excluding intervention costs. It is not measured savings. Capture measures the share of all actual delays found; precision measures the share of flagged shipments that are actually delayed.

## Reproduce the notebook

The recorded execution used **Python 3.14.4**. From this repository's root on Linux/macOS, prepare the environment and register the notebook kernel:

```bash
python3.14 -m venv .venv
.venv/bin/python -m pip install -r requirements.txt
.venv/bin/python -m ipykernel install --user --name coursework-logistics --display-name "Coursework Logistics"
```

Open `coursework.ipynb`, select **Coursework Logistics**, and run all cells. Alternatively, execute it from a fresh kernel using:

```bash
.venv/bin/python -c "from pathlib import Path; import nbformat; from nbclient import NotebookClient; p=Path('coursework.ipynb').resolve(); nb=nbformat.read(p,as_version=4); NotebookClient(nb,timeout=1800,kernel_name='coursework-logistics',resources={'metadata':{'path':str(p.parent)}}).execute(); nbformat.write(nb,p)"
```

Execution regenerates the model, results, validation predictions and figures, and saves notebook outputs. Keep the four supplied CSVs in `data/`; redistribute them only if permitted by the course.

## Saved model contract and verification

`model.pkl` accepts the **engineered feature table**, not the four raw CSV tables. Feature preparation is implemented in the notebook; required columns and types are documented in its Section 8.2 and in `artifact.required_columns` in `results.json`. The pipeline contains fitted imputation, encoding, scaling where applicable, and the classifier.

The recorded fresh-kernel verification completed **20/20 code cells without error outputs**. Reloaded holdout probabilities matched exactly. Checks also exercised missing numeric inputs, unseen categorical values, weather boundaries and overlap, exact intervention capacity, and SHAP reconstruction.

## Limitations and submission status

- Random-split validation does not establish performance on future periods or entirely new routes. Pickup-time availability of supplied historical/weather attributes remains a documented assumption.
- Weather coverage is incomplete near the end of the shipment period. Observational associations, route clusters and SHAP contributions do not establish causal effects.

